# Lesson 11 — Jenkins Credentials & Secrets

## Why do we need Credentials?

CI/CD pipelines often need access to things that must stay private.

Examples:

- GitHub access token
- SSH key
- Database password
- API key
- Docker registry login
- Cloud credentials

These are called **credentials or secrets**.

The most important rule is:

**Never put real secrets directly inside Git code.**

---

## 1. Jenkins Credentials Store

Jenkins provides a secure place called the **Credentials Store**.

Instead of putting a password inside the Jenkinsfile:

```text
❌ Jenkinsfile
    ↓
password123
```

we store the secret in Jenkins:

```text
Jenkins Credentials Store
        ↓
     Secret
        ↓
   Jenkinsfile
```

The Jenkinsfile refers to the credential instead of containing the actual secret.

---

## 2. What is `credentialsId`?

Each Jenkins credential has an ID.

For example:

```text
credentialsId = github-token
```

The Jenkinsfile can use that ID to access the stored credential.

Simple idea:

```text
Jenkinsfile
    ↓
credentialsId
    ↓
Jenkins Credentials Store
    ↓
Actual secret
```

The Jenkinsfile does not need to know the secret value.

---

## 3. Secret Text

Secret text is useful for values such as:

- API keys
- Tokens
- Application secrets

Example:

```groovy
environment {
    API_TOKEN = credentials('api-token')
}
```

Jenkins gets the secret from its Credentials Store.

Important:

Do not write the actual token in the Jenkinsfile.

---

## 4. Username and Password

Some services require a username and password.

For example:

```text
Username → registry-user
Password → ********
```

These can be stored as a Jenkins username/password credential.

The pipeline references the credential instead of storing the password in Git.

---

## 5. SSH Keys

SSH keys are commonly used when Jenkins needs secure SSH access.

For example:

```text
Jenkins Agent
     ↓ SSH
Deployment Server
```

The private key should be stored securely in Jenkins credentials.

Do not commit the private key to Git.

---

## 6. Git Credentials

Jenkins may need credentials when accessing a private Git repository.

Flow:

```text
Jenkins
   ↓
Git Credentials
   ↓
Private Git Repository
   ↓
Checkout code
```

For GitHub, this can involve an appropriate access token or SSH credential depending on the setup.

---

## 7. Docker Registry Credentials

When Jenkins pushes a private Docker image to a registry, it needs authentication.

Example:

```text
Jenkins
   ↓
Docker login
   ↓
Docker Registry
   ↓
Push image
```

The registry username/password or token should be stored in Jenkins Credentials.

Never do this:

```bash
docker login -u myuser -p mypassword
```

with real credentials written directly into the Jenkinsfile or shell script.

---

## 8. Environment Variables

Secrets are often made available to a build through environment variables.

Example:

```text
Jenkins Credentials
        ↓
Environment Variable
        ↓
Application / Command
```

For example:

```bash
echo $API_TOKEN
```

But be careful:

**Never intentionally print secrets into build logs.**

---

## 9. Secret Masking

Jenkins can mask supported credential values in build logs.

For example, instead of showing:

```text
my-real-secret-value
```

the log may show something like:

```text
****
```

This reduces the chance of accidentally exposing a secret.

However, masking is not a reason to handle secrets carelessly.

---

## 10. Why should secrets NOT be committed to Git?

Suppose someone writes:

```javascript
const API_KEY = 'real-secret-key';
```

and commits it.

Even if the line is deleted later, the secret may still exist in Git history or other copies.

Possible consequences:

```text
Secret committed
      ↓
Repository history
      ↓
Secret may be exposed
      ↓
Unauthorized access
```

If a real secret is accidentally exposed, it should be treated as compromised and rotated/revoked.

---

## Complete Credentials Flow

```text
Developer
    ↓
Git repository
    │
    │ Jenkinsfile contains credential ID
    ↓
Jenkins
    ↓
Credentials Store
    ↓
Secret injected when needed
    ↓
Build / Git / Docker / Deployment
```

Notice:

**The source code contains the reference, not the secret itself.**

---

## Example Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        DOCKER_CREDS = credentials('docker-registry')
    }

    stages {
        stage('Login') {
            steps {
                sh 'docker login -u "$DOCKER_CREDS_USR" -p "$DOCKER_CREDS_PSW"'
            }
        }
    }
}
```

The important concept is not memorizing the syntax.

Understand this:

```text
credentials('docker-registry')
            ↓
Jenkins finds credential by ID
            ↓
Credential becomes available to pipeline
```

Also, use safer command patterns where possible and avoid exposing secrets through command output.

---

## ⭐ Interview Points

### Where should Jenkins secrets be stored?
> Jenkins secrets should be stored in the Jenkins Credentials Store rather than committed to source control.

### What is `credentialsId`?
> It is the identifier Jenkins uses to find a stored credential.

### How does Jenkins access a private Git repository?
> Jenkins uses securely stored Git credentials, such as an SSH key or access token.

### Why should secrets not be committed to Git?
> Because they can be exposed through repository history, forks, logs, backups, or other copies.

### What is secret masking?
> It is a Jenkins feature that helps hide supported secret values from build logs.

### What should you do if a secret is accidentally committed?
> Treat it as compromised, revoke or rotate it immediately, remove it from the repository/history where appropriate, and update the Jenkins credential.

---

## 🧠 Remember

```text
Secret
  ↓
Jenkins Credentials Store
  ↓
credentialsId
  ↓
Jenkins Pipeline
  ↓
Use secret
  ↓
Do not expose it
```

**Code tells Jenkins which credential to use.**

**Jenkins stores the actual secret.**

**Git should never be your secret store.**