# Lesson 13 — Build a Real CI Pipeline

## What is a CI Pipeline?

A CI pipeline automatically checks our code whenever we make a change.

For a Node.js / Next.js project, a simple CI pipeline can look like:

~~~text
Developer
   ↓
GitHub
   ↓
Jenkins
   ↓
Install dependencies
   ↓
Lint
   ↓
Tests
   ↓
Build
   ↓
Artifact
   ↓
CI Result
~~~

The main goal is simple:

**Find problems before the code reaches production.**

---

## 1. GitHub + Jenkins

Our code is stored in GitHub.

When we push code:

~~~text
git push
   ↓
GitHub
   ↓
Webhook
   ↓
Jenkins
   ↓
CI Pipeline starts
~~~

Jenkins gets the latest code and starts checking it.

---

## 2. Node.js

For a Node.js or Next.js project, Jenkins needs a Node.js environment.

The agent should have the required Node.js version installed.

~~~text
Jenkins Agent
     ↓
Node.js
     ↓
npm
     ↓
Project
~~~

The same environment should be used consistently so that builds behave predictably.

---

## 3. npm ci

After Jenkins checks out the code, we install dependencies.

For CI, we normally use:

~~~bash
npm ci
~~~

Why?

~~~text
package-lock.json
       ↓
    npm ci
       ↓
Exact locked dependencies
~~~

npm ci is designed for clean, reproducible CI installations.

It uses the lock file and installs the dependencies without modifying the lock file.

---

## 4. ESLint

Next, we can check the code quality.

Example:

~~~bash
npm run lint
~~~

Flow:

~~~text
Source Code
    ↓
ESLint
    ↓
Problems?
 ┌──┴──┐
Yes    No
 ↓      ↓
Fail   Continue
~~~

If linting fails, Jenkins can stop the pipeline.

This prevents code with known linting problems from moving forward.

---

## 5. Tests

After linting, we can run automated tests.

Example:

~~~bash
npm test
~~~

Simple flow:

~~~text
Code
 ↓
Tests
 ↓
Pass? ── No ──→ Pipeline fails
  │
 Yes
  ↓
Next stage
~~~

Tests help us verify that existing functionality still works after a code change.

---

## 6. Build

After the checks pass, Jenkins can build the application.

For a Next.js project:

~~~bash
npm run build
~~~

The build converts our source code into a production-ready application.

~~~text
Source Code
    ↓
npm run build
    ↓
Production Build
~~~

A successful build does not mean the application is already live.

It means the application was successfully prepared for the next step.

---

## 7. Build Artifacts

The build can produce files that we need later.

These files are called **artifacts**.

Simple idea:

~~~text
Source Code
    ↓
Build
    ↓
Artifact
    ↓
Store / use later
~~~

For example, Jenkins can archive selected build output using:

~~~groovy
archiveArtifacts artifacts: 'dist/**'
~~~

The exact artifact path depends on the project.

Remember:

**Artifact = output produced by the build.**

---

## 8. Simple Jenkins Pipeline

A basic example:

~~~groovy
pipeline {
    agent any

    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}
~~~

The stages run in order:

~~~text
Install
  ↓
Lint
  ↓
Test
  ↓
Build
~~~

If an earlier stage fails, later stages normally do not continue.

---

## 9. Pipeline Failure Handling

Failure is an important part of CI.

Example:

~~~text
Install
  ↓
Lint
  ↓
Test ❌
  ↓
Pipeline FAILED
~~~

Jenkins should clearly show:

- Which stage failed
- Why it failed
- Build logs
- Build result

The developer can then fix the problem and push again.

~~~text
Code change
   ↓
Push
   ↓
CI
   ↓
Failure
   ↓
Fix code
   ↓
Push again
   ↓
CI
   ↓
Success
~~~

This is one of the main benefits of CI.

---

## 10. Complete Real CI Flow

~~~text
Developer
    │
    │ git push
    ↓
GitHub
    │
    │ webhook
    ↓
Jenkins
    │
    ↓
Agent
    │
    ↓
Checkout Code
    │
    ↓
npm ci
    │
    ↓
ESLint
    │
    ↓
Tests
    │
    ↓
npm run build
    │
    ↓
Build Artifact
    │
    ↓
CI Result
 ┌──┴──┐
Pass  Fail
 ↓      ↓
Next   Fix code
step      ↓
       Push again
~~~

---

## 11. Why This Pipeline Is Useful

Without CI:

~~~text
Developer
   ↓
Code
   ↓
Manual checking
   ↓
Maybe broken code reaches deployment
~~~

With CI:

~~~text
Developer
   ↓
Push
   ↓
Automated checks
   ↓
Problems found early
   ↓
Only successful code moves forward
~~~

CI gives the team fast feedback.

---

## ⭐ Interview Points

### What is a CI pipeline?

> A CI pipeline automatically builds and validates code whenever changes are made.

### Why use npm ci instead of npm install in CI?

> npm ci uses the lock file for a clean and reproducible dependency installation.

### What happens if tests fail?

> Jenkins marks the build as failed and normally stops the later pipeline stages.

### What is an artifact?

> An artifact is an output produced by a build that can be stored or used by later stages.

### Does a successful build mean the application is live?

> No. A build prepares the application; deployment is a separate step.

---

## 🧠 Remember

~~~text
GitHub
  ↓
Jenkins
  ↓
npm ci
  ↓
Lint
  ↓
Test
  ↓
Build
  ↓
Artifact
  ↓
CI Result
~~~

**CI = Automatically check the code before it moves forward.**

**Build success ≠ Deployment success.**

**The main goal of CI is fast feedback and early failure detection.**
