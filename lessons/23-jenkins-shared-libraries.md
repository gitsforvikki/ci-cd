# Lesson 23 — Shared Libraries

## What are Jenkins Shared Libraries?

A Jenkins Shared Library is reusable pipeline code that can be shared across multiple Jenkinsfiles and repositories.

Instead of putting all CI/CD logic into every Jenkinsfile:

~~~text
Repo A → Large Jenkinsfile
Repo B → Large Jenkinsfile
Repo C → Large Jenkinsfile
~~~

we can move common logic into a shared library:

~~~text
              Shared Library
             /      |      \
            ↓       ↓       ↓
        Repo A    Repo B   Repo C
       Jenkinsfile Jenkinsfile Jenkinsfile
~~~

The main idea:

> **Write common pipeline logic once and reuse it across projects.**

---

## 1. Why Jenkinsfiles Become Large

A simple Jenkinsfile may start small:

~~~groovy
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
    }
}
~~~

But a production pipeline can contain:

- Checkout
- Dependency installation
- Lint
- Tests
- Security scanning
- Docker build
- Image tagging
- Registry login
- Image push
- Staging deployment
- Smoke tests
- Approval
- Production deployment
- Rollback
- Notifications
- Cleanup

It can become difficult to maintain.

~~~text
Small Jenkinsfile
       ↓
More projects
       ↓
More stages
       ↓
Repeated code
       ↓
Large Jenkinsfiles
       ↓
Harder maintenance
~~~

---

## 2. The Problem with Copying Pipeline Code

Suppose five repositories use the same Docker build logic.

Without a shared library:

~~~text
Repo A → Docker logic
Repo B → Docker logic
Repo C → Docker logic
Repo D → Docker logic
Repo E → Docker logic
~~~

If the Docker process changes, engineers may need to update five Jenkinsfiles.

This creates:

- Duplication
- Inconsistent pipelines
- More maintenance
- Higher chance of mistakes

---

## 3. Reusable Pipeline Code

A Shared Library lets us create reusable functions.

Conceptually:

~~~text
Shared Library
      │
      ├── build()
      ├── test()
      ├── dockerBuild()
      ├── dockerPush()
      ├── deploy()
      └── notify()
~~~

Then Jenkinsfiles can call those functions.

Example:

~~~groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                buildApplication()
            }
        }

        stage('Docker') {
            steps {
                buildAndPushDockerImage()
            }
        }
    }
}
~~~

The Jenkinsfile becomes smaller and easier to understand.

---

## 4. Typical Shared Library Structure

A common Shared Library repository can contain:

~~~text
jenkins-shared-library/
│
├── vars/
│   ├── buildApplication.groovy
│   ├── dockerBuild.groovy
│   └── deployApplication.groovy
│
├── src/
│   └── org/company/
│       └── PipelineUtils.groovy
│
└── resources/
    └── templates/
~~~

The exact structure depends on how the organization designs its library.

For interview purposes, remember the three important areas:

- `vars/` → reusable global pipeline steps
- `src/` → more structured Groovy classes
- `resources/` → supporting resources/templates

---

## 5. `vars/` — Reusable Pipeline Steps

A file under `vars/` can expose a reusable pipeline step.

Example:

~~~groovy
// vars/buildApplication.groovy

def call() {
    sh 'npm ci'
    sh 'npm run build'
}
~~~

Then a Jenkinsfile can use:

~~~groovy
stage('Build') {
    steps {
        buildApplication()
    }
}
~~~

The Jenkinsfile does not need to repeat the implementation.

---

## 6. `src/` — Structured Reusable Code

The `src/` directory is useful for more complex Groovy code and classes.

Example concept:

~~~text
src/
└── org/company/
    └── DockerUtils.groovy
~~~

A utility class might contain reusable Docker-related logic.

This is useful when the shared library becomes large and needs proper code organization.

Simple mental model:

~~~text
vars/
→ Simple reusable pipeline steps

src/
→ Reusable classes and structured logic
~~~

---

## 7. `resources/` — Supporting Resources

The `resources/` directory can contain files/templates needed by the library.

