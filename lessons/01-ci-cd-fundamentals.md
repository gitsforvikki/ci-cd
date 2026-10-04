# Lesson 01 — CI/CD Fundamentals

## 1. What is CI/CD?

CI/CD is a way to **automate the process of building, testing, and delivering software**.

Instead of doing everything manually, tools such as Jenkins can run these steps automatically.

---

## 2. What is CI?

**CI = Continuous Integration**

It means developers regularly integrate their code into a shared repository, and an automated process checks whether the new code works.

Typical CI flow:

```text
Developer
   ↓
Push Code
   ↓
Git Repository
   ↓
Jenkins / CI Tool
   ↓
Install Dependencies
   ↓
Lint
   ↓
Test
   ↓
Build
```

### Main goal of CI

Find problems **early**, before they reach production.

For example:

- Code has a syntax error
- Tests fail
- Build fails
- Dependencies are broken
- Linting fails

---

## 3. What is CD?

CD means **Continuous Delivery** or **Continuous Deployment**, depending on how the organization uses the term.

### Continuous Delivery

The application is automatically prepared for release, but production deployment may require manual approval.

```text
Code
 ↓
CI
 ↓
Build
 ↓
Staging
 ↓
Approval
 ↓
Production
```

### Continuous Deployment

The validated application is automatically deployed to production.

```text
Code
 ↓
CI
 ↓
Build
 ↓
Production
```

### Easy way to remember

> **CI checks the code. CD delivers the code.**

---

## 4. What is a Build?

A **build** converts source code into something that can be used or deployed.

For a Node.js/Next.js application, for example:

```bash
npm run build
```

The build may generate output such as:

```text
dist/
.next/
```

The exact output depends on the technology.

---

## 5. What is an Artifact?

An **artifact** is a useful output produced by the build that can be stored or deployed.

Examples:

```text
dist/
.next/
application.zip
.jar file
Docker image
```

Simple flow:

```text
Source Code
    ↓
   Build
    ↓
 Artifact
    ↓
 Deployment
```

### Important distinction

**Build ≠ Artifact ≠ Deployment**

- Build = process
- Artifact = output
- Deployment = making the version available in an environment

---

## 6. What is an Environment?

An environment is a place where the application runs.

Common environments:

```text
Development
     ↓
   Staging
     ↓
 Production
```

They can have different:

- Databases
- API URLs
- Environment variables
- Credentials
- Domains
- External services

For example:

```text
Development → localhost
Staging     → staging.example.com
Production  → example.com
```

---

## 7. What is Deployment?

**Deployment** means making a particular application version available in an environment.

Example:

```text
Docker Image
codebuddy:9ab42ef
       ↓
   Production
       ↓
Application Running
```

Deployment can happen using:

- Docker
- SSH
- Kubernetes
- Cloud platforms
- Other deployment systems

---

## 8. What is a Pipeline?

A **pipeline** is a sequence of automated steps used to build, test, and deploy an application.

Example:

```text
Checkout
   ↓
Install
   ↓
Lint
   ↓
Test
   ↓
Build
   ↓
Deploy
```

Jenkins can automate this entire process.

---

## 9. What is Jenkins?

**Jenkins is an automation server** commonly used to create and run CI/CD pipelines.

Jenkins can:

- Get code from Git
- Run tests
- Run builds
- Build Docker images
- Push images to a registry
- Deploy applications
- Run health checks
- Trigger rollback workflows

Important:

> Jenkins is mainly the **orchestrator**. It tells other tools/commands what to execute.

For example:

```text
Jenkins
  ↓
npm
  ↓
Tests / Build

Jenkins
  ↓
Docker
  ↓
Docker Image

Jenkins
  ↓
kubectl
  ↓
Kubernetes
```

---

## 10. Complete Basic CI/CD Flow

The most important diagram from this lesson:

```text
Developer
    ↓
Git Repository
    ↓
     CI
    ↓
Checkout
    ↓
Install
    ↓
Lint
    ↓
Test
    ↓
Build
    ↓
 Artifact
    ↓
   CD
    ↓
 Staging
    ↓
Production
```

---

## ⭐ Important Interview Points

### What is CI?

> Continuous Integration is the practice of frequently integrating code changes and automatically validating them through steps such as build, lint, and tests.

### What is CD?

> Continuous Delivery/Deployment automates the process of delivering or deploying validated software to environments.

### CI vs CD?

```text
CI
↓
Validate code

CD
↓
Deliver / Deploy code
```

### What is an artifact?

> A build output that can be stored, transferred, or deployed.

### What is a pipeline?

> A sequence of automated steps used to validate, build, and deploy software.

### What is Jenkins?

> Jenkins is an automation server used to orchestrate CI/CD pipelines.

---

## 🧠 Remember

```text
CODE
 ↓
CI
 ↓
BUILD
 ↓
ARTIFACT
 ↓
CD
 ↓
DEPLOYMENT
 ↓
ENVIRONMENT
```

**One-line memory:**

> **CI validates the code; CD delivers/deploys the validated software.**
