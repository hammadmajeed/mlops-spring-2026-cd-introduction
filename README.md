Here is a complete, hands-on lab task designed to demonstrate **Continuous Integration (CI)**.

This lab uses **GitHub Actions** because it is free, integrated directly into GitHub, and requires no external server setup. The application is a simple Python calculator, making the code easy to understand so students can focus on the CI concepts.

---

### **Lab Overview: The "Bulletproof" Calculator**

**Objective:**
Students will set up a CI pipeline that automatically runs unit tests every time they push code. They will learn how CI protects the codebase from "breaking changes."

**Prerequisites:**

* A GitHub Account.
* Basic understanding of Git (commit, push).

---

### **Part 1: The Repository Files**

You (or the students) should create a folder structure like this:

```text
ci-lab-calculator/
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
│   └── calculator.py
├── tests/
│   └── test_calculator.py
├── requirements.txt
└── README.md

```

Here is the code for each file to copy-paste.

#### **1. The Application Code (`src/calculator.py`)**

A simple script with basic math functions.

```python
def add(x, y):
    return x + y

def subtract(x, y):
    return x - y

def multiply(x, y):
    return x * y

def divide(x, y):
    if y == 0:
        raise ValueError("Cannot divide by zero")
    return x / y

```

#### **2. The Test Suite (`tests/test_calculator.py`)**

Unit tests that verify the calculator works.

```python
import unittest
from src.calculator import add, subtract, multiply, divide

class TestCalculator(unittest.TestCase):

    def test_add(self):
        self.assertEqual(add(10, 5), 15)
        self.assertEqual(add(-1, 1), 0)

    def test_subtract(self):
        self.assertEqual(subtract(10, 5), 5)

    def test_multiply(self):
        self.assertEqual(multiply(10, 5), 50)

    def test_divide(self):
        self.assertEqual(divide(10, 2), 5)
        with self.assertRaises(ValueError):
            divide(10, 0)

if __name__ == '__main__':
    unittest.main()

```

#### **3. Dependencies (`requirements.txt`)**

We only need a standard install, but it's good practice to include this file.

```text
# No external dependencies for this simple lab
# But usually, you would list things like:
# pytest==7.0.0

```

#### **4. The CI Pipeline (`.github/workflows/ci.yml`)**

This is the heart of the lab. It tells GitHub what to do when code is pushed.

```yaml
name: Python CI Pipeline

# Trigger the workflow on push or pull request to the main branch
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    # Step 1: Check out the repository code
    - name: Checkout code
      uses: actions/checkout@v3

    # Step 2: Install Python
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'

    # Step 3: Install dependencies (if any)
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

    # Step 4: Run the tests
    - name: Run Unit Tests
      run: |
        python -m unittest discover tests

```

---

### **Part 2: The Student `README.md**`

Copy the content below into the `README.md` file. This serves as the instruction manual for the student.

---

# Lab: Intro to Continuous Integration (CI)


## 🏁 Step 1: Setup

1. Click the **Actions** tab in your GitHub repository. You should see a workflow run listed (triggered by your initial fork/push).
2. Click on the workflow run to see the steps. Ensure all steps have a green checkmark ✅.

## 🧪 Step 2: The "Happy Path"

1. Open `src/calculator.py`.
2. Add a small comment to the file (e.g., `# This is a change`).
3. Commit and push the change to GitHub.
4. Go to the **Actions** tab immediately. Watch the pipeline start automatically.
5. Observe how it sets up Python, installs dependencies, and runs the tests. It should pass.

## 💥 Step 3: Break the Build!

Now, let's see what happens when we make a mistake.

1. Open `src/calculator.py`.
2. Change the `add` function so it is **incorrect**:
```python
def add(x, y):
    return x - y  # This is a bug!

```


3. Commit and push this change.
4. Go to the **Actions** tab.
5. **Observe:** The pipeline will fail (Red X ❌).
6. Click into the failure details and find the specific test that failed (`test_add`). This demonstrates how CI catches bugs before they reach production.

## 🚑 Step 4: Fix the Build

1. Revert the change in `src/calculator.py` to fix the bug:
```python
def add(x, y):
    return x + y

```


2. Commit and push.
3. Verify in the **Actions** tab that the build is Green ✅ again.

---

### **Part 3: Student Exercises**

Once the students have completed the README steps above, assign them these 5 exercises to deepen their understanding.

#### **Exercise 1: Feature Expansion**

* **Task:** Add a new function `power(x, y)` (exponentiation) to `src/calculator.py`.
* **Requirement:** If you push *only* the function, the CI will pass, but the code is untested. You must also write a new test case `test_power` in `tests/test_calculator.py`.
* **Goal:** Verify that the CI runs your *new* test automatically without changing the configuration file.

#### **Exercise 2: The "Strict" Gatekeeper**

* **Task:** Modify the `tests/test_calculator.py` to enforce that the `divide` function returns a float, not an integer (e.g., `10 / 2` should be `5.0`).
* **Goal:** Learn how updating tests can force code updates.

#### **Exercise 3: Linting (Code Quality)**

* **Task:** CI isn't just for testing; it's for style, too. Edit the `.github/workflows/ci.yml` file. Add a step *before* "Run Unit Tests" to install and run `flake8`.
* *Hint command:* `pip install flake8` and `flake8 src/`


* **Goal:** Understand multi-step pipelines.

#### **Exercise 4: Broken Dependency Simulation**

* **Task:** Create a file named `requirements.txt` and add a non-existent package name (e.g., `fakepackage==99.99`). Push the code.
* **Goal:** Observe at which step the pipeline fails. Does it even reach the "Run Unit Tests" stage? (Answer: No, it fails at the "Install dependencies" step).

#### **Exercise 5: Add a Status Badge**

* **Task:** Go to your repository **Actions** tab, select the "Python CI Pipeline," click the "..." menu, and select "Create status badge." Copy the Markdown code and paste it at the very top of your `README.md`.
* **Goal:** Learn how developers communicate build health visually.
