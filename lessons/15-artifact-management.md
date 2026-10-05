# Lesson 15 — Artifact Management

## What is an Artifact?

An artifact is the output produced by a successful build that we can store and use later.

Simple example:

~~~text
Source Code
    ↓
Build
    ↓
Artifact
    ↓
Deploy
~~~

Examples of artifacts:

- `app.zip`
- `dist/`
- `.next/`
- `.jar`
- Docker image

The exact artifact depends on the application and deployment method.

---

## 1. Why Do We Need Artifacts?

Imagine Jenkins builds our application successfully.

Instead of rebuilding it every time, we can save the build output.

~~~text
Build
  ↓
Artifact
  ↓
Store
  ↓
Deploy
~~~

This gives us a specific version of the application that can be deployed again.

---

## 2. Creating an Artifact

Suppose our build creates a `dist/` directory:

~~~text
Project
  ↓
npm run build
  ↓
dist/
~~~

We can package it:

~~~text
dist/
  ↓
app-v1.zip
~~~

Now `app-v1.zip` is an artifact.

---

## 3. Jenkins archiveArtifacts

Jenkins can store build output using `archiveArtifacts`.

Example:

~~~groovy
post {
    success {
        archiveArtifacts artifacts: 'dist/**', fingerprint: true
    }
}
~~~

This tells Jenkins to archive the files produced by the build.

Simple flow:

~~~text
Build
  ↓
dist/
  ↓
archiveArtifacts
  ↓
Jenkins stores artifact
~~~

The exact path depends on the project.

---

## 4. Artifact Storage

An artifact needs a place where it can be stored.

For a simple Jenkins setup:

~~~text
Jenkins
   ↓
Artifact
   ↓
Jenkins Storage
~~~

In larger systems, artifacts are commonly stored in dedicated artifact repositories or object storage.

The important idea is:

**The build output should be stored somewhere reliable and retrievable.**

---

## 5. Artifact Versioning

We should know exactly which version of an artifact we are deploying.

For example:

~~~text
app-v1.0.0
app-v1.0.1
app-v1.0.2
~~~

Or:

~~~text
app-build-101
app-build-102
app-build-103
~~~

Simple flow:

~~~text
Git Commit
    ↓
Jenkins Build
    ↓
Versioned Artifact
    ↓
Deployment
~~~

Versioning makes it easier to identify and roll back to a previous version.

---

## 6. Artifact vs Docker Image

An artifact is a general term for build output.

A Docker image is a specific packaged application environment.

### Traditional artifact

~~~text
Source Code
    ↓
Build
    ↓
app.zip
    ↓
Server
    ↓
Run Application
~~~

### Docker image

~~~text
Source Code
    ↓
Docker Build
    ↓
Docker Image
    ↓
Registry
    ↓
Server
    ↓
Container
~~~

So:

**Docker image can be treated as a deployable build artifact, but not every artifact is a Docker image.**

---

## 7. Why We Don't Rebuild the Same Code for Production

This is one of the most important CI/CD concepts.

Imagine:

~~~text
Code
  ↓
Build
  ↓
Staging Artifact
  ↓
Test
  ↓
Production
~~~

We want to promote the **same artifact** to production.

Not:

~~~text
Code
  ↓
Build for Staging
  ↓
Staging

Code
  ↓
Build again
  ↓
Production
~~~

Why?

Because two builds can potentially produce different results.

Possible reasons include:

- Dependency changes
- Different build environments
- Different build tools
- Environment differences
- Build-time configuration
- Timestamps or other generated data

Therefore:

**Build once → Test → Promote the same artifact.**

---

## 8. Build Once, Deploy Many Times

The ideal flow is:

~~~text
Git Commit
    ↓
Build
    ↓
Artifact v1.0.0
    ↓
Staging
    ↓
Testing
    ↓
Production
~~~

The artifact does not need to be rebuilt when moving from staging to production, when the application's configuration model supports this approach.

This gives us more confidence that production is running exactly what was tested.

---

## 9. Artifact and Rollback

Artifacts also make rollback easier.

Suppose:

~~~text
Version 1.0.0 → Working
Version 1.0.1 → Deployed
Version 1.0.1 → Problem
~~~

If version 1.0.0 is still available:

~~~text
Production
    ↓
Rollback
    ↓
Artifact v1.0.0
~~~

We can deploy the previous known-good version.

This connects artifact management directly with the rollback lesson.

---

## 10. Complete Artifact Flow

~~~text
Developer
    ↓
GitHub
    ↓
Jenkins
    ↓
npm ci
    ↓
Lint
    ↓
Test
    ↓
Build
    ↓
Artifact
    ↓
Artifact Storage
    ↓
Staging
    ↓
Test
    ↓
Production
~~~

The important part is:

~~~text
Build
  ↓
One Versioned Artifact
  ↓
Staging
  ↓
Production
~~~

---

## ⭐ Interview Points

### What is an artifact?

> An artifact is the output produced by a build that can be stored and used by later stages or deployments.

### Why do we store artifacts?

> To preserve a specific build output so it can be tested, deployed, promoted, or rolled back later.

### What is `archiveArtifacts`?

> It is a Jenkins Pipeline step used to archive files produced by a build.

### Why should artifacts be versioned?

> Versioning lets us identify exactly what was deployed and makes rollback easier.

### Is a Docker image an artifact?

> A Docker image can be treated as a deployable artifact, but an artifact is a broader concept and can also be a ZIP, JAR, build directory, or other build output.

### Why shouldn't we rebuild the application for production?

> We want to deploy the same artifact that was already built and tested, avoiding differences between the staging and production builds.

---

## 🧠 Remember

~~~text
Source Code
    ↓
Build
    ↓
Artifact
    ↓
Store
    ↓
Test
    ↓
Deploy
~~~

The most important rule:

**Build once → Store → Test → Promote the same artifact.**

**Artifact = a specific output of a specific build.**

**Versioned artifacts make deployments traceable and rollback easier.**
