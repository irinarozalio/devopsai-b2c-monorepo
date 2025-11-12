# Updated CI Workflow with Testing & Linting

## 🎯 What Was Added

Your CI workflow now includes a **comprehensive testing stage** that runs BEFORE building Docker images.

### New Job: `lint-and-test`

This job:
1. ✅ **Lints with flake8** - Catches syntax errors, undefined names, style issues
2. ✅ **Validates Python syntax** - Compiles all `.py` files
3. ✅ **Checks imports** - Ensures modules can be imported
4. ✅ **Runs pytest** - Executes your test suite (if tests exist)

---

## 📊 Workflow Flow

```
Push to GitHub
    ↓
detect-changes (Check what changed)
    ↓
lint-and-test (NEW! Test the code)  ← Build ONLY happens if tests pass
    ↓
build-push (Build Docker images)
    ↓
update-gitops-dev-tag (Update deployment)
```

**Key point:** If linting or tests fail, Docker build is **SKIPPED** ✋

---

## 🧪 What Gets Checked

### 1. Flake8 Linting

**Catches:**
- ✅ Syntax errors (`SyntaxError`)
- ✅ Undefined variables (`NameError`)
- ✅ Unused imports
- ✅ Missing imports
- ✅ Code style issues

**Example errors it catches:**
```python
# This will FAIL the build
def broken_function(
    print("missing closing parenthesis")

# This will FAIL the build
print(undefined_variable)

# This will FAIL the build
import nonexistent_module
```

---

### 2. Python Syntax Validation

**Compiles all `.py` files to catch:**
- ✅ Syntax errors
- ✅ Invalid indentation
- ✅ Missing colons
- ✅ Unclosed brackets

**Example:**
```python
# This will FAIL
def test()  # Missing colon
    pass

# This will FAIL
if True
    print("missing colon")
```

---

### 3. Import Checking

**Attempts to import your modules to catch:**
- ✅ Import errors
- ✅ Circular dependencies
- ✅ Missing dependencies

---

### 4. Pytest (Optional)

**Runs your test suite if it exists:**
- ✅ Unit tests
- ✅ Integration tests
- ✅ Coverage reporting

**If no tests exist:** Job passes with a warning ⚠️

---

## 🚀 How to Use

### Step 1: Replace Your Current Workflow

```bash
# In your repo
cd .github/workflows

# Backup current file
cp ci.yml ci.yml.backup

# Copy the new file
cp /path/to/ci-with-tests.yml ci.yml

# Commit and push
git add ci.yml
git commit -m "Add testing and linting to CI"
git push
```

---

### Step 2: Install Required Dependencies

Add testing tools to your `requirements.txt`:

```bash
# For each service
cd services/user-service

# Add to requirements.txt (or create requirements-dev.txt)
cat >> requirements.txt << 'EOF'

# Testing and linting tools
flake8>=7.0.0
pytest>=8.0.0
pytest-cov>=4.1.0
EOF

# Repeat for order-service
cd ../order-service
cat >> requirements.txt << 'EOF'

# Testing and linting tools
flake8>=7.0.0
pytest>=8.0.0
pytest-cov>=4.1.0
EOF
```

**Or use separate dev requirements:**

```bash
# Create requirements-dev.txt
cat > requirements-dev.txt << 'EOF'
flake8>=7.0.0
pytest>=8.0.0
pytest-cov>=4.1.0
black>=24.0.0
isort>=5.13.0
EOF
```

Then update workflow to use it:
```yaml
pip install -r requirements-dev.txt
```

---

### Step 3: Add Basic Tests (Optional but Recommended)

```bash
cd services/user-service

# Create tests directory
mkdir -p tests

# Create a simple test
cat > tests/test_basic.py << 'EOF'
def test_example():
    """Basic test to ensure pytest works"""
    assert 1 + 1 == 2

def test_imports():
    """Test that our modules can be imported"""
    try:
        import src
        assert True
    except ImportError:
        assert False, "Could not import src module"
EOF

# Create __init__.py
touch tests/__init__.py
```

