Got it! Here’s a **complete, beginner-friendly, step-by-step guide** to your secret scanning GitHub task — from creating the repo to setting branch protections. I’ll break it down into **10 clear steps**, each with simple explanations.

---

# Secret Scan: Complete Step-by-Step Guide (Step 1 to Step 10)

---

## Step 1: Create a GitHub Repository and Initialize Code

**Goal:** Create a place to store your project on GitHub and add some initial code.

### What to do:

1. Go to [github.com](https://github.com) and log in.
2. Click **New Repository** (usually a green button).
3. Name your repo (e.g., `secret-scan-demo`).
4. Choose Public or Private (your choice).
5. Check **Initialize with a README**.
6. Click **Create repository**.

---

## Step 2: Clone Repo Locally and Add Sample Code

**Goal:** Copy the repo to your local machine and add a simple Python file.

### What to do:

1. Open your terminal or command prompt.
2. Run:

```bash
git clone https://github.com/<your-username>/secret-scan-demo.git
cd secret-scan-demo
```

3. Create a file `main.py` with:

```python
def hello():
    print("Hello from main branch!")

hello()
```

4. Add and commit changes:

```bash
git add .
git commit -m "Initial commit on main branch"
git push origin main
```

**Why?**
This sets the starting point of your project.

---

## Step 3: Create a `dev` Branch and Add a Fake Secret

**Goal:** Simulate adding a secret in a development branch to test scanning.

### What to do:

1. Create and switch to the `dev` branch:

```bash
git checkout -b dev
```

2. Modify `main.py` to add a fake secret:

```python
def hello():
    print("Hello from dev branch!")

API_KEY = "sk_test_1234567890abcdefg"  # <-- Fake secret

hello()
```

3. Commit and push:

```bash
git add main.py
git commit -m "Added fake secret in dev branch"
git push origin dev
```

**Why?**
You want to test that the secret scanner can catch this fake API key.

---

## Step 4: Create a Pull Request (PR) from `dev` to `main`

**Goal:** Request to merge your changes from `dev` into `main`.

### What to do:

1. Go to your GitHub repo page.
2. You’ll see a banner like: “Compare & pull request” — click it.
3. Make sure:

   * Base branch: `main`
   * Compare branch: `dev`
4. Add title: `Add secret for testing`
5. Add description: `This PR adds a fake API key to test secret scanning.`
6. Click **Create pull request**.

**Why?**
GitHub Actions will run secret scanning when a PR targets the main branch.

---

## Step 5: Set Up GitHub Actions Workflow for Secret Scanning

**Goal:** Automate secret detection every time a PR is opened or updated.

### What to do:

1. In your local repo, create folder and workflow file:

```bash
mkdir -p .github/workflows
touch .github/workflows/secret-scan.yml
```

2. Open `.github/workflows/secret-scan.yml` and add:

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

3. Commit and push:

```bash
git add .github/workflows/secret-scan.yml
git commit -m "Add secret scanning GitHub Action"
git push origin dev
```

**Why?**
This workflow runs TruffleHog to scan your code for secrets every time a PR is made to `main`.

---

## Step 6: Trigger the Workflow by Updating the PR

**Goal:** Let the secret scanning run on your open PR.

### What to do:

1. Because your PR is open from `dev` to `main`, pushing the workflow file triggers it.
2. Go to the **Actions** tab on GitHub or the PR page.
3. Watch the workflow run.

**What to expect:**
The secret scanner should detect the fake `API_KEY` and fail the check.

---

## Step 7: Understand and Review the Failed PR Check

**Goal:** Learn what caused the failure and where to see details.

### What to do:

1. On the PR page, see the red ❌ next to `Secret Scan on PR`.
2. Click **Details** or **View logs**.
3. Look for lines like:

```
Secrets found!
Reason: High Entropy String
Raw: sk_test_1234567890abcdefg
```

**Why?**
This confirms the secret was detected, so the PR fails for security.

---

## Step 8: Fix the Issue (Remove Secret) and Push Fix

**Goal:** Remove the secret from your code and update the PR.

### What to do:

1. Edit `main.py`:

```python
def hello():
    print("Hello from dev branch!")

API_KEY = "REDACTED"  # Secret removed

hello()
```

2. Commit and push:

```bash
git add main.py
git commit -m "Remove fake secret"
git push origin dev
```

3. The GitHub Action will automatically re-run.

---

## Step 9: Verify the PR Check Passes

**Goal:** Confirm the secret scan now passes and PR is safe to merge.

### What to do:

1. Check the **Checks** tab or PR page.
2. Wait for `Secret Scan on PR` to finish.
3. You should see a green ✅ meaning success.

---

## Step 10: Add Branch Protection to Prevent Unsafe Merges

**Goal:** Block merging if secret scans fail, protecting your `main` branch.

### What to do:

1. Go to your repo → **Settings** → **Branches**.
2. Under “Branch protection rules,” click **Add rule**.
3. In "Branch name pattern," enter `main`.
4. Enable:

   * **Require status checks to pass before merging**
   * Select `Secret Scan on PR` (your workflow)
   * **Require pull request before merging**
   * Optional: **Require approvals** for code review
5. Save changes.

---

### What happens now?

* No one can merge code into `main` unless **all checks pass**.
* If a PR contains secrets, the secret scan fails, blocking merge.
* This keeps your codebase safe from accidental secret leaks.

---

# Summary Table

| Step | What You Do                     | Why It Matters            |
| ---- | ------------------------------- | ------------------------- |
| 1    | Create repo & add README        | Project base ready        |
| 2    | Clone and add code              | Prepare sample code       |
| 3    | Create `dev` branch, add secret | Simulate secret leak      |
| 4    | Open PR from `dev` to `main`    | Trigger scans on new code |
| 5    | Add GitHub Actions workflow     | Automate secret scanning  |
| 6    | Trigger workflow by PR update   | Run scans on your code    |
| 7    | Review failure logs             | Understand why PR failed  |
| 8    | Remove secret and push fix      | Fix security risk         |
| 9    | Verify checks pass              | Ensure safe code          |
| 10   | Set branch protection           | Block unsafe merges       |

---

If you want, I can also:

* Generate the full GitHub Action YAML file for you
* Help set up Slack/email notifications on failure
* Show how to use other secret scanning tools like Gitleaks

Just ask!
# public-trivy-scan