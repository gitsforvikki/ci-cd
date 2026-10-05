# Lesson 25 — CI/CD Security

## 1. Why CI/CD Security Matters

CI/CD pipelines have access to source code, credentials, cloud resources, Docker registries, and production systems.

A compromised pipeline can become a direct path to production.

**Golden rule:**

> Secure the pipeline because the pipeline can access everything needed to deploy the application.

---

## 2. Jenkins Security

Important Jenkins security areas:

- Authentication — who can log in
- Authorization — what each user can do
- Credentials — how secrets are stored
- Agents — where builds execute
- Plugins — what code runs inside Jenkins
- Network security — who can reach Jenkins
- Audit logs — who changed or executed what

Use:

- Strong authentication
- Role-based access control
- HTTPS
- Least privilege
- Regular plugin updates
- Restricted agent permissions

---

## 3. Credentials Security

Never hard-code secrets inside a Jenkinsfile.

❌ Bad:

```groovy
environment {
    API_KEY = 'my-secret-key'
}
```

Use Jenkins Credentials instead:

```groovy
withCredentials([string(
    credentialsId: 'production-api-key',
    variable: 'API_KEY'
)]) {
    sh 'npm run deploy'
}
```

The Jenkinsfile contains only the credential ID, not the actual secret.

---

## 4. Least Privilege

Give users, Jenkins jobs, agents, and credentials only the permissions they actually need.

Example:

A CI job that only builds an application should not have production deployment credentials.

A production deployment job may need:

- Registry pull permission
- Kubernetes deployment permission

It should not automatically receive:

- Database administrator access
- Full cloud administrator access
- Unrelated application secrets

**Interview definition:**

> Least privilege means giving each identity only the minimum permissions required to perform its job.

---

## 5. Secret Management

Common secrets include:

- API keys
- Database passwords
- Cloud credentials
- SSH keys
- Registry credentials
- JWT signing secrets
- Third-party service credentials

Good practices:

- Store secrets outside source code
- Use Jenkins Credentials or an external secret manager
- Never commit `.env` files containing secrets
- Rotate credentials regularly
- Use different credentials for different environments
- Restrict who and what can access production secrets

Mental model:

```text
Code
  ↓
Jenkinsfile
  ↓
Credential ID
  ↓
Jenkins Credentials / Secret Manager
  ↓
Secret available only during required step
```

---

## 6. Dependency Scanning

Third-party dependencies can contain known vulnerabilities.

Example:

```text
package.json
     ↓
npm install
     ↓
Dependency vulnerability scan
     ↓
Pass / Fail
```

For Node.js applications:

```bash
npm audit
```

In production CI, dependency scanning can become a security gate.

Example:

```text
Install
  ↓
Dependency Scan
  ↓
  ├── Safe → Continue
  └── Vulnerable → Fail / Review
```

Do not blindly fail every vulnerability without considering severity, exploitability, and whether the dependency is actually reachable in production.

---

## 7. SAST

**SAST = Static Application Security Testing**

SAST analyzes source code without running the application.

It can detect patterns such as:

- Hard-coded secrets
- SQL injection risks
- Unsafe input handling
- Command injection
- Insecure APIs
- Other coding security issues

Typical flow:

```text
Source Code
    ↓
SAST Scanner
    ↓
Security Findings
    ↓
Pass / Fail / Review
```

SAST is usually performed during CI before deployment.

---

## 8. Container Scanning

If the application is packaged as a Docker image, the image itself must also be scanned.

```text
Source
  ↓
Build Docker Image
  ↓
Container Scan
  ↓
Push to Registry
  ↓
Deploy
```

Container scanning can identify:

- Vulnerable OS packages
- Vulnerable application dependencies
- Known CVEs
- Risky packages

Important principle:

> Scanning source code does not automatically mean the final container image is secure.

---

## 9. OWASP Concepts

OWASP provides widely used application security guidance.

A major reference is the **OWASP Top 10**, which covers common web application security risks.

Important concepts for interviews include:

- Broken access control
- Cryptographic failures
- Injection
- Security misconfiguration
- Vulnerable and outdated components
- Identification and authentication failures

CI/CD security should help detect or prevent these issues before production.

---

## 10. Secure Jenkins Configuration

A production Jenkins setup should consider:

### Authentication

Only authorized users should access Jenkins.

### Authorization

Users should have only the permissions they need.

### HTTPS

Protect credentials and Jenkins traffic in transit.

### Credentials

Use Jenkins Credentials or an external secret manager.

### Plugins

