# Lesson 19 — Health Checks & Deployment Verification

## What is Deployment Verification?

A deployment is not successful just because Jenkins completed the deployment commands.

We must verify that the application is actually running and responding correctly.

~~~text
Deploy
  ↓
Health Check
  ↓
Smoke Test
  ↓
Deployment Verification
  ↓
PASS → Continue
FAIL → Rollback / Investigate
~~~

The main idea is:

**Deploy → Verify → Decide**

---

## 1. /health Endpoint

A health endpoint is a simple endpoint that tells us whether the application is running.

Example in Express:

~~~js
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'ok'
  });
});
~~~

Request:

~~~http
GET /health
~~~

Expected response:

~~~text
HTTP 200
{
  "status": "ok"
}
~~~

The endpoint should be lightweight and fast.

---

## 2. HTTP Status Checks

Jenkins or another monitoring system can check the HTTP response.

Example:

~~~bash
curl -f https://example.com/health
~~~

If the server returns a successful response:

~~~text
HTTP 200
   ↓
Health Check PASS
~~~

If it returns an error:

~~~text
HTTP 500
   ↓
Health Check FAIL
~~~

The -f option makes curl fail for HTTP error responses, allowing the CI/CD pipeline to detect the failure.

---

## 3. Health Check vs Smoke Test

### Health Check

Usually answers:

> Is the application alive and ready?

Example:

~~~text
GET /health
~~~

### Smoke Test

Checks important application functionality.

Example:

~~~text
GET /
Login
Important API
Critical user flow
~~~

Simple comparison:

~~~text
Health Check
→ Is the application healthy?

Smoke Test
→ Does important functionality work?
~~~

A production deployment may use both.

---

## 4. Deployment Verification

After deployment, Jenkins can perform several checks.

~~~text
Deployment
    ↓
Container / Process Running
    ↓
HTTP Health Check
    ↓
Smoke Tests
    ↓
Logs / Readiness
    ↓
Deployment Verified
~~~

Possible verification checks:

- Application process is running
- Container is running
- Health endpoint returns HTTP 200
- Critical API responds
- Smoke tests pass
- Readiness check passes
- No immediate critical errors in logs

---

## 5. Why Deployment Success Is Not Enough

Imagine Jenkins runs:

~~~bash
docker run ...
~~~

and the command succeeds.

That only tells us the container was started.

It does not guarantee:

- The application is listening on the expected port
- The database is reachable
- Required environment variables are present
- The application can serve requests
- The application is ready for traffic

Therefore:

~~~text
Deployment Command Success
          ≠
Application Success
~~~

We need verification.

---

## 6. What Happens When Health Check Fails?

Suppose version 2 is deployed:

~~~text
Deploy v2
   ↓
Health Check
   ↓
FAIL
~~~

The pipeline should not blindly continue.

Possible flow:

~~~text
Health Check FAIL
       ↓
Stop Promotion
       ↓
Investigate
       ↓
Rollback if appropriate
       ↓
Deploy Previous Version
       ↓
Verify Again
~~~

Example:

~~~text
v1 = codebuddy:7f83a91
v2 = codebuddy:9ab42ef

Production
    ↓
Deploy v2
    ↓
Health Check FAIL
    ↓
Rollback
    ↓
Deploy v1
    ↓
Health Check PASS
~~~

The previous version should ideally be a known-good immutable artifact/image.

---

## 7. Automated Rollback Concept

Automated rollback means the deployment system can automatically return to a previous known-good version when defined verification checks fail.

Conceptually:

~~~text
Deploy v2
    ↓
Health Check
    ↓
   FAIL
    ↓
Rollback to v1
    ↓
Health Check
    ↓
   PASS
    ↓
Deployment Recovered
~~~

This can reduce recovery time.

However, automated rollback should be designed carefully.

Not every failure should automatically trigger rollback.

For example, a temporary network problem may require retrying the health check rather than immediately rolling back.

---

## 8. Retry vs Rollback

### Retry

Use when the failure may be temporary.

~~~text
Health Check
    ↓
Temporary timeout
    ↓
Retry
    ↓
PASS
~~~

### Rollback

Use when the deployed version is unhealthy or a release is clearly broken.

~~~text
Health Check
    ↓
Application Error
    ↓
Rollback
~~~

Mental model:

~~~text
Transient Failure
→ Retry

Release Failure
→ Rollback
~~~

A production pipeline can use a small number of retries before deciding that the deployment has failed.

---

## 9. Health Checks in Docker

Docker can also define a container health check.

Example:

~~~dockerfile
HEALTHCHECK --interval=30s --timeout=5s CMD curl -f http://localhost:3000/health || exit 1
~~~

Conceptually:

~~~text
Docker Container
      ↓
GET /health
      ↓
Healthy / Unhealthy
~~~

A container being running does not necessarily mean the application is healthy.

~~~text
Container Running
      ≠
Application Healthy
~~~

---

## 10. Health Checks in Kubernetes

Kubernetes commonly uses probes.

### Readiness Probe

Answers:

> Is this Pod ready to receive traffic?

~~~text
Readiness
   ↓
PASS → Receive Traffic
FAIL → Do not receive traffic
~~~

### Liveness Probe

