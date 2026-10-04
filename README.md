# CI/CD Learning Roadmap

A complete beginner-to-advanced CI/CD learning roadmap focused on Jenkins, GitHub, Docker, production deployment, security, and interview preparation.

---

## Lesson 1 — Foundations

First we'll understand:

1. What is **deployment**?
2. What is **CI**?
3. What is **CD**?
4. CI vs Continuous Delivery vs Continuous Deployment
5. What happens when you `git push`
6. What happens when a PR is merged
7. What is a **build**?
8. What is an **artifact**?
9. What is a **server**?
10. What does "deploying to production" actually mean?

- [Lesson 01 — Foundations](lessons/01-ci-cd-fundamentals.md)

## Lesson 2 — What exactly happens between "code" and "LIVE"?

- [Lesson 02 — What exactly happens between "code" and "LIVE"?](lessons/02-real-world-ci-cd-workflow.md)

## Lesson 3 — How does Jenkins know that code has changed?

- [Lesson 03 — How does Jenkins know that code has changed?](lessons/03-jenkins-fundamentals.md)

## Lesson 4 — What Actually Happens When You Click "Build Now"?

- [Lesson 04 — What Actually Happens When You Click "Build Now"?](lessons/04-jenkins-pipeline.md)

## Lesson 5 — Jenkinsfile & Declarative Pipeline

- [Lesson 05 — Jenkinsfile & Declarative Pipeline](lessons/05-jenkinsfile.md)

## Lesson 6 — Git → Jenkins: How Jenkins Gets Your Code

- [Lesson 06 — Git → Jenkins: How Jenkins Gets Your Code](lessons/06-git-jenkins-how-jenkins-gets-your-code.md)

## Lesson 7 — Jenkins Agent Architecture

- [Lesson 07 — Jenkins Agent Architecture](lessons/07-jenkins-agent-architecture.md)

## 8. Complete Jenkins Pipeline Execution Flow

- Git push → webhook → Jenkins
- Job/build creation
- Agent allocation
- Workspace
- Checkout
- Jenkinsfile execution
- Stages → steps → commands
- Build result

- [Lesson 08 — Complete Jenkins Pipeline Execution Flow](lessons/08-complete-jenkins-pipeline-execution-flow.md)

## 9. Jenkinsfile Deep Dive

- Declarative Pipeline syntax
- `pipeline`
- `agent`
- `stages`
- `stage`
- `steps`
- `environment`
- `parameters`
- `when`
- `post`
- `input`
- `options`
- `tools`
- `triggers`

- [Lesson 09 — Jenkinsfile Deep Dive](lessons/09-jenkinsfile-deep-dive.md)

## 10. Jenkins Pipeline Control Flow

- Stage dependencies
- Parallel stages
- Sequential stages
- Conditional execution
- Manual approvals
- Timeouts
- Retries
- Error handling
- `catchError`
- `try/catch`

- [Lesson 10 — Jenkins Pipeline Control Flow](lessons/10-jenkins-pipeline-control-flow.md)

## 11. Jenkins Credentials & Secrets

- Credentials store
- `credentialsId`
- Secret text
- Username/password
- SSH keys
- Git credentials
- Docker registry credentials
- Environment variables
- Secret masking
- Why secrets shouldn't be committed to Git

- [Lesson 11 — Jenkins Credentials & Secrets](lessons/11-jenkins-credentials-secrets.md)

## 12. Jenkins Plugins

- What plugins are
- Git plugin
- Pipeline plugin
- Credentials plugin
- Docker plugins
- SSH-related plugins
- Blue Ocean / UI considerations
- Plugin management and security

- [Lesson 12 — Jenkins Plugins](lessons/12-jenkins-plugins.md)

---

# Phase 3 — CI/CD With a Real Node/Next.js Project

## 13. Build a Real CI Pipeline

- GitHub
- Jenkins
- Node.js
- `npm ci`
- ESLint
- Tests
- `npm run build`
- Build artifacts
- Pipeline failure handling

- [Lesson 13 — Build a Real CI Pipeline](lessons/13-build-a-real-ci-pipeline.md)

## 14. Environment Management

- Development
- Staging
- Production
- Environment variables
- `.env`
- Jenkins environment variables
- Build-time vs runtime variables
- Secrets per environment

- [Lesson 14 — Environment Management](lessons/14-environment-management.md)

## 15. Artifact Management

- What exactly is an artifact?
- Creating artifacts
- `archiveArtifacts`
- Artifact storage
- Versioning
- Artifact vs Docker image
- Why we don't rebuild the same code for production

- [Lesson 15 — Artifact Management](lessons/15-artifact-management.md)

## 16. Docker + Jenkins

- Docker build inside Jenkins
- Docker image tagging
- Registry
- Docker login
- Push image
- Pull image on server
- Container deployment