Conceptually:

~~~text
resources/
└── templates/
    ├── Dockerfile
    └── deployment.yaml
~~~

The exact resources depend on the organization's implementation.

---

## 8. How a Shared Library Is Used

The organization first makes the library available to Jenkins.

Conceptually:

~~~text
GitHub
   ↓
Jenkins Shared Library
   ↓
Jenkins Configuration
   ↓
Jenkinsfile
~~~

A Jenkinsfile can load a configured library.

Example:

~~~groovy
@Library('company-ci') _

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                buildApplication()
            }
        }
    }
}
~~~

The library name is configured in Jenkins.

---

## 9. Organization-Wide CI/CD Standards

Shared Libraries are especially useful for organizations with many repositories.

Example:

~~~text
                 Company CI/CD Library
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
    Frontend          Backend          Services
      Repo               Repo             Repo
        │                │                │
        └────── Standard Pipeline Rules ──┘
~~~

The organization can standardize:

- Testing
- Security scanning
- Docker builds
- Image tagging
- Registry authentication
- Deployment
- Notifications
- Approval processes
- Compliance checks

This creates consistency across projects.

---

## 10. Example Organization Standard

Suppose every production application must:

~~~text
1. Run tests
2. Run security scan
3. Build Docker image
4. Tag with Git SHA
5. Push to registry
6. Deploy staging
7. Run smoke tests
8. Require approval
9. Deploy production
10. Verify health
~~~

Instead of every team implementing this independently, the Shared Library can provide common functions.

~~~text
              Shared Library
                    ↓
        Standard CI/CD Pipeline
                    ↓
      ┌─────────────┼─────────────┐
      ↓             ↓             ↓
   App A          App B          App C
      ↓             ↓             ↓
  Same standards and controls
~~~

---

## 11. Shared Library vs Jenkinsfile

A useful distinction:

### Jenkinsfile

Defines the pipeline for a specific application.

### Shared Library

Contains reusable pipeline logic used by multiple applications.

~~~text
Jenkinsfile
   ↓
Application-specific configuration
   +
Shared Library
   ↓
Reusable organization-wide logic
~~~

For example:

~~~text
Jenkinsfile:
- app name
- environment choices
- deployment-specific configuration

Shared Library:
- standard test
- standard security scan
- Docker build
- deployment logic
- notifications
~~~

The exact division depends on the organization.

---

## 12. Versioning Shared Libraries

Shared Libraries should be versioned.

For example:

~~~text
company-ci@v1
company-ci@v2
company-ci@v3
~~~

Why?

If the library changes unexpectedly, every application should not suddenly receive breaking behavior.

A repository can intentionally use a known library version.

Conceptually:

~~~text
Application A → Shared Library v1
Application B → Shared Library v2
Application C → Shared Library v2
~~~

This gives teams controlled upgrades.

---

## 13. Centralized Changes

One major advantage is centralized improvement.

Without a library:

~~~text
10 repositories
↓
10 Jenkinsfiles
↓
10 updates
~~~

With a library:

~~~text
Shared Library
      ↓
Common logic updated once
      ↓
Repositories use updated logic
~~~

However, changes should still be tested carefully because a shared library can affect many applications.

---

## 14. Shared Library Governance

A Shared Library becomes important infrastructure, so it should be managed carefully.

Good practices include:

- Code review
- Versioning
- Testing
- Documentation
- Backward compatibility
- Controlled releases
- Clear ownership
- Avoiding unnecessary abstractions

Do not put every tiny shell command into a shared library.

The goal is reusable, meaningful organization-wide behavior.

---

## 15. Shared Library and Security

Shared libraries can contain sensitive deployment logic, so security matters.

The library should:

- Avoid hard-coded secrets
- Use Jenkins Credentials
- Follow least privilege
- Validate inputs
- Avoid unsafe shell construction
- Be reviewed like production code

Example:

~~~text
Shared Library
      ↓
credentialsId
      ↓
Jenkins Credentials Store
      ↓
Secret
~~~

The secret itself should not be stored in the library repository.

---

## 16. Example Real-World Architecture

