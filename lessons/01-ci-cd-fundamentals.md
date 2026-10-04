# Lesson 1 — Foundations

This lesson builds the basic mental model of CI/CD before we go deeper into Jenkins.

---

## 1. What is Deployment?

**Deployment** means taking a particular version of your application and making it available in an environment where it can run.

Simple example:

```text
Your Code
   ↓
Build
   ↓
Application Version
   ↓
Deploy
   ↓
Server
   ↓
Application is Running
```

Deployment does **not** necessarily mean production. You can deploy to:

- Development
- Staging
- Production

### Easy definition

> **Deployment = making a specific version of an application available to run in an environment.**

---

## 2. What is CI?

**CI = Continuous Integration**

CI means developers frequently push code to a shared repository, and an automated process checks whether the code is still working.

Typical CI:

```text
Developer
    ↓
git push
    ↓
GitHub
    ↓
CI Tool / Jenkins
    ↓
Install Dependencies
    ↓
Lint
    ↓
Test
    ↓
Build
```

The main purpose is to **find problems early**.

For example:

- Tests fail
- Lint fails
- Build fails
- Dependencies are broken
- Code introduces an error

### Easy definition

> **CI = automatically validate code changes before they move further in the delivery process.**

---

## 3. What is CD?

CD can mean:

- **Continuous Delivery**
- **Continuous Deployment**

### Continuous Delivery

The application is automatically built, tested, and prepared for release, but production deployment may require a manual approval.

```text
Code
 ↓
CI
 ↓
Build
 ↓
Test
 ↓
Staging
 ↓
Manual Approval
 ↓
Production
```

### Continuous Deployment

After the automated checks pass, the application is automatically deployed to production.

```text
Code
 ↓
CI
 ↓
Build
 ↓
Test
 ↓
Production
```

### Easy definition

> **CD = automatically deliver or deploy validated software.**

---

## 4. CI vs Continuous Delivery vs Continuous Deployment

The easiest way to remember the difference:

| Concept | Main purpose |
|---|---|
| **CI** | Integrate and validate code |
| **Continuous Delivery** | Keep software ready for release |
| **Continuous Deployment** | Automatically release to production |

Think of the complete flow:

```text
             CI
             ↓
       Build + Test
             ↓
   Continuous Delivery
             ↓
      Ready to Release
             ↓
   Manual Approval
             ↓
        Production

OR

             CI
             ↓
       Build + Test
             ↓
 Continuous Deployment
             ↓
        Production
```

### Interview point

> **Continuous Delivery usually has a release/approval step before production, while Continuous Deployment automatically releases validated changes to production.**

---

## 5. What Happens When You `git push`?

Suppose you change your application and run:

```bash
git add .
git commit -m "Add login feature"
git push origin main
```

A simplified flow is:

```text
Developer
    ↓
git push
    ↓
GitHub Repository
    ↓
Webhook / CI Trigger
    ↓
Jenkins
    ↓
Pipeline Starts
```

Important:

> **git push itself does not deploy your application.**

It only sends your committed code to the remote Git repository.

A CI/CD system can then detect that change and start the pipeline.

---

## 6. What Happens When a PR is Merged?

A common team workflow looks like this:

```text
Developer
    ↓
Feature Branch
    ↓
Pull Request
    ↓
CI Checks
    ↓
Code Review
    ↓
PR Merged
    ↓
main branch changes
    ↓
Jenkins / CI Pipeline
    ↓
Build + Test + Deploy
```

For example:

1. Developer creates a feature branch.
2. Developer opens a Pull Request.
3. CI validates the PR.
4. Reviewers approve it.
5. PR is merged into `main`.
6. The merge changes the `main` branch.
7. Jenkins can detect the change.
8. The production pipeline may start, depending on the configuration.

### Important

A PR merge **can trigger deployment**, but it does not automatically have to.

It depends on the CI/CD pipeline configuration.

---

## 7. What is a Build?

A **build** is the process of converting source code into an application output that can be run, packaged, or deployed.

For example, a Next.js application may use:

```bash
npm run build
```

The build process can:

- Compile code
- Bundle files
- Optimize assets
- Validate the application
- Generate production output

Simple mental model:

```text
Source Code
    ↓
   Build
    ↓
Production-ready Output
```

### Important distinction

**Build is a process.**

The output produced by that process can become an **artifact**.

---

## 8. What is an Artifact?

An **artifact** is a useful output produced by a build that can be stored, transferred, or deployed.

Examples:

```text
application.zip
dist/
.next/
.jar
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

### Remember

```text
Build      = process
Artifact   = output
Deployment = making the version available
```

This distinction becomes very important when we learn artifact management later.

---

## 9. What is a Server?

A **server** is a computer or computing environment that runs services or applications and responds to requests.

For a web application:

```text
User Browser
     ↓
 Internet
     ↓
 Server
     ↓
Your Application
     ↓
Database / Other Services
```

A server can be:

- A physical machine
- A virtual machine
- A cloud server
- A container environment
- A Kubernetes workload/node environment

For example, your Next.js application might eventually run inside a Docker container on a server.

### Easy definition

> **A server is a computing environment that runs a service/application and makes it available to other systems or users.**

---

## 10. What Does "Deploying to Production" Actually Mean?

**Production** is the environment used by real users.

When we say:

> "Deploy the application to production"

we mean that a specific version of the application is placed into the production environment and started so real users can access it.

Example:

```text
Developer
    ↓
GitHub
    ↓
Jenkins
    ↓
Build
    ↓
Artifact / Docker Image
    ↓
Production Server
    ↓
Application Starts
    ↓
example.com
    ↓
Real Users
```

A production deployment may involve:

- Copying an artifact
- Pulling a Docker image
- Starting/replacing a container
- Updating application configuration
- Running database migrations
- Restarting a service
- Running health checks
- Verifying the application

### Important

Deployment is **not simply "uploading files."**

It is the process of making a particular application version **available and operational** in the target environment.

---

# ⭐ Complete Mental Model

Put everything together:

```text
Developer writes code
        ↓
   git commit
        ↓
    git push
        ↓
   GitHub
        ↓
   CI Trigger
        ↓
     Jenkins
        ↓
   Checkout Code
        ↓
      Build
        ↓
      Test
        ↓
    Artifact
        ↓
      Deploy
        ↓
    Production
        ↓
   Real Users
```

---

# ⭐ Interview Points

### What is CI?

> **Continuous Integration is the practice of frequently integrating code changes and automatically validating them through activities such as linting, testing, and building.**

### What is CD?

> **Continuous Delivery/Deployment automates the process of delivering or deploying validated software to environments.**

### Continuous Delivery vs Continuous Deployment?

> **Continuous Delivery keeps software ready for release, usually with a manual production approval. Continuous Deployment automatically releases validated changes to production.**

### Does `git push` deploy the application?

> **No. `git push` sends code to the remote repository. A configured CI/CD pipeline can then build, test, and deploy that code.**

### What is a build?

> **A build is the process of converting source code into a runnable or deployable output.**

### What is an artifact?

> **An artifact is a useful output produced by a build that can be stored, transferred, or deployed.**

### What is a server?

> **A server is a computing environment that runs applications or services and responds to requests.**

### What does production deployment mean?

> **It means making a specific application version available and operational in the production environment for real users.**

---

# 🧠 One-Line Memory

```text
git push
   ↓
Code reaches GitHub
   ↓
CI validates it
   ↓
Build creates output
   ↓
Artifact is produced
   ↓
CD delivers/deploys it
   ↓
Production runs the version
```

> **CI checks the code. CD delivers/deploys the validated software. Deployment makes a specific version available in an environment.**