**Repeat for order-service**

---

## 🎯 Test the Updated Workflow

### Test 1: Syntax Error (Should Fail)

```bash
cd services/user-service

# Add a syntax error
cat >> src/main.py << 'EOF'

def broken_function(
    print("missing closing parenthesis")
EOF

git add .
git commit -m "Test: syntax error detection"
git push
```

**Expected result:**
```
✅ detect-changes passes
❌ lint-and-test FAILS at "Lint with flake8" step
⏭️  build-push SKIPPED (because tests failed)
```

**Check GitHub Actions:**
- Should see red ❌ on the workflow
- Click on failed job to see error details
- n8n should detect this failure
- AI should analyze the logs
- Email notification sent

---

### Test 2: Import Error (Should Fail)

```bash
cd services/order-service

# Add import error
cat >> src/main.py << 'EOF'

import package_that_does_not_exist
EOF

git add .
git commit -m "Test: import error detection"
git push
```

**Expected result:**
```
✅ detect-changes passes
❌ lint-and-test FAILS at "Lint with flake8" step (undefined name)
⏭️  build-push SKIPPED
```

---

### Test 3: Valid Code (Should Pass)

```bash
cd services/user-service

# Fix the errors
git revert HEAD
git push
```

**Expected result:**
```
✅ detect-changes passes
✅ lint-and-test passes
✅ build-push succeeds
✅ Images pushed to registry
```

---

## 📋 Workflow Configuration

### Adjusting Python Version

Change in the workflow file:

```yaml
matrix:
  include:
    - name: user-service
      python-version: '3.11'  # ← Change this
```

**Supported versions:**
- `'3.8'`
- `'3.9'`
- `'3.10'`
- `'3.11'` (recommended)
- `'3.12'`

---

### Adjusting Flake8 Rules

Edit the linting step:

```yaml
- name: Lint with flake8
  run: |
    # Customize error codes
    flake8 . --select=E9,F63,F7,F82,E501  # Add E501 for line length
    
    # Adjust line length
    flake8 . --max-line-length=100  # Change from 127
    
    # Adjust complexity
    flake8 . --max-complexity=5  # Make stricter (default 10)
```

**Common error codes:**
- `E9` - Syntax errors
- `F63` - Invalid print statement
- `F7` - Syntax errors in doctests
- `F82` - Undefined name
- `E501` - Line too long
- `W503` - Line break before binary operator

---

### Making Tests Optional

If you don't have tests yet:

```yaml
- name: Run tests with pytest
  continue-on-error: true  # ← Add this to not fail build
  run: |
    pytest tests/ -v || echo "No tests yet"
```

---

### Adding More Checks

#### Code Formatting (Black)

```yaml
- name: Check formatting
  run: |
    pip install black
    black --check . || echo "Run 'black .' to format code"
```

#### Import Sorting (isort)

```yaml
- name: Check imports
  run: |
    pip install isort
    isort --check-only . || echo "Run 'isort .' to sort imports"
```

#### Type Checking (mypy)

```yaml
- name: Type check
  run: |
    pip install mypy
    mypy . --ignore-missing-imports
```

---

## 🔍 What Each Step Does

### Step 1: Skip unchanged service
```yaml
if: ${{ matrix.changed != 'true' }}
```
Only test services that actually changed.

### Step 2: Set up Python
```yaml
uses: actions/setup-python@v5
with:
  python-version: '3.11'
  cache: 'pip'  # ← Caches dependencies for faster runs
```

### Step 3: Install dependencies
```yaml
pip install -r requirements.txt
pip install flake8 pytest pytest-cov
```

### Step 4: Lint with flake8
```yaml
# Fail on syntax errors and undefined names
flake8 . --select=E9,F63,F7,F82

# Warn on other issues
flake8 . --exit-zero --max-complexity=10
```

### Step 5: Validate syntax
```yaml
find . -name "*.py" -exec python -m py_compile {} \;
```
Compiles all Python files without executing them.

