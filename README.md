## Step 1: Create a GitHub Repository and Initialize Code


---

1. I opened [GitHub.com](https://github.com) and logged into my account.

2. I clicked the **"New Repository"** button to make a new project.

3. I typed a random name for the repository (example: `secret-scan-demo`).

4. I tried both options — **Public** and **Private** — to test them.

5. I selected the checkbox **"Initialize with a README"** to add a starting file.

6. Then, I clicked **"Create repository"** to finish creating the repo.

---



## Step 2: I Copied the Repo to My Computer and Added Sample Code


---

1. I opened the **terminal** (or command prompt) on my computer.

2. I ran this command to **clone the repo** from GitHub to my computer:
   (I used my real GitHub username)

   ```bash
   git clone https://github.com/<your-username>/secret-scan-demo.git
   cd secret-scan-demo
   ```

3. Inside the folder, I made a new file called `main.py` and added this code:

   ```python

   def hello():
       print("Hello from main branch!")

   hello()

   ```

  This is just a small program that prints a message on the screen.

4. I saved the file and went back to the terminal.

   Then, I ran these commands to **save the file and send it to GitHub**:

   ```bash

   git add .

   git commit -m "Initial commit on main branch"

   git push origin main

   ```

---


## Step 3: I Created a New Branch and Added a Fake Secret

---

###  Goal:

I wanted to test secret scanning by adding a fake API key.

This helps check if the scanner can find secrets in the code.

---

1. I created a new branch called `dev` and switched to it by running this command:

   ```bash

   git checkout -b dev

   ```

   This made a copy of the code so I could test changes without affecting the main code.


2. I opened the file `main.py` and added this code:

   ```python


   def hello():
       print("Hello from dev branch!")

   API_KEY = "sk_test_1234567890abcdefg"  # This is a fake secret

   hello()


   ```

   This line is not a real secret. I added it just to test if secret scanning works.

3. I saved the file and ran these commands to send the changes to GitHub:

   ```bash

   git add main.py
   git commit -m "Added fake secret in dev branch"
   git push origin dev

   ```

---


##  Step 4: I Created a Pull Request (PR) from `dev` to `main`

---

###  Goal:

I want to merge my changes from the `dev` branch into the `main` branch.

---

###  What I Did (Steps):

1. I went to my GitHub repository page.

2. I saw a banner saying **“Compare & pull request”** and clicked on it.

3. I checked that the **base branch** was set to `main` and the **compare branch** was set to `dev`.

4. I added a title: **“Add secret for testing”**.

5. I added a description: **“This PR adds a fake API key to test secret scanning.”**

6. Finally, I clicked on **Create pull request**.

---
   * Base branch: `main`

   * Compare branch: `dev`

     Add title: `Add secret for testing`

     Add description: `This PR adds a fake API key to test secret scanning.`

      Click **Create pull request**.

**GitHub Actions will run secret scanning when a PR targets the main branch.**

---


##  Step 5: I Set Up GitHub Actions to Scan for Secrets Automatically

---

###  Goal:

Make GitHub check for secrets every time someone makes or updates a pull request (PR).

---

###  What I Did:

1. On my computer, I created a folder and a file for the workflow:

   ```bash


   mkdir -p .github/workflows
   touch .github/workflows/secret-scan.yml


   ```

2. I opened `.github/workflows/secret-scan.yml` and added this code (this tells GitHub what to do):

   ```yaml

   name: Secret Scan with Trivy on PR

   on:
     pull_request:
       branches:
         - main

   jobs:
     trivy-secret-scan:
       runs-on: ubuntu-latest
       name: Run Trivy Secret Scan on PR
       steps:
         - name: Checkout PR code
           uses: actions/checkout@v3

         - name: Install Trivy
           run: |
             sudo apt-get update
             sudo apt-get install -y wget apt-transport-https gnupg lsb-release
             wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
             echo deb https://aquasecurity.github.io/trivy-repo/deb "$(lsb_release -sc)" main | sudo tee -a /etc/apt/sources.list.d/trivy.list
             sudo apt-get update
             sudo apt-get install -y trivy

         - name: Run Trivy Secret Scan
           id: trivy-secrets
           run: |
             trivy fs --scanners secret --exit-code 1 --format table .



   ```


---


   ### . Workflow Name 
   name: Secret Scan with Trivy on PR  //  This is the name of the workflow. It tells you what the workflow does — it checks for secrets in your code.

```

---

 ### . When to Run the Workflow


This means the workflow runs every time someone opens or changes a **pull request** to the `main` branch.

```yaml

on:
  pull_request:
    branches:
      - main
```
---

### Define the Job

```yaml

* This creates a job named `trivy-secret-scan`.
* It runs on a computer with the latest Ubuntu operating system.
* The job will run the secret scan.

jobs:
  trivy-secret-scan:
    runs-on: ubuntu-latest
    name: Run Trivy Secret Scan on PR
```

---

###  Checkout Code Step

```yaml
* This step copies your pull request’s code so the workflow can check it.

steps:
  - name: Checkout PR code
    uses: actions/checkout@v3
```

---

### . Install Trivy Step

```yaml

* This step installs the Trivy tool, which finds secrets in your code.
* It runs commands to prepare the computer and then install Trivy.

  - name: Install Trivy
    run: |
      sudo apt-get update
      sudo apt-get install -y wget apt-transport-https gnupg lsb-release
      wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
      echo deb https://aquasecurity.github.io/trivy-repo/deb "$(lsb_release -sc)" main | sudo tee -a /etc/apt/sources.list.d/trivy.list
      sudo apt-get update
      sudo apt-get install -y trivy
```


---

### . Run Trivy Secret Scan Step

```yaml
* This step runs Trivy to scan your code folder (`.`).
* It looks only for secrets like passwords or keys.
* If Trivy finds secrets, it will **fail the check** (exit code 1).
* The results are shown in a simple table.

  - name: Run Trivy Secret Scan
    id: trivy-secrets
    run: |
      trivy fs --scanners secret --exit-code 1 --format table .
```



3. Then I saved the file and ran these commands to upload it to GitHub:

   ```bash

   git add .github/workflows/secret-scan.yml
   git commit -m "Add secret scanning GitHub Action"
   git push origin dev

   ```

---

###  Why This Step Is Important:

* It makes sure GitHub will run secret scans automatically on every PR to the `main` branch.
* Helps catch secrets before code is merged.

---

##  Step 6: I Triggered the Secret Scan by Updating the Pull Request

---

###  Goal:

Run the secret scanning workflow on the open PR.

---

###  What I Did:

1. Since the PR is open from `dev` to `main`, pushing the workflow file starts the scan automatically.

2. I went to the **Actions** tab or the PR page on GitHub.

3. I watched the secret scan run.

---

###  What to Expect:

* The scanner should find the fake API key and mark the check as failed.

---

##  Step 7: I Checked Why the PR Failed

---

###  Goal:

Understand the scan results and find where the secret was detected.

---

###  What I Did:

1. On the PR page, I saw a red  next to `Secret Scan on PR`.

2. I clicked **Details** or **View logs**.

3. I looked for messages like:

   ```

   Error: Process completed with exit code 1.

      “Error: Process completed with exit code 1” means the program finished but found a problem.
      In this case, the problem is the fake API key (a secret) in your code.
      Because of this, the scan stopped and showed an error to protect your project.


   ```

---

### Why This Is Important:

* This shows the secret scanner worked and found the fake secret.
* The PR fails for security reasons until fixed.

---

##  Step 8: I Removed the Secret and Updated the PR

---

###  Goal:

Fix the problem by removing the secret and sending the fixed code.

---

###  What I Did:

1. I edited `main.py` to remove the secret:

   ```python

   def hello():
       print("Hello from dev branch!")

   API_KEY = "REDACTED"  # Secret removed

   hello()

   ```

2. I saved the file and ran:

   ```bash

   git add main.py
   git commit -m "Remove fake secret"
   git push origin dev

   ```

3. The GitHub Action ran again automatically.

---

##  Step 9: I Verified the Secret Scan Passed

---

###  Goal:

Make sure the secret scanner says everything is okay.

---

###  What I Did:

1. I checked the **Checks** tab or the PR page.

2. I waited for the `Secret Scan on PR` check to finish.

3. I saw a green , meaning the scan passed and it’s safe to merge.

---

##  Step 10: I Added Branch Protection to Keep `main` Safe

---

###  Goal:

Stop anyone from merging code with secrets into the `main` branch.

---

###  What I Did:

1. I went to my repo’s **Settings** → **Branches**.

2. Under “Branch protection rules,” I clicked **Add rule**.

3. In “Branch name pattern,” I typed `main`.

4. I turned on:

   * **Require status checks to pass before merging**

   * Selected `Secret Scan on PR`

   * **Require pull request before merging**

   * (Optional) **Require approvals** for code review

5. I saved the changes.

---

###  What Happens Now:

* No one can merge to `main` unless the secret scan and other checks pass.
* This protects the code from secrets accidentally getting in.

---

























