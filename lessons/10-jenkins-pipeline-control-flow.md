# Lesson 10 — Jenkins Pipeline Control Flow

## What is Pipeline Control Flow?

Pipeline control flow means controlling:

- Which stage runs first
- Which stage runs next
- Whether a stage should run
- Whether stages can run in parallel
- What happens when something fails
- When a human must approve
- How long a stage is allowed to run
- Whether Jenkins should retry a failed operation

Simple idea:

```text
Pipeline
   ↓
Decide what should run
   ↓
Run stage
   ↓
Check result
   ↓
Continue / Skip / Retry / Stop
```

---

## 1. Stage Dependencies

Normally, Declarative Pipeline stages run in the order they are written.

```text
Build
  ↓
Test
  ↓
Deploy
```

Example:

```groovy
stages {
    stage('Build') {
        steps {
            sh 'npm run build'
        }
    }

    stage('Test') {
        steps {
            sh 'npm test'
        }
    }

    stage('Deploy') {
        steps {
            sh './deploy.sh'
        }
    }
}
```

If Build fails, Test and Deploy normally do not continue.

```text
Build ❌
  ↓
Test ⛔
  ↓
Deploy ⛔
```

---

## 2. Sequential Stages

Sequential means one stage finishes before the next stage starts.

```text
Build
  ↓
Test
  ↓
Deploy
```

This is the normal flow of many pipelines.

For example:

```text
npm ci
  ↓
npm test
  ↓
npm run build
  ↓
deploy
```

Why is this useful?

We usually don't want to deploy an application before its tests and build have succeeded.

---

## 3. Parallel Stages

Sometimes two tasks do not depend on each other.

Instead of running them one after another, we can run them in parallel.

```text
             ┌── Unit Tests
Build ───────┤
             └── Lint
                    ↓
                  Deploy
```

Example:

```groovy
stage('Quality Checks') {
    parallel {
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }
    }
}
```

Instead of:

```text
Test → Lint → Deploy
```

we can do:

```text
       ┌→ Test ──┐
Build ─┤         ├→ Deploy
       └→ Lint ──┘
```

Parallel execution can reduce pipeline time.

---

## 4. Conditional Execution

Sometimes a stage should run only when a condition is true.

Jenkins provides the `when` block for this.

Example:

```groovy
stage('Deploy') {
    when {
        branch 'main'
    }
    steps {
        sh './deploy.sh'
    }
}
```

Flow:

```text
Branch = main?
     ↓
   Yes → Deploy
   No  → Skip
```

This is useful for rules such as:

- Deploy only from `main`
- Run production deployment only for a specific branch
- Run a stage only for a particular environment

---

## 5. Manual Approvals

Sometimes production deployment should not happen automatically.

We can stop the pipeline and ask a human for approval.

Example:

```groovy
stage('Production Approval') {
    steps {
        input message: 'Deploy to production?'
    }
}
```

Flow:

```text
Build
  ↓
Test
  ↓
Approval required
  ↓
Human approves
  ↓
Production Deploy
```

If the person does not approve, production deployment does not continue.

This is commonly called an **approval gate**.

---

## 6. Timeouts

Sometimes a build or deployment gets stuck.

A timeout prevents Jenkins from waiting forever.

Example:

```groovy
options {
    timeout(time: 10, unit: 'MINUTES')
}
```

If the pipeline takes longer than the configured limit, Jenkins stops it.

Simple flow:

```text
Pipeline starts
      ↓
Timer starts
      ↓
Work continues
      ↓
10 minutes reached
      ↓
Pipeline stopped
```

Timeouts are useful for preventing stuck builds and deployments.

---

## 7. Retries

Some failures can be temporary.

For example:

- Temporary network problem
- Registry connection problem
- Temporary external service failure

In these cases, we may want Jenkins to try again.

Example:

```groovy
retry(3) {
    sh 'npm test'
}
```

This means Jenkins can try the block again when it fails, up to the configured number of attempts.

Flow:

```text
Attempt 1 ❌
    ↓
Attempt 2 ❌
    ↓
Attempt 3 ✅
    ↓
Continue
```

Do not blindly retry every failure. A real code error will normally fail again.

---

## 8. Error Handling

A pipeline can fail for many reasons.

```text
Test failure
Build failure
Network failure
Deployment failure
Configuration error
```

Jenkins provides ways to control what happens after an error.

Two important approaches are:

- `catchError`
- `try/catch`

---

## 9. `catchError`

`catchError` lets us catch a failure and control the build/stage result instead of immediately stopping all pipeline logic.

Example:

```groovy
steps {
    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
        sh 'some-command'
    }
}
```

The exact result behavior depends on the values we configure.

The important idea is:

**catchError lets the pipeline handle an error instead of simply failing at that point.**

This can be useful when a failure should be recorded but should not necessarily stop every later action.

---

## 10. `try/catch`

Jenkins Pipeline uses Groovy, so we can also use `try/catch` for more direct error handling.

Example:

```groovy
steps {
    script {
        try {
            sh 'npm test'
        } catch (err) {
            echo 'Tests failed'
        }
    }
}
```

Flow:

```text
Run command
    ↓
Success? ── Yes → Continue
    │
    No
    ↓
catch error
    ↓
Handle failure
```

Use `try/catch` when you need custom logic around an operation.

---

## Putting Everything Together

A production pipeline can combine these controls:

```text
Build
  ↓
Test ───────────────┐
  ↓                 │
Lint ───────────────┤ parallel
  ↓                 │
Quality checks ─────┘
  ↓
Deploy to Staging
  ↓
Health Check
  ↓
Manual Approval
  ↓
Deploy to Production
  ↓
Verification
```

With rules such as:

```text
Only main branch → Production
Failed test → Stop
Temporary failure → Retry
Long-running job → Timeout
Production → Manual approval
Special failure → Error handling
```

---

## Example Pipeline

```groovy
pipeline {
    agent any

    options {
        timeout(time: 20, unit: 'MINUTES')
    }

    stages {
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Checks') {
            parallel {
                stage('Test') {
                    steps {
                        retry(2) {
                            sh 'npm test'
                        }
                    }
                }

                stage('Lint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?'
                sh './deploy.sh'
            }
        }
    }
}
```

Do not memorize this whole pipeline.

Understand the control flow:

```text
Build
  ↓
Parallel checks
  ↓
Branch condition
  ↓
Approval
  ↓
Deploy
```

---

## ⭐ Interview Points

### What is sequential execution?
> Running pipeline stages one after another, where the next stage starts after the previous stage finishes.

### Why use parallel stages?
> To run independent tasks at the same time and reduce total pipeline execution time.

### What is conditional execution?
> Running a stage only when a defined condition is satisfied, commonly using `when`.

### Why use a manual approval?
> To add a human approval gate before sensitive actions such as production deployment.

### Why use a timeout?
> To stop a pipeline that takes too long or becomes stuck.

### Why use retry?
> To automatically retry operations that may fail temporarily.

### `catchError` vs `try/catch`?
> Both can handle failures, but `catchError` is a Jenkins Pipeline step for controlling build/stage results, while `try/catch` gives Groovy-style control for custom error-handling logic.

---

## 🧠 Remember

```text
Sequential  → One after another
Parallel    → At the same time
when        → Should this stage run?
input       → Ask a human
timeout     → Don't wait forever
retry       → Try again
catchError  → Control Jenkins failure result
try/catch   → Custom error handling
```

**Pipeline control flow = controlling how Jenkins moves through the pipeline and handles different situations.**