- [Lesson 16 — Docker + Jenkins](lessons/16-docker-jenkins.md)

## 17. Deployment Strategies

- SSH deployment
- SCP/rsync
- Docker deployment
- Nginx
- PM2/systemd
- Container deployment
- Deployment scripts
- Restarting applications
- Zero/minimal downtime basics

- [Lesson 17 — Deployment Strategies](lessons/17-deployment-strategies.md)

---

# Phase 4 — Production CI/CD

## 18. Staging → Production Pipeline

- Automatic staging deployment
- Testing staging
- Manual approval
- Production deployment
- Production verification

- [Lesson 18 — Staging → Production Pipeline](lessons/18-staging-production-pipeline.md)

## 19. Health Checks & Deployment Verification

- /health
- HTTP status checks
- Smoke tests
- Deployment verification
- What happens when health check fails
- Automated rollback concepts

- [Lesson 19 — Health Checks & Deployment Verification](lessons/19-health-checks-deployment-verification.md)

## 20. Rollback

- Why deployments fail
- Previous version
- Docker image rollback
- Git-based rollback
- Database migration concerns
- Safe rollback strategy

- [Lesson 20 — Rollback](lessons/20-rollback.md)

## 21. Webhooks & GitHub Integration — Advanced

- Push webhook
- PR events
- Branch filtering
- Main/master deployment
- Multibranch Pipeline
- PR validation pipeline
- Build only relevant branches

- [Lesson 21 — Webhooks & GitHub Integration — Advanced](lessons/21-webhooks-github-integration-advanced.md)

---

# Phase 5 — Advanced Jenkins

## 22. Parallel & Distributed Builds

- Parallel stages
- Multiple agents
- Executors
- Build queues
- Resource management
- Build concurrency

- [Lesson 22 — Parallel & Distributed Builds](lessons/22-parallel-distributed-builds.md)

## 23. Shared Libraries

- Why Jenkinsfiles become large
- Reusable pipeline code
- Jenkins Shared Libraries
- Organization-wide CI/CD standards

- [Lesson 23 — Shared Libraries](lessons/23-jenkins-shared-libraries.md)

## 24. Pipeline Optimization

- Dependency caching
- Docker layer caching
- Workspace reuse
- Parallel execution
- Avoiding unnecessary builds
- Reducing pipeline time

- [Lesson 24 — Pipeline Optimization](lessons/24-pipeline-optimization.md)

## 25. CI/CD Security

- Jenkins security
- Credentials security
- Least privilege
- Secret management
- Dependency scanning
- SAST
- Container scanning
- OWASP concepts
- Secure Jenkins configuration

- [Lesson 25 — CI/CD Security](lessons/25-ci-cd-security.md)

---

# Phase 6 — Enterprise-Level CI/CD

## 26. Production-Grade CI/CD Architecture

- Complete enterprise pipeline
- Multiple environments
- Artifact repositories
- Docker registry
- Jenkins agents
- Approval gates
- Security scanning
- Monitoring
- Logging
- Notifications
- Rollbacks

- [Lesson 26 — Production-Grade CI/CD Architecture](lessons/26-production-grade-ci-cd-architecture.md)

---

## After these 26 lessons

Then we can move into real-world tools and architecture:

- Advanced CI/CD ecosystem
- SonarQube
- SAST / DAST
- Trivy
- Docker Registry
- AWS
- Kubernetes
- Helm
- Terraform
- GitOps
- Argo CD
- Monitoring with Prometheus/Grafana
- Logging
- Blue-Green deployment
- Rolling deployment

---

## Learning Flow

```text
Foundations
    ↓
Jenkins Pipeline Deep Dive
    ↓
Real CI/CD With Node/Next.js
    ↓
Production CI/CD
    ↓
Advanced Jenkins
    ↓
Enterprise-Level CI/CD
    ↓
Advanced CI/CD Ecosystem
```

## Interview Focus

- Deployment, CI, CD
- Git push → webhook → Jenkins
- Build → Artifact → Deployment
- Jenkinsfile / Declarative Pipeline
- Jenkins Controller/Agent architecture
- Credentials & Secrets
- Jenkins Plugins
- Node/Next.js CI pipeline
- Environment management
- Docker + Jenkins
- Deployment strategies
- Staging → Production
- Health checks
- Rollback
- Webhooks & GitHub integration
- Parallel & Distributed Builds
- Shared Libraries
- Pipeline Optimization
- CI/CD Security
- Production-grade CI/CD architecture

> **Important:** This is the original 26-lesson curriculum. The content should be learned lesson-by-lesson in the same simple, beginner-to-advanced style, with practical mental models, important concepts, diagrams when useful, and interview-focused takeaways.