Answers:

> Is this container still alive and should it be restarted?

~~~text
Liveness
   ↓
PASS → Keep Running
FAIL → Kubernetes may restart
~~~

Simple distinction:

~~~text
Liveness
→ Should the application be restarted?

Readiness
→ Should the application receive traffic?
~~~

These concepts become especially important during rolling deployments.

---

## 11. Complete Verification Flow

A robust deployment can use multiple verification layers:

~~~text
Deployment
    ↓
Process / Container Running
    ↓
Health Check
    ↓
Readiness
    ↓
Smoke Tests
    ↓
Critical Functionality
    ↓
PASS
~~~

If an important layer fails:

~~~text
Verification Failed
       ↓
Stop Promotion
       ↓
Retry / Investigate
       ↓
Rollback if required
~~~

---

## 12. Example Jenkins Verification

A simplified Jenkins stage:

~~~groovy
stage('Verify Deployment') {
    steps {
        sh 'curl -f https://example.com/health'
    }
}
~~~

For a small smoke test:

~~~groovy
stage('Smoke Test') {
    steps {
        sh './smoke-tests.sh'
    }
}
~~~

If either command returns a non-zero exit code, Jenkins normally marks the step/pipeline as failed.

---

## 13. Verification with Retry

A deployment may need a few seconds before the application is ready.

Instead of immediately failing:

~~~text
Deploy
 ↓
Health Check
 ↓
Not Ready
 ↓
Wait
 ↓
Retry
 ↓
Health Check
 ↓
PASS
~~~

Jenkins can use retry and timeout mechanisms.

Example:

~~~groovy
stage('Health Check') {
    steps {
        timeout(time: 2, unit: 'MINUTES') {
            retry(5) {
                sh 'curl -f https://example.com/health'
                sleep 5
            }
        }
    }
}
~~~

The exact retry count and delay should be chosen based on application startup behavior.

---

## 14. Production Verification Example

Suppose Jenkins deploys:

~~~text
codebuddy:9ab42ef
~~~

Then:

~~~text
Deploy
  ↓
Container Running?
  ↓
/health → 200?
  ↓
Critical API works?
  ↓
Smoke Test passes?
  ↓
Logs normal?
  ↓
PASS
~~~

If the health check fails repeatedly:

~~~text
Deploy
  ↓
/health → 500
  ↓
Retry
  ↓
/health → 500
  ↓
Retry
  ↓
Still FAIL
  ↓
Rollback
~~~

---

## 15. Deployment Verification and Rollback

The relationship is:

~~~text
Deployment
     ↓
Verification
     ↓
 ┌───┴───┐
 ↓       ↓
PASS    FAIL
 ↓       ↓
Success  Retry / Rollback
         ↓
      Previous
      Version
         ↓
      Verify Again
~~~

This creates a safer deployment process.

---

## 16. Important Production Considerations

### Keep health endpoints simple

A health endpoint should normally be fast and reliable.

### Do not expose sensitive information

Avoid returning:

- Database passwords
- API keys
- Internal secrets
- Sensitive infrastructure details

A response like this is enough:

~~~json
{
  "status": "ok"
}
~~~

### Use timeouts

Never allow deployment verification to wait forever.

### Use retries carefully

Retries should handle transient startup/network failures, not hide genuine application failures.

### Keep rollback artifacts available

If you want fast rollback, keep previous known-good image/artifact versions available.

---

## ⭐ Interview Points

### What is a health check?

> A health check is an automated check used to determine whether an application or service is alive and ready to operate.

### Why use a /health endpoint?

> It provides a simple, predictable endpoint that deployment and monitoring systems can use to verify application health.

### What is the difference between a health check and a smoke test?

> A health check verifies basic application availability, while smoke tests verify important application functionality.

### Why is HTTP 200 important?

> It indicates that the HTTP request completed successfully, although additional checks may still be required to determine full application health.

### What happens when a health check fails?

> The pipeline should stop promotion, retry when appropriate, investigate the failure, and roll back when the deployed version is unhealthy.

### What is automated rollback?

> Automated rollback returns the application to a previous known-good version when defined deployment verification checks fail.

### Retry vs rollback?

> Retry is appropriate for transient failures; rollback is appropriate when the deployed release is determined to be unhealthy.

### Is a running Docker container necessarily healthy?

> No. A container can be running while the application inside it is failing or not ready to accept requests.

### What is readiness?

> Readiness indicates whether an application is ready to receive traffic.

### What is liveness?

> Liveness indicates whether an application is alive and should continue running rather than being restarted.

---

## 🧠 Remember

The most important flow is:

~~~text
Deploy
  ↓
Container / Process Running?
  ↓
/health
  ↓
HTTP Status
  ↓
Smoke Tests
  ↓
Verification
  ↓
 ┌───────────────┐
 ↓               ↓
PASS            FAIL
 ↓               ↓
Success       Retry / Investigate
                 ↓
              Rollback
                 ↓
          Previous Version
                 ↓
            Verify Again
~~~

### Golden rule

**Deployment is not complete until the application has been verified.**

Remember:

> **Deploy → Health Check → Smoke Test → Verify → Decide**

And for failures:

> **Retry transient failures; rollback unhealthy releases.**
