# Lesson 12 — Jenkins Plugins

## What is a Jenkins Plugin?

A Jenkins Plugin adds extra functionality to Jenkins.

Jenkins by itself provides the core CI/CD system. Plugins allow Jenkins to work with different tools and services.

Simple idea:

```text
Jenkins
   +
Plugins
   ↓
More capabilities
```

For example:

```text
Git plugin       → Git repositories
Pipeline plugin  → Jenkinsfile pipelines
Credentials      → Secure credentials
Docker plugins   → Docker workflows
SSH plugins      → SSH-based tasks
```

---

## 1. Why are plugins needed?

Imagine Jenkins needs to work with GitHub, Docker, SSH, and many other tools.

Instead of putting every integration directly into Jenkins core, Jenkins uses plugins.

```text
                 Jenkins
                    │
       ┌────────────┼────────────┐
       ↓            ↓            ↓
      Git         Docker        SSH
    Plugin        Plugin       Plugin
```

This makes Jenkins extensible.

---

## 2. Git Plugin

The Git plugin provides Git integration for Jenkins.

It helps Jenkins work with Git repositories and checkout source code.

Flow:

```text
GitHub
   ↓
Git Plugin
   ↓
Jenkins Workspace
```

For example, when Jenkins needs to checkout code from a Git repository, Git-related plugins provide the integration needed for that operation.

---

## 3. Pipeline Plugin

Pipeline functionality allows Jenkins to run pipelines defined as code.

For example:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}
```

The Pipeline ecosystem provides the features needed to execute Jenkinsfiles.

Simple idea:

```text
Jenkinsfile
    ↓
Pipeline functionality
    ↓
Jenkins executes pipeline
```

---

## 4. Credentials Plugin

Credentials-related plugins provide secure handling and integration for credentials stored in Jenkins.

Examples include:

- Username/password
- Secret text
- SSH credentials
- Other credential types depending on installed plugins

Flow:

```text
Jenkinsfile
    ↓
credentialsId
    ↓
Credentials Store
    ↓
Secret
```

Remember from Lesson 11:

**Do not store real secrets in Git.**

---

## 5. Docker Plugins

Jenkins can be integrated with Docker using Docker-related plugins and tools.

Typical CI/CD flow:

```text
Source Code
    ↓
Jenkins
    ↓
docker build
    ↓
Docker Image
    ↓
Registry
```

Depending on the Jenkins setup, Docker may be used directly from the agent, through Docker Pipeline functionality, or through other Docker integrations.

The important concept is:

**Plugins help Jenkins integrate with Docker; the Docker CLI/engine still performs Docker operations.**

---

## 6. SSH-related Plugins

SSH-related plugins can help Jenkins communicate with remote machines.

For example:

```text
Jenkins Agent
      ↓ SSH
Remote Server
      ↓
Application
```

This can be useful for deployments where Jenkins needs to execute commands or transfer files to another server.

SSH credentials should be stored securely in Jenkins.

---

## 7. Blue Ocean / Jenkins UI

Blue Ocean was created to provide a more visual Jenkins pipeline experience.

It can make pipeline stages easier to understand visually.

Example idea:

```text
Build ✓ → Test ✓ → Deploy ✓
```

However, Blue Ocean should not be confused with the core Jenkins pipeline itself.

The important skill for interviews is understanding the Jenkinsfile and pipeline execution, not memorizing a particular UI.

---

## 8. Plugin Management

Plugins are managed from Jenkins administration.

Typical flow:

```text
Jenkins Administration
        ↓
Plugins
        ↓
Install / Update / Remove
```

Before installing a plugin, understand why it is needed.

Do not install plugins just because they are available.

---

## 9. Why too many plugins can be a problem

Plugins are powerful, but they also add complexity.

More plugins can mean:

- More dependencies
- More updates
- More compatibility concerns
- More security exposure
- More maintenance

Simple rule:

**Install only the plugins you actually need.**

---

## 10. Plugin Security

Plugins are part of the Jenkins installation and can have security vulnerabilities.

That means we should:

- Keep Jenkins and plugins updated
- Remove unused plugins
- Use trusted plugin sources
- Review security advisories
- Restrict administrator access
- Test important plugin updates before production use

Think of plugins as software dependencies.

```text
Application
    ↓
Dependencies
    ↓
Security updates

Jenkins
    ↓
Plugins
    ↓
Security updates
```

---

## 11. Plugin Dependencies

Some plugins depend on other plugins.

Example idea:

```text
Plugin A
   ↓
needs Plugin B
   ↓
needs Plugin C
```

Therefore, installing or updating one plugin can affect other plugins.

This is another reason to manage plugins carefully.

---

## 12. Plugin vs Jenkinsfile

This distinction is important.

### Plugin

Provides functionality to Jenkins.

```text
Plugin
  ↓
Jenkins capability
```

### Jenkinsfile

Defines how our pipeline should run.

```text
Jenkinsfile
  ↓
Pipeline instructions
```

Example:

```text
Git Plugin
   ↓
Allows Git integration
   ↓
Jenkinsfile
   ↓
Defines when/how checkout and pipeline stages happen
```

---

## Complete Picture

```text
                    Jenkins
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
      Git           Pipeline       Credentials
     Plugin          Plugins          Plugin
        │              │              │
        ↓              ↓              ↓
     Checkout      Jenkinsfile      Secrets
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
          Docker                SSH
          Plugins              Plugins
```

Plugins provide capabilities.

The Jenkinsfile uses those capabilities to define the pipeline.

---

## ⭐ Interview Points

### What is a Jenkins plugin?
> A Jenkins plugin extends Jenkins with additional functionality or integrations.

### Why are plugins used?
> Plugins allow Jenkins to integrate with tools and services such as Git, Docker, credentials systems, and SSH.

### Why should we avoid unnecessary plugins?
> Extra plugins increase maintenance, dependency, compatibility, and security risks.

### How should plugins be managed?
> Install only required plugins, keep them updated, remove unused plugins, and monitor security advisories.

### Plugin vs Jenkinsfile?
> A plugin provides functionality to Jenkins, while a Jenkinsfile defines how a particular pipeline should execute.

---

## 🧠 Remember

```text
Plugin
  ↓
Adds capability
  ↓
Jenkins
  ↓
Jenkinsfile
  ↓
Uses the capability
  ↓
Pipeline runs
```

**Plugins extend Jenkins.**

**Jenkinsfile defines the pipeline.**

**Use only the plugins you need and keep them secure.**