A larger organization might have:

~~~text
                    GitHub
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
     App Repo       App Repo       App Repo
        │              │              │
        └──────────────┼──────────────┘
                       ↓
               Jenkins Pipelines
                       ↓
              Shared CI/CD Library
                       ↓
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
      Test          Security        Docker
        ↓              ↓              ↓
        └──────────────┼──────────────┘
                       ↓
                    Registry
                       ↓
                   Deployment
~~~

The shared library becomes the common automation layer.

---

## 17. Example: Application Jenkinsfile

A simplified application Jenkinsfile might become:

~~~groovy
@Library('company-ci@v2') _

pipeline {
    agent any

    stages {
        stage('CI') {
            steps {
                runStandardCI()
            }
        }

        stage('Docker') {
            steps {
                buildAndPushImage()
            }
        }

        stage('Deploy') {
            steps {
                deployApplication()
            }
        }
    }
}
~~~

The exact functions are organization-specific.

The important idea is that the Jenkinsfile describes **what the application pipeline needs**, while the shared library provides **how the standard process is implemented**.

---

## 18. Shared Library vs Copy-Paste

### Copy-paste approach

~~~text
App A → Pipeline Code A
App B → Pipeline Code B
App C → Pipeline Code C
~~~

Changes become difficult to manage.

### Shared Library

~~~text
             Shared Library
             /     |     \
            ↓      ↓      ↓
          App A  App B  App C
~~~

Common logic is centralized and reusable.

---

## ⭐ Interview Points

### What is a Jenkins Shared Library?

> A Jenkins Shared Library is a version-controlled collection of reusable pipeline code that can be shared across multiple Jenkinsfiles and repositories.

### Why use Shared Libraries?

> To reduce duplicated pipeline code, standardize CI/CD practices, simplify maintenance, and enforce organization-wide automation standards.

### Why do Jenkinsfiles become large?

> As pipelines gain testing, security, Docker, deployment, approval, rollback, notification, and cleanup logic, repeated implementation details can make Jenkinsfiles difficult to maintain.

### What is `vars/` used for?

> It commonly contains reusable global pipeline steps that can be called from Jenkinsfiles.

### What is `src/` used for?

> It is commonly used for structured reusable Groovy classes and supporting logic.

### What is `resources/` used for?

> It stores supporting resources or templates used by the shared library.

### Shared Library vs Jenkinsfile?

> The Jenkinsfile defines application-specific pipeline behavior, while the Shared Library provides reusable common implementation and organizational standards.

### Why version a Shared Library?

> Versioning prevents unexpected changes from breaking multiple pipelines and allows controlled upgrades.

### Can Shared Libraries contain secrets?

> Secrets should not be hard-coded in the library. Jenkins Credentials or an external secret manager should be used.

### What is the biggest benefit for a large organization?

> Standardization: many repositories can follow the same tested CI/CD process without duplicating the implementation.

---

## 🧠 Remember

The most important mental model:

~~~text
                Shared CI/CD Library
                       ↓
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       App A         App B         App C
          ↓            ↓            ↓
      Jenkinsfile   Jenkinsfile   Jenkinsfile
          ↓            ↓            ↓
          └──── Reusable Logic ─────┘
                       ↓
              Standard CI/CD
~~~

### Golden rules

**1. Jenkinsfile = application pipeline definition.**

**2. Shared Library = reusable pipeline implementation.**

**3. Use Shared Libraries to remove repeated CI/CD logic.**

**4. Version Shared Libraries.**

**5. Test and review shared pipeline code.**

**6. Never hard-code secrets in the library.**

**7. Use Shared Libraries for meaningful reusable organization-wide behavior, not every small command.**

### Final interview-ready summary

> Jenkins Shared Libraries allow organizations to centralize and reuse pipeline logic across many repositories. They reduce Jenkinsfile duplication, improve maintainability, and help enforce consistent CI/CD standards. The common structure uses `vars/` for reusable pipeline steps, `src/` for structured Groovy code, and `resources/` for supporting resources. Shared libraries should be versioned, tested, reviewed, and secured like production code.
