# Lesson 3 — How does Jenkins know that code has changed?

## Simple idea

Jenkins does not magically know when we push code.

Usually, GitHub **notifies Jenkins** using a **webhook**.

### Basic flow

```
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
```

## What is a webhook?

A webhook is a notification sent from one system to another when something happens.

For example:

> "A new commit was pushed to this repository."

GitHub sends that notification to Jenkins.

GitHub webhooks can be used to trigger CI pipelines when code is pushed. citeturn0search5turn0search1

## What happens step by step?

### 1. Developer pushes code

```bash
git add .
git commit -m "update login"
git push
```

The new commit reaches GitHub.

### 2. GitHub sends a webhook

GitHub sends an HTTP request to the Jenkins webhook endpoint.

```
GitHub
   │
   │  "New push happened!"
   ↓
Jenkins
```

### 3. Jenkins receives the event

Jenkins checks whether the event belongs to a repository/job that should be built.

With the GitHub plugin, a push hook can trigger Jenkins to check the repository for a change and start a build. citeturn0search1turn0search2

### 4. Jenkins starts the pipeline

If the change matches the configured job, Jenkins creates a build and starts the pipeline.

```
Push
 ↓
Webhook
 ↓
Jenkins receives event
 ↓
Change detected
 ↓
Build starts
```

---

## Webhook vs Polling

There are two common ways Jenkins can know about changes.

### Webhook — preferred

GitHub tells Jenkins immediately.

```
GitHub ────────→ Jenkins
       "New push"
```

This is faster and avoids Jenkins repeatedly checking GitHub.

### Polling

Jenkins asks GitHub periodically:

```
Jenkins ───────→ GitHub
        "Anything new?"
```

If Jenkins finds a new commit, it can start a build.

Polling is useful as a fallback, but repeated polling can create unnecessary work. Jenkins documentation recommends push notifications when possible. citeturn0search2turn0search3

---

## Important point

The webhook **does not send your whole project to Jenkins**.

It mainly tells Jenkins:

> "Something changed in this repository."

Jenkins then uses its Git/SCM configuration to find the repository and get the required code.

---

## Complete picture

```
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
    │ run Jenkinsfile
    ↓
 Build → Test → Deploy
```

## Interview answer

**Q: How does Jenkins know that code has changed?**

**Answer:**

> Jenkins can be notified by a GitHub webhook when a push occurs. Jenkins receives the event, checks whether it matches the configured repository/job, and then triggers the pipeline. Jenkins can also use SCM polling, where it periodically checks for changes.

## Remember

**Push → Webhook → Jenkins → Build**

That is the main idea of this lesson.
