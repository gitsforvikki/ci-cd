# Lesson 20 — Rollback

## What is Rollback?

Rollback means returning an application to a previously known-good version after a deployment causes problems.

~~~text
v1 → Production
     ↓
Deploy v2
     ↓
Problem
     ↓
Rollback
     ↓
v1 → Production
~~~

The main idea is:

**If the new release is unhealthy, restore the previous known-good release quickly and safely.**

---

## 1. Why Deployments Fail

A deployment can fail for many reasons.

### Application problems

- Bugs introduced in the new release
- Runtime errors
- Broken API
- Incorrect application configuration

### Environment problems

- Missing environment variables
- Wrong Node.js/runtime version
- Missing system dependency
- Incorrect server configuration

### Infrastructure problems

- Container fails to start
- Server problem
- Network failure
- Insufficient resources

### Dependency problems

- Database unavailable
- External API unavailable
- Incompatible package
- Database schema mismatch

### Deployment problems

- Wrong Docker image
- Incorrect deployment configuration
- Failed health check
- Incorrect port
- Incorrect Kubernetes configuration

Mental model:

~~~text
Deployment Failure
       ↓
Code / Config / Dependency / Infrastructure
       ↓
Investigate
       ↓
Rollback if required
~~~

---

## 2. Previous Version

A rollback requires knowing which version was working before the failed deployment.

Example:

~~~text
Previous:
codebuddy:7f83a91

New:
codebuddy:9ab42ef
~~~

If the new version fails:

~~~text
Production
    ↓
v2 = 9ab42ef
    ↓
FAIL
    ↓
Rollback
    ↓
v1 = 7f83a91
~~~

A good CI/CD system should allow us to identify:

~~~text
Git Commit
    ↓
Jenkins Build
    ↓
Docker Image / Artifact
    ↓
Environment
~~~

---

## 3. Docker Image Rollback

Docker makes rollback straightforward when images are versioned.

Suppose:

~~~text
Current:
codebuddy:9ab42ef

Previous:
codebuddy:7f83a91
~~~

If the current release fails:

~~~bash
docker pull codebuddy:7f83a91
~~~

Then deploy the previous image.

~~~text
Registry
   │
   ├── codebuddy:7f83a91  ← Known good
   │
   └── codebuddy:9ab42ef  ← Failed
                ↓
             Rollback
                ↓
          7f83a91
~~~

### Why immutable tags matter

Avoid relying only on:

~~~text
codebuddy:latest
~~~

Prefer:

~~~text
codebuddy:7f83a91
codebuddy:9ab42ef
~~~

This makes the rollback target explicit.

---

## 4. Git-Based Rollback

Git rollback and deployment rollback are related but not identical.

### Deployment rollback

Return production to a previous built artifact/image.

~~~text
Production v2
     ↓
Deploy previous image
     ↓
Production v1
~~~

### Git revert

Create a new commit that reverses a previous change.

Example:

~~~bash
git revert <commit>
~~~

Flow:

~~~text
Commit A
   ↓
Commit B
   ↓
Bug discovered
   ↓
git revert B
   ↓
New commit C
~~~

Git history becomes:

~~~text
A → B → C
        ↑
      Reverts B
~~~

### Important distinction

**Deployment rollback changes what is running.**

**Git revert changes the source history with a new commit.**

You may use both, but they solve different problems.

---

## 5. Why Rebuilding an Old Commit Is Not Always Ideal

Suppose production currently runs:

~~~text
codebuddy:9ab42ef
~~~

and the previous known-good image is:

~~~text
codebuddy:7f83a91
~~~

The preferred rollback is usually:

~~~text
Pull existing known-good image
        ↓
Deploy it
~~~

rather than:

~~~text
Checkout old commit
        ↓
Build again
        ↓
Deploy
~~~

Why?

The existing image was already built, tested, and identified.

This supports:

**Build once → Deploy many**

and gives better reproducibility.

---

## 6. Database Migration Concerns

Database changes make rollback more complicated.

Example:

~~~text
Application v1
    ↓
Database Schema v1
~~~

Then v2 introduces:

~~~text
Application v2
    ↓
Database Schema v2
~~~

If we rollback only the application:

~~~text
Application v1
       ↓
Database Schema v2
~~~

The old application may not understand the new schema.

This can break production.

Therefore:

**Application rollback does not automatically mean database rollback.**

---

## 7. Expand and Contract Strategy

A safer database migration approach is to make changes backward compatible.

### Step 1 — Expand

Add the new database structure without immediately removing the old structure.

~~~text
Database
 ├── old_column
 └── new_column
~~~

Both application versions can continue working.

### Step 2 — Deploy New Application

~~~text
v1 → v2
~~~

The new application starts using the new structure.

### Step 3 — Migrate Data

Move or backfill data as required.

### Step 4 — Contract

After the old application is no longer needed, remove the old structure.

~~~text
new_column
~~~

Simple model:

~~~text
Expand
  ↓
Deploy
  ↓
Migrate
  ↓
Verify
  ↓
Contract
~~~

This reduces rollback risk.

---

## 8. Safe Rollback Strategy

A safe rollback strategy should include:

### 1. Keep previous versions available

~~~text
Current image
Previous image
Older image
~~~

### 2. Use immutable version tags

~~~text
app:commit-sha
~~~

### 3. Monitor deployment health

~~~text
Deploy
 ↓
Health Check
 ↓
Smoke Test
~~~

### 4. Define rollback conditions

For example:

~~~text
Health Check FAIL
        ↓
Retry
        ↓
Still FAIL
        ↓
Rollback
~~~

### 5. Verify after rollback

Rollback is not complete until the previous version is healthy.

~~~text
Rollback
   ↓
Health Check
   ↓
Smoke Test
   ↓
