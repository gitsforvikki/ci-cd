# Lesson 18 — Staging → Production Pipeline

## What is a Staging → Production Pipeline?

A production CI/CD pipeline should not normally deploy a new version directly to production without validation.

~~~text
GitHub
   ↓
Jenkins
   ↓
CI: Test / Build
   ↓
Deploy to Staging
   ↓
Test Staging
   ↓
Manual Approval
   ↓
Deploy to Production
   ↓
Verify Production
~~~

**Main idea:** Staging validates the release before production receives it.

---

## 1. Automatic Staging Deployment

After CI succeeds, Jenkins can automatically deploy the new version to staging.

~~~text
Developer
    ↓
GitHub
    ↓
Jenkins
    ↓
Lint / Test / Build
    ↓
Staging Deployment
~~~

For Docker:

~~~text
Jenkins
   ↓
Docker Image
   ↓
Registry
   ↓
Staging Server
   ↓
Container
~~~

Automatic staging gives developers a fast feedback loop.

---

## 2. Testing Staging

After staging deployment, Jenkins should verify that the application actually works.

Deployment success only means the deployment process completed.

Application success means the application is healthy and functional.

~~~text
Deploy
  ↓
Container starts
  ↓
Health Check
  ↓
Smoke Tests
  ↓
PASS
~~~

If an important test fails:

~~~text
Staging Test Failed
        ↓
Stop Pipeline
        ↓
Production NOT Deployed
~~~

---

## 3. Health Checks

A simple health endpoint might be:

~~~http
GET /health
~~~

Expected response:

~~~text
HTTP 200
~~~

Jenkins can verify it:

~~~bash
curl -f https://staging.example.com/health
~~~

If the command fails, Jenkins should stop the pipeline.

A container can be running while the application is not ready:

~~~text
Container = Running
        ↓
Database connection failed
        ↓
Application = Not Ready
~~~

So deployment status alone is not enough.

---

## 4. Smoke Testing Staging

Smoke tests are quick tests that verify the most important functionality.

For a web application:

~~~text
GET /
GET /health
Login
Important API
Critical user flow
~~~

The goal is to answer:

> Is this release healthy enough to continue toward production?

If an important test fails:

~~~text
Staging Test Failed
        ↓
Stop Pipeline
        ↓
Production NOT Deployed
~~~

---

## 5. Manual Approval

For production, many organizations require a human approval step.

~~~text
CI
 ↓
Staging
 ↓
Tests
 ↓
Manual Approval
 ↓
Production
~~~

Jenkins Declarative Pipeline can use an input step:

~~~groovy
stage('Production Approval') {
    steps {
        input message: 'Deploy this release to production?'
    }
}
~~~

A reviewer may verify:

- Staging tests passed
- Product behavior looks correct
- Release is expected
- Deployment window is appropriate
- No known blocking issue exists

Manual approval provides a controlled checkpoint before production.

---

## 6. Production Deployment

After approval, Jenkins deploys the validated release to production.

Preferred flow:

~~~text
Build
  ↓
Artifact / Docker Image
  ↓
Staging
  ↓
Test
  ↓
Approval
  ↓
Production
~~~

Important principle:

**Do not rebuild a different version for production if you can promote the exact artifact/image that was tested in staging.**

For Docker:

~~~text
codebuddy:9ab42ef
       ↓
    Staging
       ↓
    Testing
       ↓
   Production
~~~

This gives strong traceability.

---

## 7. Production Verification

Production deployment is not the final step.

Jenkins should verify the production application after deployment.

Typical checks:

~~~text
Production Deployment
        ↓
Health Check
        ↓
Smoke Test
        ↓
Application Ready
~~~

Checks can include:

- HTTP status
- Health endpoint
- Critical API
- Application logs
- Container status
- Readiness
- Important user flow

Example:

~~~bash
curl -f https://example.com/health
~~~

---

## 8. What If Production Verification Fails?

A production deployment can succeed technically but fail functionally.

~~~text
Production Deploy
       ↓
Health Check
       ↓
    FAIL
       ↓
Rollback
       ↓
Previous Version
~~~

For example:

~~~text
v1 = codebuddy:7f83a91
v2 = codebuddy:9ab42ef

Deploy v2
    ↓
Health Check
    ↓
FAIL
    ↓
Rollback
    ↓
Deploy v1
~~~

The safest rollback strategy is normally to redeploy the previously known-good artifact/image.

---

## 9. Complete Production Pipeline

~~~text
                         GitHub
                            ↓
                         Webhook
                            ↓
                          Jenkins
                            ↓
                    Checkout / CI Tests
                            ↓
                         Build
                            ↓
                  Artifact / Docker Image
                            ↓
                 Automatic Staging Deploy
                            ↓
                    Staging Health Check
                            ↓
                     Staging Smoke Tests
                            ↓
                    ┌───────┴───────┐
                    ↓               ↓
                  FAIL             PASS
                    ↓               ↓
              Stop Pipeline    Manual Approval
                                    ↓
                              Production Deploy
                                    ↓
                              Health Check
                                    ↓
                             Smoke / Verification
                                    ↓
                              ┌─────┴─────┐
                              ↓           ↓
                            PASS         FAIL
                              ↓           ↓
                           Success     Rollback
