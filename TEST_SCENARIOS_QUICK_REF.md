# Quick Test Scenarios Reference

## 🎯 Creating Build Failures for AI Analysis

Now that your CI includes testing, here are quick ways to trigger different types of failures:

---

## 1️⃣ Syntax Error (Easiest)

```bash
cd services/user-service
echo "def broken(" >> src/main.py
git add . && git commit -m "Test: syntax error" && git push
```

**What happens:**
- ❌ Fails at: `Lint with flake8`
- Error: `SyntaxError: invalid syntax`
- AI Analysis: ✅ Will analyze flake8 output

**Fix:**
```bash
git revert HEAD && git push
```

---

## 2️⃣ Undefined Variable

```bash
cd services/order-service
cat >> src/main.py << 'EOF'

print(undefined_variable)
EOF
git add . && git commit -m "Test: undefined variable" && git push
```

**What happens:**
- ❌ Fails at: `Lint with flake8`
- Error: `F821 undefined name 'undefined_variable'`
- AI Analysis: ✅ Will identify undefined variable

---

## 3️⃣ Import Error

```bash
cd services/user-service
echo "import nonexistent_package" >> src/main.py
git add . && git commit -m "Test: import error" && git push
```

**What happens:**
- ❌ Fails at: `Lint with flake8`
- Error: `F401 'nonexistent_package' imported but unused`
- AI Analysis: ✅ Will suggest installing package

---

## 4️⃣ Missing Dependency

```bash
cd services/order-service
echo "requests>=2.31.0" >> requirements.txt
cat >> src/main.py << 'EOF'

import requests
print(requests.get('https://api.github.com').status_code)
EOF
git add . && git commit -m "Test: missing dependency" && git push
```

**What happens:**
- ❌ Fails at: `Install dependencies`
- Error: Package installation fails
- AI Analysis: ✅ Will analyze pip error

**Fix:**
```bash
# If requests is actually needed
pip install requests
pip freeze > requirements.txt
git add . && git commit -m "Fix: add requests" && git push
```

---

## 5️⃣ Failed Test

```bash
cd services/user-service
mkdir -p tests
cat > tests/test_fail.py << 'EOF'
def test_intentional_failure():
    """This test will fail"""
    assert False, "Intentional test failure for AI analysis"
EOF
git add . && git commit -m "Test: failing test" && git push
```

**What happens:**
- ❌ Fails at: `Run tests with pytest`
- Error: `AssertionError: Intentional test failure`
- AI Analysis: ✅ Will analyze pytest output

---

## 6️⃣ Indentation Error

```bash
cd services/order-service
cat >> src/main.py << 'EOF'

def test():
print("wrong indentation")
EOF
git add . && git commit -m "Test: indentation error" && git push
```

**What happens:**
- ❌ Fails at: `Validate Python syntax`
- Error: `IndentationError: expected an indented block`
- AI Analysis: ✅ Will identify indentation issue

---

## 7️⃣ Type Error (Runtime)

```bash
cd services/user-service
cat >> src/main.py << 'EOF'

def add_numbers():
    return "hello" + 5  # TypeError
    
add_numbers()  # Call it so it runs during import check
EOF
git add . && git commit -m "Test: type error" && git push
```

**What happens:**
- ❌ Fails at: `Check imports`
- Error: `TypeError: can only concatenate str to str`
- AI Analysis: ✅ Will identify type mismatch

---

## 🎨 Complex Scenarios

### Multiple Errors

```bash
cd services/user-service
cat >> src/main.py << 'EOF'

# Error 1: Syntax error
def broken(

# Error 2: Undefined variable
print(undefined_var)

# Error 3: Wrong import
import fake_package
EOF
git add . && git commit -m "Test: multiple errors" && git push
```

**What happens:**
- ❌ Fails at first error (syntax)
- AI Analysis: ✅ Will prioritize most critical error

---

### Realistic Database Error

```bash
cd services/order-service
cat >> src/database.py << 'EOF'
import psycopg2

# This will fail if run
conn = psycopg2.connect(
    host='nonexistent-db.example.com',
    database='orders',
    user='admin',
    password='password'
)
EOF

# Import it to trigger error
echo "from . import database" >> src/__init__.py
git add . && git commit -m "Test: database connection" && git push
```

**What happens:**
- ❌ Fails at: `Check imports`
- Error: Connection error or import error
- AI Analysis: ✅ Will suggest database configuration