Recovered
~~~

---

## 9. Manual vs Automated Rollback

### Manual rollback

An engineer decides to rollback.

~~~text
Failure
  ↓
Alert
  ↓
Engineer
  ↓
Rollback
~~~

### Automated rollback

The pipeline or deployment platform automatically rolls back when predefined conditions fail.

~~~text
Deploy
  ↓
Health Check
  ↓
FAIL
  ↓
Automatic Rollback
~~~

Automated rollback is useful when the failure condition is clear and rollback is safe.

---

## 10. Rollback with Blue-Green Deployment

Blue-green deployment makes rollback very fast.

~~~text
Blue = v1
Green = v2
~~~

Initially:

~~~text
Users
  ↓
Blue v1
~~~

Deploy v2 to green:

~~~text
Users
  ↓
Blue v1

Green v2
~~~

After testing:

~~~text
Users
  ↓
Green v2
~~~

If v2 fails:

~~~text
Users
  ↓
Blue v1
~~~

Rollback can simply switch traffic back to blue.

---

## 11. Rollback with Rolling Deployment

With rolling deployment, instances are updated gradually.

~~~text
v1  v1  v1
 ↓
v2  v1  v1
 ↓
v2  v2  v1
~~~

If v2 becomes unhealthy, the deployment system can stop the rollout and replace unhealthy v2 instances with v1.

The exact rollback behavior depends on the deployment platform.

---

## 12. Rollback with Canary Deployment

Canary deployment sends a small amount of traffic to the new version.

~~~text
95% → v1
 5% → v2
~~~

If v2 shows errors:

~~~text
v2 unhealthy
    ↓
Stop rollout
    ↓
Route traffic back to v1
~~~

This limits the impact of a bad release.

---

## 13. Complete Rollback Flow

A production rollback can look like:

~~~text
Developer
    ↓
GitHub
    ↓
Jenkins
    ↓
Build v2
    ↓
Deploy v2
    ↓
Health Check
    ↓
Smoke Test
    ↓
     FAIL
       ↓
Retry / Investigate
       ↓
Rollback to v1
       ↓
Health Check
       ↓
Smoke Test
       ↓
Production Recovered
~~~

This is the key mental model.

---

## 14. Example Jenkins Rollback Concept

A simplified example:

~~~groovy
post {
    failure {
        echo 'Deployment failed - rollback may be required'
        sh './rollback.sh'
    }
}
~~~

A real production pipeline should not blindly rollback every failure.

It should define:

- Which failures trigger rollback
- Which version to restore
- How to verify the rollback
- What happens if rollback itself fails
- Who receives the alert

---

## 15. What If Rollback Fails?

Rollback can also fail.

Example:

~~~text
v2 fails
 ↓
Rollback
 ↓
Rollback fails
 ↓
Production still unhealthy
~~~

At this point:

- Alert the responsible team
- Stop further automated changes if necessary
- Investigate infrastructure/application state
- Restore service using the safest available method
- Follow the incident response procedure

Important lesson:

**Rollback is a recovery mechanism, not a guarantee that every failure will be fixed automatically.**

---

## 16. Rollback vs Roll-forward

### Rollback

Return to a previous known-good version.

~~~text
v2
 ↓
v1
~~~

### Roll-forward

Fix the problem and deploy a new version.

~~~text
v2
 ↓
v3 (fixed)
~~~

Example:

~~~text
v2 has a bug
   ↓
Can safely restore v1?
   ↓
Yes → Rollback
No / fix is ready → Roll-forward
~~~

Both are valid strategies.

---

## ⭐ Interview Points

### What is rollback?

> Rollback is the process of returning production to a previously known-good application version after a failed or unhealthy deployment.

### Why do deployments fail?

> Common causes include application bugs, configuration errors, dependency failures, infrastructure issues, incorrect images, and failed health checks.

### How does Docker help with rollback?

> Versioned Docker images allow us to redeploy a previously known-good image without rebuilding it.

### Why use Git SHA image tags?

> They make each image traceable to an exact source commit and provide reliable rollback targets.

### Deployment rollback vs git revert?

> Deployment rollback changes the version currently running in an environment. Git revert creates a new commit that reverses a previous source-code change.

### Why is database rollback difficult?

> Database schema and data changes can be irreversible or incompatible with an older application version.

### What is the safest rollback strategy?

> Keep immutable versioned artifacts, monitor deployment health, identify a known-good previous version, redeploy it, and verify the rollback.

### Should we always rollback automatically?

> No. Automatic rollback should only be used when failure conditions are well defined and rollback is safe. Some failures require investigation or a roll-forward fix.

### What is build once, deploy many?

> Build an artifact or Docker image once, test that exact version, and promote the same artifact through environments rather than rebuilding it for each environment.

### What is roll-forward?

> Roll-forward means fixing the problem and deploying a new corrected version instead of returning to the previous version.

---

## 🧠 Remember

The most important rollback flow:

~~~text
Deploy New Version
       ↓
Health Check
       ↓
Smoke Test
       ↓
 ┌─────┴─────┐
 ↓           ↓
PASS        FAIL
 ↓           ↓
Success    Retry / Investigate
               ↓
            Rollback
               ↓
        Previous Version
               ↓
          Verify Again
~~~

### Golden rules

**1. Always know what version is running.**

**2. Keep the previous known-good artifact/image available.**

**3. Prefer immutable versioned images over latest.**

**4. Do not assume application rollback also rolls back the database.**

**5. Verify after rollback.**

**6. Rollback quickly when the release is clearly unhealthy, but do not use rollback to hide every transient failure.**

Final mental model:

> **Detect → Decide → Rollback → Verify → Recover**

And remember:

> **Deployment rollback changes the running version; Git revert changes the source history.**
