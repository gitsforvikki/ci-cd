# Lesson 02 — Real-World CI/CD Workflow

## 1. How CI/CD Works in a Real Company

A common real-world workflow looks like this:

    Developer
        ↓
    Feature Branch
        ↓
    Pull Request
        ↓
    Code Review
        ↓
    Merge to main
        ↓
    GitHub
        ↓
    Webhook
        ↓
    Jenkins
        ↓
    Pipeline
        ↓
    Build / Test / Deploy

The important idea is that **developers usually don't directly deploy production code**.

The CI/CD pipeline handles the repeatable deployment process.

---

## 2. Developer Creates a Feature Branch

A developer normally works on a separate branch.

Example:

    main
      │
      └── feature/login

The developer writes code and pushes the branch to GitHub.

---

## 3. Pull Request

The developer creates a **Pull Request (PR)**.

A PR allows the team to:

- Review the code
- Discuss changes
- Run CI checks
- Find problems before merging

Example:

    feature/login
          ↓
     Pull Request
          ↓
      Code Review
          ↓
         main

---

## 4. Merge to Main

After the PR is reviewed and approved, it can be merged into the main branch.

The main branch commonly represents code that is ready for the next deployment stage.

---

## 5. GitHub Webhook

After code is pushed or merged, GitHub can notify Jenkins using a **webhook**.

A webhook is simply an HTTP notification.

    GitHub
       │
       │ "Something happened"
       ↓
    Jenkins

For example, GitHub can notify Jenkins that a push happened on the main branch.

### Important

A webhook normally **does not send the complete source code to Jenkins**.

It tells Jenkins that an event happened. Jenkins then checks out the required source code.

---

## 6. Jenkins Starts the Pipeline

Jenkins receives the event and starts the configured job/pipeline.

    GitHub
       ↓
    Webhook
       ↓
    Jenkins
       ↓
    Checkout Code
       ↓
    Run Jenkinsfile

The Jenkinsfile contains the instructions for the pipeline.

---

## 7. Jenkins Gets the Code

Jenkins checks out the required Git revision into a workspace on a Jenkins agent.

    GitHub Repository
           ↓
         Checkout
           ↓
    Jenkins Workspace

The workspace is where Jenkins runs commands such as:

    npm ci
    npm run lint
    npm test
    npm run build

---

## 8. Jenkins Runs the Pipeline Stages

A typical CI pipeline might be:

    Checkout
       ↓
    Install Dependencies
       ↓
    Lint
       ↓
    Test
       ↓
    Build

If one important stage fails, the pipeline normally stops.

For example:

    Checkout ✓
    Install  ✓
    Lint     ✓
    Test     ✗
    Build    ✗
    Deploy   ✗

This prevents broken code from continuing to deployment.

---

## 9. Deployment

If all required checks pass, the pipeline can continue to deployment.

Example:

    Build
      ↓
    Docker Image
      ↓
    Container Registry
      ↓
    Staging
      ↓
    Health Check
      ↓
    Production

The exact deployment process depends on the company's infrastructure.

---

## 10. Complete Workplace Flow

This is the main diagram to remember:

    Developer
        ↓
    Feature Branch
        ↓
    Pull Request
        ↓
    Code Review
        ↓
    Merge to main
        ↓
    GitHub
        ↓
    Webhook
        ↓
    Jenkins Controller
        ↓
    Jenkins Agent
        ↓
    Checkout Code
        ↓
    Jenkinsfile
        ↓
    Install
        ↓
    Lint
        ↓
    Test
        ↓
    Build
        ↓
    Artifact / Docker Image
        ↓
    Staging
        ↓
    Health / Smoke Test
        ↓
    Production

---

## 11. What Does Jenkins Actually Do?

Jenkins mainly **orchestrates the process**.

For example:

    Jenkins
      │
      ├── npm ci
      ├── npm test
      ├── npm run build
      ├── docker build
      ├── docker push
      └── deploy

The tools themselves perform the actual work.

So:

- Git handles source control
- npm runs Node.js commands
- Docker builds containers
- Kubernetes can run/manage containers
- Jenkins coordinates these steps

---

## ⭐ Important Interview Points

### What happens after a developer merges code?

> The Git repository can trigger Jenkins, commonly through a webhook. Jenkins checks out the required revision and runs the pipeline defined by the Jenkinsfile.

### What is a webhook?

> A webhook is an HTTP notification sent by one system to another when an event occurs.

### Does a GitHub webhook send the complete source code?

> Usually no. It notifies Jenkins about the event, and Jenkins then checks out the required source code.

### Why use a feature branch and PR?

> It separates development work from the main branch and provides code review and automated validation before merging.

### What happens if a test fails?

> The pipeline normally stops at that stage, preventing the failed version from moving to later deployment stages.

---

## 🧠 Remember

Think about the workflow in four parts:

    Developer
       ↓
    GitHub
       ↓
    Jenkins
       ↓
    Deployment

And the detailed version:

    Code
     ↓
    PR + Review
     ↓
    Merge
     ↓
    Webhook
     ↓
    Jenkins
     ↓
    Checkout
     ↓
    Test
     ↓
    Build
     ↓
    Deploy

**One-line memory:**

> **Developer pushes → GitHub triggers Jenkins → Jenkins gets the code → pipeline validates/builds → deployment happens.**
