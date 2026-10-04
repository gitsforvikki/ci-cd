# Lesson 6 — Git → Jenkins: How Jenkins Gets Your Code

## The main idea

Jenkins needs the project's source code before it can build, test, or deploy it.

The basic flow is:

```text
Developer
   ↓
git push
   ↓
GitHub
   ↓
Jenkins is triggered
   ↓
Jenkins connects to GitHub
   ↓
Jenkins checks out the code
   ↓
Code is available in workspace
   ↓
Pipeline continues
```

---

## 1. Where does Jenkins get the code?

Usually, our code is stored in a Git repository such as GitHub.

Example:

```text
GitHub Repository
       │
       ├── src/
       ├── package.json
       └── Jenkinsfile
```

Jenkins does not magically have this code.

It must **clone/fetch the repository** and check out the required revision.

---

## 2. How does Jenkins know which repository to use?

A Jenkins job/pipeline contains Git repository information.

For example:

```text
Repository
    ↓
GitHub URL
    ↓
Branch
    ↓
Credentials (if required)
```

Jenkins uses this information to connect to the repository.

---

## 3. What happens after Jenkins is triggered?

Suppose a developer pushes code:

```bash
git push origin main
```

The flow is:

```text
Developer
    ↓
git push
    ↓
GitHub
    ↓
Webhook
    ↓
Jenkins
    ↓
Build starts
    ↓
Git checkout
    ↓
Workspace contains project code
```

Now Jenkins can run commands such as:

```bash
npm ci
npm test
npm run build
```

---

## 4. What is checkout?

**Checkout** means getting the required version of the source code from Git.

Jenkins may:

- Clone the repository for a new workspace.
- Fetch new changes.
- Checkout a particular branch.
- Checkout a particular commit.

Simple idea:

```text
GitHub
   ↓
Git checkout
   ↓
Jenkins Workspace
```

The **workspace** is the directory on the Jenkins agent where the project files are available while the build runs.

---

## 5. Does the webhook send the source code?

**No.**

This is very important.

The webhook mainly **notifies Jenkins that an event happened**.

For example:

```text
GitHub
   │
   │ "A push happened!"
   ↓
Jenkins
   │
   │ "I need to get the code."
   ↓
Git checkout
   ↓
Workspace
```

So:

**Webhook = notification**

**Git checkout = gets the source code**

---

## 6. What if the repository is private?

Jenkins needs permission to access the repository.

Credentials can be configured in Jenkins.

Examples:

- SSH key
- Username/password or token
- GitHub access token

The important rule is:

> Do not put Git credentials directly inside the Jenkinsfile.

Instead, store them securely in Jenkins Credentials.

---

## 7. Which code does Jenkins checkout?

Jenkins should build the correct revision.

For example:

```text
main
 │
 ├── commit A
 ├── commit B
 └── commit C  ← latest commit
```

If the pipeline is triggered for commit C, Jenkins should checkout the revision associated with that build.

This makes the build reproducible and helps us know exactly which code was tested.

---

## Complete flow

```text
Developer
    │
    │ git push
    ↓
GitHub
    │
    │ webhook
    ↓
Jenkins
    │
    │ create/start build
    ↓
Jenkins Agent
    │
    │ Git checkout
    ↓
Workspace
    │
    ├── source code
    ├── package.json
    └── Jenkinsfile
    │
    ↓
Install → Test → Build → Deploy
```

---

## ⭐ Interview Points

### How does Jenkins get code from GitHub?

> Jenkins connects to the configured Git repository and performs a Git checkout to get the required source code into the agent workspace.

### Does a webhook send the source code to Jenkins?

> No. A webhook notifies Jenkins about a Git event. Jenkins then connects to the repository and checks out the required code.

### What is a Jenkins workspace?

> A workspace is the directory on the Jenkins agent where the source code and other files are available during a build.

### How does Jenkins access a private repository?

> Jenkins uses securely stored Git credentials, such as an SSH key or access token.

---

## 🧠 Remember

```text
Webhook
   ↓
"Something changed"

Git Checkout
   ↓
"Get the code"

Workspace
   ↓
"Code is ready"

Pipeline
   ↓
"Build / Test / Deploy"
```

**Webhook tells Jenkins.**

**Git checkout gets the code.**

**Workspace holds the code.**
