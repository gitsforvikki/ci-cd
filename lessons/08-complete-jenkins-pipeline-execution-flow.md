# Lesson 8 — Complete Jenkins Pipeline Execution Flow ⭐⭐⭐⭐⭐

## The complete picture

Now we connect everything from the previous lessons.

```text
Developer
   ↓
git push / PR merge
   ↓
GitHub
   ↓
Webhook
   ↓
Jenkins Controller
   ↓
Build #105
   ↓
Agent
   ↓
Workspace
   ↓
Checkout code
   ↓
Read Jenkinsfile
   ↓
Stages
   ↓
Steps / Commands
   ↓
Build → Test → Package
   ↓
Deploy
   ↓
Health Check
   ↓
Success / Failure
```

---

## 1. Developer pushes code

The process starts when a developer pushes code to GitHub.

```bash
git push origin main
```

Or a pull request may be merged into the deployment branch.

```text
Developer
   ↓
git push
   ↓
GitHub
```

---

## 2. GitHub sends a webhook

GitHub sends a webhook to Jenkins.

The webhook basically says:

> Something changed in the repository.

Important:

**Webhook does not send the complete source code to Jenkins.**

Jenkins will later get the code using Git checkout.

```text
GitHub
   │
   │ webhook
   ↓
Jenkins
   │
   │ checkout
   ↓
Source code
```

Remember:

**Webhook tells Jenkins something changed.**

**Checkout gets the actual source code.**

---

## 3. Jenkins creates a build

Jenkins receives the event and starts a build.

For example:

```text
Build #101
Build #102
Build #103
Build #104
Build #105 ← current build
```

Every build gets a build number.

This helps us identify and track a particular pipeline execution.

---

## 4. Jenkins selects an Agent

The Controller decides where the pipeline should run.

```text
Jenkins Controller
       ↓
Find available Agent
       ↓
Jenkins Agent
```

The Agent is the machine that actually executes the commands.

For example:

```bash
npm ci
npm test
npm run build
```

These commands run on the Agent, not magically inside the GitHub repository.

---

## 5. Jenkins creates/uses a Workspace

The Agent needs a directory where it can work.

This directory is called the **workspace**.

```text
Agent
  ↓
Workspace
  ↓
Project files
```

---

## 6. Jenkins checks out the code

Jenkins now gets the required source code from Git.

```text
GitHub Repository
       ↓
Git checkout
       ↓
Jenkins Workspace
```

The workspace now contains the project files.

```text
workspace/
├── src/
├── package.json
├── package-lock.json
└── Jenkinsfile
```

---

## 7. Jenkins reads the Jenkinsfile

Jenkins now knows how the pipeline should run.

For example:

```groovy
pipeline {
    agent any

    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
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
```

Jenkins reads the pipeline and starts executing its stages.

---

## 8. Stages → Steps → Commands

This is an important structure to understand:

```text
Pipeline
   ↓
Stages
   ↓
Stage
   ↓
Steps
   ↓
Commands
```

For example:

```text
Stage: Test
    ↓
Step
    ↓
npm test
```

Another example:

```text
Stage: Build
    ↓
Step
    ↓
npm run build
```

---

## 9. Build, Test and Package

A typical application pipeline may look like:

```text
Install dependencies
        ↓
Lint
        ↓
Test
        ↓
Build
        ↓
Package / Artifact
```

For a Node.js or Next.js project, this could be:

```bash
npm ci
npm run lint
npm test
npm run build
```

---

## 10. What is the artifact?

After a successful build, we may have a build output or artifact.

```text
Source Code
    ↓
Build
    ↓
Artifact
```

The important production idea is:

**Build once, deploy the same built version.**

We should avoid rebuilding different code again during production deployment.

---

## 11. Deployment

After the required checks succeed, the pipeline can deploy the application.

Deployment could mean:

- Copying files to a server
- Deploying a Docker container
- Deploying a Docker image
- Starting/restarting the application

Simple flow:

```text
Build
  ↓
Artifact / Image
  ↓
Deployment
  ↓
Server / Production
```

---

## 12. Health Check

After deployment, we should verify that the application is actually working.

For example, Jenkins can call a health endpoint:

```text
Deploy
  ↓
GET /health
  ↓
HTTP 200
  ↓
Deployment successful
```

If the health check fails:

```text
Deploy
  ↓
Health check ❌
  ↓
Deployment failed
```

This becomes especially important in production pipelines.

---

## 13. What happens when something fails?

Suppose the test stage fails:

```text
Install ✅
   ↓
Lint ✅
   ↓
Test ❌
   ↓
Build ⛔
   ↓
Deploy ⛔
```

A critical failed stage normally prevents later stages from running.

Jenkins marks the build as failed.

---

## Complete real-world flow

```text
Developer
    ↓
git push / PR merge
    ↓
GitHub
    ↓
Webhook
    ↓
Jenkins Controller
    ↓
Build #105
    ↓
Agent allocation
    ↓
Workspace
    ↓
Git checkout
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
Artifact / Image
    ↓
Deploy
    ↓
Health check
    ↓
Success ✅
```

---

## Controller vs Agent

Do not confuse these two.

```text
Controller
    ↓
Manages / schedules
    ↓
Agent
    ↓
Executes commands
```

Example:

`sh 'npm test'` runs on the Jenkins Agent.

The Controller coordinates the pipeline but normally does not perform the application build work itself.

---

## Build Time vs Runtime

Another important distinction:

### Build time

Jenkins performs work such as:

```text
Install dependencies
Lint
Test
Build
Create artifact/image
```

### Runtime

The application is already deployed and is serving users.

```text
User
  ↓
Production Server
  ↓
Running Application
```

So:

**Jenkins builds and deploys the application.**

**The production server runs the application.**

---

## ⭐ Interview Answer

### Explain a complete Jenkins pipeline flow.

> A developer pushes code to GitHub. GitHub sends a webhook to Jenkins. Jenkins creates a build and selects an available agent. The agent gets a workspace and checks out the required source code. Jenkins reads the Jenkinsfile and executes its stages and steps, such as installing dependencies, running tests, building the application, and creating an artifact or image. If everything succeeds, Jenkins deploys the built version and performs a health check. The build is then marked successful or failed.

---

## 🧠 Final Mental Model

Remember these seven words:

```text
Trigger
   ↓
Build
   ↓
Agent
   ↓
Workspace
   ↓
Checkout
   ↓
Pipeline
   ↓
Result
```

Or even simpler:

**Something changes → Jenkins starts → Agent gets code → Pipeline runs → Application is deployed → Result is checked.**