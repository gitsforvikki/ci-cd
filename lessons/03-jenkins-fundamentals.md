# Lesson 3 — How does Jenkins know that code has changed?

## Simple idea

Jenkins needs a way to know that new code was pushed.

The most common way is a **GitHub webhook**.

A webhook is simply a notification:

> "Something changed in this repository."

### Basic flow

~~~text
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
Pipeline starts
~~~

## What is a webhook?

A webhook is an HTTP notification sent from one system to another when an event happens.

For example:

~~~text
Developer pushes code
        ↓
      GitHub
        ↓
 "New push happened!"
        ↓
      Jenkins
~~~

Jenkins then checks the configured job and starts the pipeline when the event matches.

## What happens step by step?

### 1. Developer pushes code

~~~bash
git add .
git commit -m "update login"
git push
~~~

The new commit goes to GitHub.

### 2. GitHub sends a webhook

GitHub sends a notification to Jenkins.

~~~text
GitHub ─────────→ Jenkins
        webhook
~~~

The webhook tells Jenkins that something changed.

### 3. Jenkins receives the event

Jenkins checks:

- Which repository changed?
- Which branch changed?
- Which Jenkins job should run?

### 4. Jenkins starts the pipeline

If the event matches the job configuration:

~~~text
Push
 ↓
Webhook
 ↓
Jenkins
 ↓
Change detected
 ↓
Pipeline starts
~~~

## Webhook vs Polling

There are two common ways Jenkins can detect changes.

### Webhook

GitHub tells Jenkins immediately.

~~~text
GitHub ─────────→ Jenkins
        "New push"
~~~

This is usually preferred because Jenkins does not need to keep asking GitHub.

### Polling

Jenkins checks GitHub periodically.

~~~text
Jenkins ─────────→ GitHub
        "Anything new?"
~~~

If Jenkins finds a new commit, it can start a build.

### Simple difference

~~~text
Webhook:
GitHub → Jenkins

Polling:
Jenkins → GitHub
~~~

## Important point

The webhook does **not** send the complete project to Jenkins.

It mainly tells Jenkins:

> "A change happened."

Jenkins then uses Git/SCM to get the code.

## Complete picture

~~~text
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
    │ checkout code
    ↓
 Workspace
    │
    │ Jenkinsfile
    ↓
 Build → Test → Deploy
~~~

## Interview answer

**Q: How does Jenkins know that code has changed?**

**Answer:**

> Jenkins can be notified by a GitHub webhook when a push occurs. Jenkins receives the event, checks the configured job and branch, and then starts the pipeline. Jenkins can also use polling to periodically check for changes.

## Remember

**Push → Webhook → Jenkins → Pipeline**

That is the main idea of this lesson.