---

## 📊 Expected AI Analysis Examples

### For Syntax Error:
```
Root Cause: Python syntax error on line 45. Missing closing 
parenthesis in function definition.

Summary: The build failed due to invalid Python syntax. The 
function definition is incomplete.

Suggestion: Add the missing closing parenthesis:
def broken():
    pass

Code Diff:
- def broken(
+ def broken():
+     pass
```

### For Import Error:
```
Root Cause: Module 'nonexistent_package' not found. The package 
is not installed or doesn't exist.

Summary: Import error - Python cannot find the specified module.

Suggestion: 
1. If the package exists, add it to requirements.txt:
   nonexistent_package>=1.0.0
   
2. If it's a typo, correct the import statement

3. If it's a local module, ensure the path is correct
```

### For Undefined Variable:
```
Root Cause: Variable 'undefined_variable' is referenced before 
being defined.

Summary: NameError - attempting to use a variable that hasn't 
been declared.

Suggestion: Define the variable before using it:
undefined_variable = "value"
print(undefined_variable)
```

---

## 🔄 Complete Test Cycle

```bash
#!/bin/bash
# test-ci-monitoring.sh

echo "🧪 Testing CI/CD monitoring with AI analysis"

cd services/user-service

# 1. Create syntax error
echo "Step 1: Creating syntax error..."
echo "def broken(" >> src/main.py
git add . && git commit -m "Test: syntax error" && git push

echo "⏳ Waiting for GitHub Actions to fail (~2 min)..."
sleep 120

echo "⏳ Waiting for n8n to detect and analyze (~5 min)..."
sleep 300

echo "📧 Check your email for AI analysis!"
echo ""

# 2. Fix the error
echo "Step 2: Fixing error..."
git revert HEAD --no-edit
git push

echo "⏳ Waiting for build to succeed (~3 min)..."
sleep 180

echo "✅ Test complete!"
echo ""
echo "Verify:"
echo "1. Received email about syntax error"
echo "2. AI analysis identified the issue"
echo "3. Database has the failure logged"
echo "4. Second build succeeded"
```

**Usage:**
```bash
chmod +x test-ci-monitoring.sh
./test-ci-monitoring.sh
```

---

## 🎯 Verification Checklist

After triggering a failure:

### GitHub Actions:
- [ ] Workflow run shows red ❌
- [ ] Failed job is `lint-and-test`
- [ ] Error message is clear
- [ ] `build-push` was skipped

### n8n:
- [ ] Workflow execution triggered
- [ ] "Fetch Job Logs" retrieved error logs
- [ ] "Trigger AI Analysis" completed
- [ ] "Send Email Notification" sent email

### PostgreSQL:
```sql
SELECT service, status, root_cause, summary, suggestion
FROM builds
WHERE status = 'failure'
ORDER BY timestamp DESC
LIMIT 1;
```
- [ ] Row exists with failure
- [ ] `root_cause` is populated
- [ ] `summary` explains the error
- [ ] `suggestion` provides fix

### Email:
- [ ] Received email notification
- [ ] Subject: "🔴 Build Failed: [service] (master)"
- [ ] Contains job name and URL
- [ ] AI analysis section present
- [ ] Root cause identified
- [ ] Suggested fix provided

### Google Sheets:
- [ ] New row added
- [ ] Status is "failure"
- [ ] Summary and suggestion populated

---

## 🚀 Quick Commands

```bash
# View recent workflow runs
gh run list --limit 5

# Watch a specific run
gh run watch

# View logs for failed run
gh run view --log-failed

# Re-run failed jobs
gh run rerun --failed

# Cancel running workflow
gh run cancel

# Check current CI status
gh run list --workflow=ci.yml --limit 1
```

---

## 📋 Maintenance

### Clean up test errors:

```bash
# Revert all test commits
git log --oneline | grep "Test:" | cut -d' ' -f1 | xargs git revert --no-edit

# Or reset to specific commit
git reset --hard <commit-hash>
git push --force
```

### Clean up database:

```sql
-- Remove test failures
DELETE FROM builds WHERE summary LIKE '%Test:%';

-- Reset state to process everything again
UPDATE workflow_state SET last_checked_run_id = 0;
```

---

**Now you can easily create various test failures and watch the AI analyze them! 🎉**