~~~

This is one of the most important production CI/CD diagrams to remember.

---

## 10. Example Jenkins Pipeline

A simplified Declarative Pipeline:

~~~groovy
pipeline {
    agent any

    stages {
        stage('CI') {
            steps {
                sh 'npm ci'
                sh 'npm run lint'
                sh 'npm test'
                sh 'npm run build'
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh './deploy-staging.sh'
            }
        }

        stage('Test Staging') {
            steps {
                sh 'curl -f https://staging.example.com/health'
                sh './smoke-tests.sh'
            }
        }

        stage('Production Approval') {
            steps {
                input message: 'Deploy this release to production?'
            }
        }

        stage('Deploy to Production') {
            steps {
                sh './deploy-production.sh'
            }
        }

        stage('Verify Production') {
            steps {
                sh 'curl -f https://example.com/health'
                sh './production-smoke-tests.sh'
            }
        }
    }
}
~~~

The commands are examples. A real pipeline should use actual deployment mechanisms, credentials, environments, and verification checks.

---

## 11. Build Once → Deploy Many

Avoid:

~~~text
Build
 ↓
Staging

Build again
 ↓
Production
~~~

Prefer:

~~~text
Build
 ↓
Artifact / Docker Image
 ↓
Staging
 ↓
Test
 ↓
Production
~~~

Why?

Because rebuilding can introduce differences.

The tested artifact should be the artifact that reaches production.

### Mental model

~~~text
Git SHA
   ↓
Jenkins Build
   ↓
Artifact / Image
   ↓
Staging
   ↓
Approval
   ↓
Production
~~~

---

## 12. Staging vs Production

### Staging

Purpose:

- Validate the release
- Run smoke tests
- Test production-like behavior
- Catch deployment/configuration problems

### Production

Purpose:

- Serve real users
- Provide the actual application
- Require stronger safety and monitoring

Simple model:

~~~text
Staging
→ "Is this release safe to promote?"

Production
→ "Serve real users safely."
~~~

Staging should be as production-like as practical while still using appropriate staging resources and data.

---

## 13. What Should Stop Production?

Production deployment should stop if important validation fails.

~~~text
Unit Test Failed
        ↓
     STOP

Build Failed
        ↓
     STOP

Staging Deployment Failed
        ↓
     STOP

Staging Health Check Failed
        ↓
     STOP

Smoke Test Failed
        ↓
     STOP

Approval Rejected
        ↓
     STOP
~~~

This is called a **quality gate** or **promotion gate**.

---

## 14. Automatic vs Manual Deployment

### Automatic staging

~~~text
Merge to main
      ↓
Jenkins
      ↓
CI
      ↓
Staging
~~~

### Manual production approval

~~~text
Staging
  ↓
Testing
  ↓
Human Approval
  ↓
Production
~~~

Some organizations automate production too when they have strong automated tests, monitoring, and rollback.

The important point is:

**The level of automation depends on the organization's risk and delivery model.**

---

## 15. Production Verification Checklist

After deployment, verify:

- Application is reachable
- Health endpoint returns success
- Critical API works
- Container/process is running
- Logs show no immediate critical errors
- Database/dependencies are reachable
- Readiness checks pass
- Important user flow works

Example:

~~~text
Production
   ↓
HTTP 200?
   ↓
Health OK?
   ↓
Smoke Test OK?
   ↓
Logs OK?
   ↓
Release Healthy
~~~

---

## ⭐ Interview Points

### Why deploy to staging before production?

> Staging provides a production-like environment where the release can be validated before exposing it to real users.

### What is a production approval gate?

> It is a controlled checkpoint that requires approval before Jenkins promotes a validated release to production.

### Should production be rebuilt after staging?

> Preferably no. Build the artifact or Docker image once, test it in staging, and promote the same version to production.

### What is the difference between deployment success and application success?

> Deployment success means the deployment commands completed; application success means the deployed application is actually healthy and working.

### What is a smoke test?

> A smoke test is a small set of fast tests that verify critical application functionality after deployment.

### What happens if staging tests fail?

> The pipeline should stop and production deployment should not proceed.

### What happens if production health checks fail?

> The deployment should be investigated and, when appropriate, rolled back to the previous known-good version.

### Why is manual approval useful?

> It creates a controlled production gate where a human can verify the release before it reaches real users.

### What is a quality gate?

> A quality gate is a condition that must pass before the pipeline can move to the next stage.

---

## 🧠 Remember

The most important production pipeline is:

~~~text
GitHub
   ↓
Jenkins
   ↓
CI
   ↓
Build Once
   ↓
Deploy to Staging
   ↓
Health Check
   ↓
Smoke Tests
   ↓
Manual Approval
   ↓
Deploy Same Version to Production
   ↓
Production Verification
   ↓
PASS → Done
FAIL → Rollback
~~~

### Golden rule

**Never think of deployment as the end of the pipeline.**

Think:

**Deploy → Verify → Decide**

And the complete production mental model is:

> **Build once → Test in staging → Approve → Promote the same artifact → Verify production → Roll back if necessary.**
