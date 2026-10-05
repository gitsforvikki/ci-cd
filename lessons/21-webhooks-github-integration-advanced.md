# Lesson 21 — Webhooks & GitHub Integration — Advanced

## What is a Webhook?

A webhook allows GitHub to notify Jenkins when an event happens.

Instead of Jenkins repeatedly asking GitHub:

~~~text
Jenkins → "Any new changes?"
Jenkins → "Any new changes?"
Jenkins → "Any new changes?"
~~~

GitHub can notify Jenkins:

~~~text
Developer
   ↓
GitHub
   ↓
Webhook
   ↓
Jenkins
   ↓
Pipeline
~~~

The webhook is a notification/trigger. Jenkins still needs to obtain the source code for the build.

---

## 1. Push Webhook

A push event happens when code is pushed to a repository.

~~~text
Developer
    ↓
git push
    ↓
GitHub
    ↓
Push Webhook
    ↓
Jenkins
    ↓
CI Pipeline
~~~

A push webhook can trigger checkout, dependency installation, linting, tests, builds, security checks, and staging deployment.

---

## 2. Pull Request Events

A Pull Request (PR) represents a proposed change before it is merged to the target branch.

~~~text
feature/login
      ↓
     PR
      ↓
main
~~~

A PR event can trigger a validation pipeline.

~~~text
Developer
    ↓
Push feature branch
    ↓
GitHub
    ↓
Open / Update PR
    ↓
Jenkins
    ↓
PR Validation
~~~

Typical PR checks:

- Install
- Lint
- Unit tests
- Build
- Security checks

Production deployment should normally not happen just because a PR was opened.

---

## 3. Push vs PR Pipeline

### Push pipeline

~~~text
Push
 ↓
Jenkins
 ↓
CI
 ↓
Build
~~~

### PR validation pipeline

~~~text
PR
 ↓
Jenkins
 ↓
Lint
 ↓
Test
 ↓
Build
 ↓
PR Status
~~~

### Main branch pipeline

~~~text
Merge to main
      ↓
Jenkins
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
~~~

This separation is very important in real CI/CD systems.

---

## 4. Branch Filtering

A repository may contain:

~~~text
main
develop
feature/login
feature/payment
bugfix/cart
release/v1.2
~~~

We usually do not want every branch to perform the same deployment.

For example:

