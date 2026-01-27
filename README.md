This lab builds directly on the previous CI lab. While CI ensures our code is healthy, **Continuous Delivery (CD)** ensures that the healthy code is packaged and moved toward a production environment automatically.

---

### **Lab Overview: Delivering the Calculator**

**Objective:**
Students will extend their pipeline to automatically create a "Production Release" on GitHub whenever a new version (tag) of the code is pushed. This simulates the process of preparing a software package for customers.

**Prerequisites:**

* Completion of the CI Lab.
* A GitHub account.

---

### **Part 1: The Repository Files**

We will add a new workflow file. Your structure should now look like this:

```text
cd-lab-calculator/
├── .github/
│   └── workflows/
│       ├── ci.yml  (From previous lab)
│       └── cd.yml  <-- NEW FILE
├── src/
│   └── calculator.py
├── tests/
│   └── test_calculator.py
├── requirements.txt
└── README.md

```

#### **1. The CD Pipeline (`.github/workflows/cd.yml`)**

This workflow is more "picky." It only runs when you tell GitHub, "This code is ready for a version release" (by using a Git Tag).

```yaml
name: Continuous Delivery (Production Release)

# This triggers only when a tag starting with 'v' is pushed (e.g., v1.0)
on:
  push:
    tags:
      - 'v*'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Run Final Sanity Check
        run: python -m unittest discover tests

      - name: Package Application
        run: |
          mkdir -p release
          cp src/calculator.py release/
          tar -cvzf calculator-v${{ github.ref_name }}.tar.gz -C release .

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          files: calculator-v${{ github.ref_name }}.tar.gz
          body: "Automated release of version ${{ github.ref_name }}"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```

---

### **Part 2: The Student `README.md**`

Add this section to your existing README.

---

# Lab Part 2: Continuous Delivery (CD)

In Part 1, we made sure the code didn't break. In Part 2, we are going to automate the **delivery** of our app. Instead of just "testing" the code, we are going to "ship" it.

## 📦 Step 1: Understanding the Trigger

Look at `cd.yml`. Notice it doesn't run on every push. It waits for a **Tag**. In the professional world, tags represent specific versions (like v1.0 or v2.4).

## 🚀 Step 2: Creating your first Release

1. Ensure your code is working and all tests pass.
2. Open your terminal (or GitHub Desktop) and run these commands to "tag" your code:
```bash
git tag v1.0
git push origin v1.0

```


3. Go to the **Actions** tab. You will see a new workflow named "Continuous Delivery" starting!
4. Once it finishes, go to the **main page** of your GitHub repository.
5. Look at the right-hand sidebar under **"Releases."** You should see `v1.0`.
6. Click it. You will find a `.tar.gz` file—this is your "shipped" software!

## 🧪 Step 3: The "Safety Net"

1. Break your code again in `src/calculator.py` (e.g., change `return x + y` to `return x * 99`).
2. Try to ship this broken version:
```bash
git add .
git commit -m "Attempting to ship broken code"
git tag v1.1
git push origin v1.1

```


3. **Observe:** The CD pipeline will fail at the "Sanity Check" step. The release will be created, but it will be marked as failed/incomplete. In a real-world CD setup, this prevents broken software from reaching your customers.

---

### **Part 3: Student Exercises**

#### **Exercise 1: The Release Notes**

* **Task:** Modify `cd.yml`. Change the `body` of the release to include the name of the person who triggered the build.
* **Hint:** Look into GitHub context variables like `${{ github.actor }}`.
* **Goal:** Understand how to customize automated deployment metadata.

#### **Exercise 2: Multi-Environment Simulation**

* **Task:** Create a new branch named `staging`. Modify the `ci.yml` so that it runs on `main`, but create a new workflow `staging.yml` that only runs when code is pushed to the `staging` branch.
* **Goal:** Understand how code moves through different "levels" (Dev -> Staging -> Prod).

#### **Exercise 3: Environment Secrets**

* **Task:** Imagine you need a password to upload your code to a server. Go to your GitHub Repo **Settings > Secrets and variables > Actions**. Create a secret named `DEPLOY_TOKEN`.
* **Task:** Add a step in `cd.yml` that simply prints "Using secret token" and then echoes the secret (Note: GitHub will mask it with `***`).
* **Goal:** Learn how to handle sensitive data in a CD pipeline.

#### **Exercise 4: The "Rollback" Concept**

* **Task:** If `v1.1` was broken, how do you fix it? Fix the bug in your code, then push a new tag `v1.2`.
* **Goal:** Understand that in CD, we "fix forward" by deploying a new, corrected version.

#### **Exercise 5: Artifact Cleanup**

* **Task:** Currently, we package only `calculator.py`. Modify the "Package Application" step in `cd.yml` to also include the `requirements.txt` file in the final `.tar.gz` bundle.
* **Goal:** Learn how to manage what actually gets sent to the user/server.

---

### **Would you like me to:**

1. Explain how to add a **Manual Approval** step (a common "Gate" in CD)?
2. Show you how to deploy this calculator as a **Docker Image** to Docker Hub instead of a GitHub Release?