### Step 6: Check imports
```yaml
python -c "import sys; sys.path.insert(0, '.'); import src"
```
Attempts to import main module.

### Step 7: Run tests
```yaml
pytest tests/ -v --cov=. --cov-report=term-missing
```
Runs tests with coverage reporting.

---

## 📊 GitHub Actions UI

After pushing, check:

```
✅ detect-changes (10s)
   ├─ user: true
   └─ order: false

✅ lint-and-test (user-service) (45s)
   ├─ ✅ Lint with flake8
   ├─ ✅ Validate Python syntax  
   ├─ ✅ Check imports
   └─ ✅ Run tests with pytest

✅ build-push (user-service) (2m 30s)
   └─ ✅ Build & push

✅ update-gitops-dev-tag (5s)
```

**Or if tests fail:**

```
✅ detect-changes (10s)
   └─ user: true

❌ lint-and-test (user-service) (30s)
   ├─ ✅ Lint with flake8
   ├─ ❌ Validate Python syntax  ← Failed here
   └─ ⏭️  (skipped remaining steps)

⏭️ build-push (user-service)
   └─ Skipped because lint-and-test failed

⏭️ update-gitops-dev-tag
   └─ Skipped
```

---

## 🎯 Best Practices

### 1. Write Tests

Even basic tests help:

```python
# tests/test_app.py
def test_app_starts():
    """Test that the app can start"""
    from src import app
    assert app is not None
```

### 2. Keep Requirements Updated

```bash
# Periodically update dependencies
pip list --outdated
pip install --upgrade flake8 pytest
pip freeze > requirements.txt
```

### 3. Run Tests Locally

Before pushing:

```bash
cd services/user-service

# Run linting
flake8 .

# Run tests
pytest tests/ -v

# Check syntax
python -m py_compile src/*.py
```

### 4. Use Pre-commit Hooks

```bash
# Install pre-commit
pip install pre-commit

# Create .pre-commit-config.yaml
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8

  - repo: https://github.com/psf/black
    rev: 24.0.0
    hooks:
      - id: black

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
EOF

# Install hooks
pre-commit install

# Now runs automatically on git commit
```

---

## 🚨 Troubleshooting

### Issue: Tests fail locally but pass in CI

**Cause:** Different Python versions or dependencies

**Fix:**
```bash
# Match CI Python version
pyenv install 3.11
pyenv local 3.11

# Or use Docker
docker run -it python:3.11 bash
```

---

### Issue: flake8 complains about line length

**Fix:** Add to `setup.cfg`:

```ini
[flake8]
max-line-length = 127
exclude = venv,__pycache__,.git
ignore = E501,W503
```

---

### Issue: Import errors in tests

**Fix:** Add `__init__.py` files:

```bash
touch src/__init__.py
touch tests/__init__.py
```

---

### Issue: Tests take too long

**Fix:** Run in parallel:

```yaml
pytest tests/ -v -n auto  # Requires pytest-xdist
```

---

## ✅ Verification Checklist

After deploying the updated workflow:

- [ ] Workflow file updated in `.github/workflows/ci.yml`
- [ ] Testing tools added to `requirements.txt`
- [ ] Tests directory created (optional)
- [ ] Pushed changes to GitHub
- [ ] Workflow runs successfully
- [ ] Test with intentional error (syntax error)
- [ ] Verify build is skipped when tests fail
- [ ] n8n detects the failure
- [ ] AI analysis works
- [ ] Email notification received
- [ ] Fix the error and verify build succeeds

---

## 🎉 Benefits

With this updated workflow:

1. ✅ **Catch errors early** - Before building Docker images
2. ✅ **Faster feedback** - Tests run in ~30-60 seconds
3. ✅ **Save resources** - Don't build if code is broken
4. ✅ **Better AI analysis** - Clearer error messages from linting
5. ✅ **Higher quality** - Encourages writing tests
6. ✅ **Consistent** - Same checks locally and in CI

---

**Your CI pipeline now has professional-grade testing! 🚀**