~~~text
feature/*
   ↓
CI only

develop
   ↓
CI + Development

main
   ↓
CI + Staging + Production
~~~

Branch filtering controls which branches trigger which pipeline behavior.

---

## 5. Main/Master Deployment

Many teams protect the main branch.

A common production flow is:

~~~text
Feature Branch
      ↓
Pull Request
      ↓
Review
      ↓
Merge to main
      ↓
Jenkins
      ↓
Production Pipeline
~~~

The important idea is:

**Production deployment is normally associated with a trusted branch such as main.**

Older repositories may use master, but main is common today.

Example Jenkins condition:

~~~groovy
when {
    branch 'main'
}
~~~

This means the stage runs only when the current branch is main.

---

## 6. Why Branch Filtering Matters

Imagine these branches:

~~~text
feature/login
feature/payment
main
~~~

If every branch automatically deployed to production:

~~~text
feature/login
      ↓
Production ❌

feature/payment
      ↓
Production ❌
~~~

This would be dangerous.

Instead:

~~~text
feature/*
   ↓
PR Validation

main
   ↓
Production Pipeline
~~~

Branch filtering creates a deployment boundary.

---

## 7. Multibranch Pipeline

A Jenkins Multibranch Pipeline automatically discovers branches that contain a Jenkinsfile and creates/manages pipelines for those branches.

~~~text
GitHub Repository
       │
       ├── main
       │     └── Jenkinsfile
       │
       ├── develop
       │     └── Jenkinsfile
       │
       ├── feature/login
       │     └── Jenkinsfile
       │
       └── feature/payment
             └── Jenkinsfile
~~~

Jenkins can discover these branches and create branch-specific jobs.

Simple mental model:

**One repository → Multiple branch pipelines**

---

## 8. Why Use Multibranch Pipeline?

Without multibranch support, teams may have to create many Jenkins jobs manually.

~~~text
Job 1 → main
Job 2 → develop
Job 3 → feature/login
Job 4 → feature/payment
~~~

Multibranch can manage these automatically:

~~~text
Multibranch Pipeline
        ↓
   ┌────┼────┐
   ↓    ↓    ↓
 main develop feature/*
~~~

This is especially useful when a repository has many active branches.

---

## 9. PR Validation Pipeline

A PR validation pipeline checks whether a proposed change is safe to merge.

~~~text
Feature Branch
      ↓
Pull Request
      ↓
Jenkins
      ↓
┌──────────────┐
│ Lint         │
│ Unit Tests   │
│ Build        │
│ Security     │
└──────────────┘
      ↓
PR Check
~~~

If everything passes:

~~~text
PR
 ↓
Checks PASS
 ↓
Review
 ↓
Merge
~~~

If a test fails:

~~~text
PR
 ↓
Checks FAIL
 ↓
Developer fixes code
 ↓
Push again
 ↓
Jenkins runs again
~~~

This creates a fast feedback loop.

---

## 10. Build Only Relevant Branches

Not every branch needs the same level of CI/CD.

A practical policy might be:

~~~text
feature/*
    ↓
Lint + Unit Test + Build

develop
    ↓
CI + Development Deployment

main
    ↓
CI + Staging + Production
~~~

This reduces unnecessary deployments and resource usage.

Exact branch policies depend on the team's workflow.

---

## 11. Jenkinsfile and Branch Behavior

The Jenkinsfile defines pipeline logic.

For example:

~~~groovy
pipeline {
    agent any

    stages {

        stage('Test') {
            steps {
                sh 'npm ci'
                sh 'npm test'
            }
        }

        stage('Deploy Staging') {
            when {
                branch 'main'
            }

            steps {
                sh './deploy-staging.sh'
            }
        }
    }
}
~~~

Here:

~~~text
All branches
   ↓
Test

main only
   ↓
Deploy Staging
~~~

This is a simple example of branch-aware pipeline behavior.

---

## 12. Webhook vs Polling

### Polling

Jenkins repeatedly checks GitHub.

~~~text
Jenkins
  ↓
GitHub?
  ↓
Any changes?
  ↓
GitHub
~~~

This can create unnecessary requests and delay detection depending on the polling interval.

### Webhook

GitHub immediately notifies Jenkins.

~~~text
GitHub
  ↓
Webhook
  ↓
Jenkins
~~~

Simple comparison:

~~~text
Polling
→ Jenkins asks GitHub

Webhook
→ GitHub tells Jenkins
~~~

Webhooks are generally more event-driven and efficient for immediate triggering.

---

## 13. Webhook Does Not Replace Checkout

A common misunderstanding is:

> "GitHub sends the whole repository through the webhook."

Usually, the webhook primarily tells Jenkins that an event occurred and provides event metadata.

Jenkins then checks out the relevant source revision.

~~~text
GitHub
  │
  ├── Webhook → Jenkins
  │
  └── Repository
          ↑
       Checkout
          ↑
        Jenkins
~~~

This distinction is important for interviews.

---

## 14. Commit SHA and Reproducibility

A pipeline should know exactly which commit it is building.

Example:

~~~text
Commit:
9ab42ef
    ↓
Jenkins Build
    ↓
Docker Image
codebuddy:9ab42ef
~~~

This allows us to answer:

- Which source created this build?
- Which image is running?
- Which Jenkins build produced it?
- Which version should be rolled back?

Mental model:

~~~text
Git SHA
   ↓
Jenkins Build
   ↓
Artifact / Image
   ↓
Environment
~~~

---

## 15. Multibranch Pipeline + PR Flow

A realistic workflow:

~~~text
Developer
    ↓
Feature Branch
    ↓
Push
    ↓
GitHub
    ↓
PR
    ↓
Jenkins Multibranch
    ↓
PR Validation
    ↓
Lint / Test / Build
    ↓
Review
    ↓
Merge to main
    ↓
Main Pipeline
    ↓
Staging
    ↓
Production
~~~

This is a very common enterprise CI/CD pattern.

---

## 16. Webhook Security

A Jenkins webhook endpoint should not be treated as an unrestricted public endpoint.

Depending on the integration, teams may use:

- Webhook secrets/signatures
- Authentication
- HTTPS
- Reverse proxy/security controls
- Network restrictions

The exact mechanism depends on the Jenkins/GitHub integration.

The goal is to ensure that untrusted requests cannot arbitrarily trigger sensitive pipelines.

---

## 17. Main Branch as Deployment Boundary

Think of the main branch as a trusted promotion boundary.

~~~text
Feature Branch
      ↓
PR
      ↓
Automated Validation
      ↓
Code Review
      ↓
Merge
      ↓
main
      ↓
Deployment Pipeline
~~~

This does not mean main itself is automatically safe.

It means the repository policy can require:

- Required reviews
- Passing checks
- Protected branch rules
- Approved merges

before changes enter the production path.

---

## 18. Complete GitHub → Jenkins Architecture

~~~text
                         GitHub
                            │
              ┌─────────────┴─────────────┐
              │                           │
          Push Event                   PR Event
              │                           │
           Webhook                     Webhook
              │                           │
              └─────────────┬─────────────┘
                            ↓
                    Jenkins Multibranch
                            ↓
                 Relevant Branch Pipeline
                            ↓
                 ┌──────────┴──────────┐
                 ↓                     ↓
             Feature / PR             main
                 ↓                     ↓
            CI Validation       CI + Deployment
                                       ↓
                                    Staging
                                       ↓
                                  Production
~~~

---

## 19. Interview Questions

### What is a webhook?

> A webhook is an HTTP notification sent by GitHub to Jenkins when a configured repository event occurs.

### Webhook vs polling?

> Polling means Jenkins repeatedly asks GitHub for changes. A webhook means GitHub notifies Jenkins when an event occurs.

### What can trigger a Jenkins pipeline?

> Events such as pushes, pull request activity, scheduled triggers, manual execution, or other configured events can trigger pipelines.

### Why use branch filtering?

> To ensure that only relevant branches run specific pipeline stages or deployment actions.

### Why should production deployment usually happen from main?

> Main is commonly treated as a trusted integration branch after review and automated validation, making it a safer deployment boundary.

### What is a Multibranch Pipeline?

> It is a Jenkins pipeline configuration that automatically discovers branches and manages separate pipeline executions for them.

### What is PR validation?

> PR validation runs automated checks such as linting, tests, builds, and security checks before a pull request is merged.

### Does a webhook send the whole source code?

> Normally no. It notifies Jenkins about the event; Jenkins checks out the required source revision from the repository.

### How do you prevent feature branches from deploying to production?

> Use branch filtering and pipeline conditions so production deployment stages run only for the trusted deployment branch, such as main.

### How do you build only relevant branches?

> Configure branch discovery/filtering and use pipeline conditions so each branch runs only the stages appropriate for that workflow.

---

## 🧠 Remember

The most important advanced GitHub/Jenkins flow is:

~~~text
Feature Branch
      ↓
Push / PR
      ↓
GitHub Webhook
      ↓
Jenkins Multibranch
      ↓
PR Validation
      ↓
Review
      ↓
Merge to main
      ↓
Main Branch Pipeline
      ↓
Build / Test
      ↓
Staging
      ↓
Production
~~~

### Golden rules

**1. Webhook = event notification.**

**2. Checkout = Jenkins obtains the source.**

**3. PR pipeline = validate before merge.**

**4. Branch filtering = control what each branch can do.**

**5. main/master can act as the production deployment boundary.**

**6. Multibranch Pipeline = one Jenkins configuration managing multiple branches.**

**7. Always know the exact commit SHA being built.**

Final mental model:

> **GitHub event → Webhook → Jenkins → Relevant branch pipeline → Validate → Merge → Deploy trusted branch.**