Keep required plugins updated and remove unnecessary plugins.

### Agents

Avoid giving build agents unnecessary access to sensitive systems.

### Network

Do not expose Jenkins directly to the public internet without appropriate protection.

### Backups

Back up important Jenkins configuration and pipeline metadata.

---

## 11. Security Gates in a Pipeline

Security checks can be placed before deployment:

```text
Checkout
   ↓
Install
   ↓
Lint / Test
   ↓
Dependency Scan
   ↓
SAST
   ↓
Build
   ↓
Docker Build
   ↓
Container Scan
   ↓
Push Image
   ↓
Deploy
```

A security gate can stop the pipeline when a defined security policy is violated.

---

## 12. Do Not Put Secrets in Docker Images

Avoid:

```dockerfile
ENV DATABASE_PASSWORD=my-password
```

The secret can become part of the image configuration.

Instead, inject runtime secrets through the deployment environment or secret-management system.

Mental model:

```text
Docker Image
    +
Runtime Configuration
    +
Runtime Secrets
    ↓
Running Application
```

Build the image once and provide environment-specific secrets at runtime.

---

## 13. CI/CD Supply Chain Security

The pipeline depends on many components:

```text
Developer
   ↓
Git Repository
   ↓
Dependencies
   ↓
Build Tools
   ↓
Jenkins Plugins
   ↓
Docker Base Image
   ↓
Container Registry
   ↓
Deployment Platform
```

Every component can become part of the software supply chain.

Important practices:

- Pin important dependency versions
- Use trusted base images
- Scan dependencies and images
- Protect Git repositories
- Protect Jenkins credentials
- Restrict pipeline permissions
- Review pipeline changes
- Keep build tools and plugins updated

---

## 14. Secure Pipeline Example

A simplified secure CI pipeline:

```text
                 GitHub
                    ↓
                 Jenkins
                    ↓
               Checkout
                    ↓
          ┌─────────┴─────────┐
          ↓                   ↓
     Dependency Scan         SAST
          ↓                   ↓
          └─────────┬─────────┘
                    ↓
                  Test
                    ↓
               Docker Build
                    ↓
             Container Scan
                    ↓
              Push Registry
                    ↓
              Deploy Staging
                    ↓
             Approval / Gate
                    ↓
              Deploy Production
```

---

## 15. Security Best Practices

- Never hard-code secrets
- Use least privilege
- Separate staging and production credentials
- Protect the Jenkins controller
- Restrict agent permissions
- Scan dependencies
- Use SAST
- Scan Docker images
- Keep plugins and tools updated
- Use trusted base images
- Protect Git branches
- Review Jenkinsfile changes
- Audit production deployments
- Rotate secrets
- Avoid putting secrets in Docker images
- Fail the pipeline when critical security policies are violated

---

## 16. Interview Questions

### Q1. How do you secure Jenkins?

Use authentication, authorization/RBAC, HTTPS, secure credentials, least privilege, plugin management, restricted agents, network controls, and auditing.

### Q2. How should secrets be stored in Jenkins?

Store them in Jenkins Credentials or an external secret manager and reference them through credential IDs.

### Q3. What is least privilege?

Giving only the minimum permissions required to perform a task.

### Q4. What is SAST?

Static Application Security Testing analyzes source code for security vulnerabilities without executing the application.

### Q5. What is dependency scanning?

It checks third-party dependencies for known vulnerabilities.

### Q6. Why scan Docker images?

Because the final image contains the runtime OS packages and application dependencies that can contain vulnerabilities.

### Q7. Should secrets be stored inside Docker images?

No. Secrets should be injected at runtime.

### Q8. What is a security gate?

A pipeline checkpoint that prevents progression when defined security requirements are not satisfied.

---

## Golden Mental Model

```text
             Secure CI/CD

Git
 ↓
Secure Checkout
 ↓
Dependencies ──→ Dependency Scan
 ↓
Source ─────────→ SAST
 ↓
Build
 ↓
Docker Image ───→ Container Scan
 ↓
Registry
 ↓
Staging
 ↓
Security / Approval Gate
 ↓
Production

Secrets → Credential / Secret Manager
Permissions → Least Privilege
```

### Final Interview Summary

> CI/CD security means protecting the entire software delivery pipeline. Secure Jenkins with authentication, authorization, HTTPS, controlled plugins and agents. Store secrets in a credential or secret-management system, apply least privilege, scan dependencies and source code with SAST, scan container images, and use security gates to prevent vulnerable builds from reaching production.
