# Lesson 2 — What exactly happens between "code" and "LIVE"?

In the previous lesson, we learned the basic meaning of CI/CD.

Now let's understand the complete journey of code from a developer's machine to a LIVE application.

> **Code does not directly go from GitHub to production. Multiple steps happen in between.**

---

## 1. The Big Picture

A simplified real-world flow looks like this:

    Developer
        ↓
    git push
        ↓
    GitHub
        ↓
    Jenkins Trigger
        ↓
    Checkout Code
        ↓
    Install Dependencies
        ↓
    Test
        ↓
    Build
        ↓
    Package / Artifact
        ↓
    Deploy
        ↓
    Production
        ↓
    LIVE

Every step has a specific purpose.

---

## 2. Developer Writes Code

A developer works on the application locally.

    Developer
       ↓
    React / Next.js Code
       ↓
    Local Testing

The developer may run commands such as npm run dev, npm test, and npm run build.

Once the change is ready, the developer commits it.

---

## 3. Developer Pushes Code

The developer pushes the commit to GitHub:

    git push origin main

Now the code exists in the remote Git repository.

    Developer
        ↓
    git push
        ↓
    GitHub

### Important

**git push does not mean the application is LIVE.**

It only sends the committed code to the remote repository.

---

## 4. Jenkins Gets Triggered

Jenkins needs to know that something changed in GitHub.

A common method is a **webhook**.

    GitHub
       │
       │  "A push happened"
       ↓
    Jenkins

The webhook tells Jenkins that an event occurred.

It does not normally send the complete source code. Jenkins then obtains the required code from GitHub.

---

## 5. Jenkins Gets the Code

Jenkins checks out the required Git revision.

    GitHub Repository
           ↓
         Checkout
           ↓
    Jenkins Workspace

The workspace is where Jenkins runs the commands required by the pipeline.

For example:

    npm ci
    npm test
    npm run build

---

## 6. Install Dependencies

Before the application can be built or tested, its dependencies need to be available.

For a Node.js project, Jenkins commonly runs:

    npm ci

This installs dependencies based on the lock file.

    package.json
    package-lock.json
           ↓
          npm ci
           ↓
       node_modules

---

## 7. Run Tests

Jenkins runs automated checks.

    Code
     ↓
    Tests
     ↓
    PASS / FAIL

If tests fail:

    Test ❌
      ↓
    Pipeline stops
      ↓
    No Deployment

This is one of the main benefits of CI/CD:

> **Broken code can be detected before it reaches production.**

---

## 8. Build the Application

If the tests pass, Jenkins can build the application.

For example:

    npm run build

The build converts source code into production-ready output.

    Source Code
        ↓
       Build
        ↓
    Production Output

Examples:

    Next.js → .next/
    Frontend → dist/
    Java → .jar
    Application → .zip
    Docker → Docker image

---

## 9. Create / Store the Artifact

The build output can become an **artifact**.

    Source Code
         ↓
        Build
         ↓
      Artifact

An artifact gives us a specific version that can be transferred or deployed.

For example:

    my-app
    version: 9ab42ef

This becomes especially important when we learn artifact management, Docker images, versioning, and rollbacks.

---

## 10. Deploy

Now the validated application version is deployed to an environment.

    Artifact / Docker Image
              ↓
           Deployment
              ↓
            Server
              ↓
          Application

The deployment method depends on the infrastructure.

It could use:

- SSH
- SCP/rsync
- Docker
- Kubernetes
- Cloud platforms
- Deployment scripts

---

## 11. Application Becomes LIVE

After deployment, the application starts running in the target environment.

For a web application:

    User
     ↓
    Internet
     ↓
    Server
     ↓
    Application
     ↓
    Database

The user can then access the application through its domain.

    example.com
         ↓
    Production Server
         ↓
    Next.js Application
         ↓
        LIVE

---

# ⭐ Complete Build-to-LIVE Flow

This is the most important diagram from this lesson:

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
    Checkout
        ↓
    Install Dependencies
        ↓
    Test
        ↓
    Build
        ↓
    Package / Artifact
        ↓
    Deploy
        ↓
    Production
        ↓
    Health / Smoke Check
        ↓
        LIVE

---

## 12. Build Time vs Runtime

This is an important concept.

### Build time

Jenkins is preparing the application.

    Source Code
        ↓
      Install
        ↓
       Test
        ↓
      Build
        ↓
     Artifact

### Runtime

The application is actually serving users.

    User
     ↓
    Internet
     ↓
    Server
     ↓
    Application
     ↓
    Database

Think of it as:

    BUILD TIME
    Code → Build → Artifact

    RUNTIME
    User → Server → Application → Database

---

## 13. What if Something Fails?

CI/CD is designed to stop when an important step fails.

Example:

    Checkout     ✓
    Install      ✓
    Test         ✓
    Build        ✗
    Deploy       ✗

Because the build failed, deployment does not continue.

Another example:

    Build        ✓
    Deploy       ✓
    Health Check ✗

The deployment may be considered unsuccessful. Later we will learn rollback strategies for this situation.

---

# ⭐ Interview Points

### Does code go directly from GitHub to production?

> **No. Code normally passes through several stages such as checkout, dependency installation, testing, building, artifact creation, deployment, and verification.**

### What happens after git push?

> **The code reaches the Git repository. A configured webhook or other trigger can notify Jenkins, which then checks out the required revision and starts the pipeline.**

### Why do we build the application?

> **The build converts source code into production-ready output that can be packaged, stored, or deployed.**

### Why do we create an artifact?

> **An artifact gives us a specific build output that can be stored, transferred, deployed, and potentially used for rollback.**

### What happens if a test fails?

> **The pipeline normally stops at the failed stage and prevents that version from moving to deployment.**

### What is the difference between build time and runtime?

> **Build time is when the application is prepared; runtime is when the deployed application is actually running and serving users.**

---

# 🧠 Remember

Think about the journey in three simple parts:

    CODE
     ↓
    CI
     ↓
    BUILD
     ↓
    ARTIFACT
     ↓
    DEPLOY
     ↓
    LIVE

Or the real-world version:

    Developer
        ↓
    GitHub
        ↓
    Jenkins
        ↓
    Checkout
        ↓
    Install
        ↓
    Test
        ↓
    Build
        ↓
    Artifact
        ↓
    Deploy
        ↓
    Production
        ↓
        LIVE

> **The key idea: Code is only the starting point. CI/CD is the automated journey that takes validated code and turns it into a running production application.**
