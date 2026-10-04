# CI/CD Learning

Interview-focused notes and practical revision for CI/CD, Jenkins, Docker, GitHub webhooks, security, Kubernetes, rollback, and production troubleshooting.

## Learning Roadmap

### Section 1 — CI/CD & Jenkins Fundamentals

1. CI/CD Fundamentals & Core Concepts
2. Real-World CI/CD Workflow
3. Jenkins Architecture & Fundamentals
4. Jenkins Pipeline Control Flow
5. Jenkins Credentials & Secrets
6. Jenkins Plugins
7. Building a Real CI Pipeline
8. Environment Management & Environment Variables
9. Artifact Management
10. Docker + Jenkins Integration
11. Deployment Strategies
12. Staging → Production Deployment
13. Health Checks & Deployment Verification
14. Jenkins Parameters & Manual Controls
15. Pipeline Notifications & Post Actions
16. Pipeline as Code & Jenkinsfile
17. Declarative vs Scripted Pipeline
18. Parallelism, Retry & Pipeline Error Handling
19. CI/CD Pipeline Optimization & Practical Patterns

### Section 2 — Advanced CI/CD

20. Rollback ⭐⭐⭐⭐⭐
21. Webhooks & GitHub Integration — Advanced ⭐⭐⭐⭐
22. Jenkins + Docker Production Pipeline ⭐⭐⭐⭐⭐
23. Advanced Deployment & Release Management
24. Security in CI/CD ⭐⭐⭐⭐⭐
25. CI/CD Reliability & Production Practices

### Skipped

26. Complete Production CI/CD Architecture — SKIPPED

### Section 3 — Advanced Production Concepts

27. Advanced Jenkins Pipeline Design
28. CI/CD Monitoring & Observability
29. Advanced Deployment Automation
30. Kubernetes + Jenkins ⭐⭐⭐⭐⭐
31. Advanced Kubernetes Deployment Concepts
32. Container Orchestration & Scaling
33. Production Infrastructure Concepts
34. Advanced CI/CD Architecture & Integration

### Section 4 — Final Revision

35. CI/CD Best Practices & Troubleshooting ⭐⭐⭐⭐⭐

## Core CI/CD Flow

```text
Developer
   ↓
GitHub
   ↓
Pull Request / Code Review
   ↓
Merge to main
   ↓
Webhook
   ↓
Jenkins
   ↓
Checkout → Install → Lint → Test → Security → Build
   ↓
Docker Image
   ↓
Container Registry
   ↓
Staging
   ↓
Health / Smoke Tests
   ↓
Production
   ↓
Kubernetes
   ↓
Monitor
   ↓
Rollback if needed
```

## Key Interview Concepts

- CI vs CD
- Jenkins Controller vs Agent
- Jenkinsfile / Pipeline as Code
- Webhooks vs Polling
- Build vs Artifact vs Deployment
- Docker Image vs Container vs Registry
- Build Once, Deploy Many
- Secrets and Least Privilege
- Kubernetes Pod / Deployment / Service / Ingress
- Health Checks and Readiness/Liveness
- Rollback strategies
- CI/CD troubleshooting and root-cause analysis

## Repository Goal

A long-term learning and interview revision reference. Each lesson should contain theory, important concepts, practical examples, commands/code, diagrams, common mistakes, interview questions, and quick revision notes.

> Lesson 26 is intentionally skipped as part of the learning path.
