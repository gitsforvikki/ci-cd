# Lesson 9 — Jenkinsfile Deep Dive

## What we already know

A Jenkinsfile defines our Jenkins pipeline as code.

In Lesson 5 we learned the basic structure:

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

Now we will understand the important Jenkinsfile blocks one by one.

---

## 1. `pipeline`

`pipeline` is the main block of a Declarative Pipeline.

```groovy
pipeline {
    ...
}
```

Everything that defines our Declarative Pipeline goes inside it.

Think:

**pipeline = the complete CI/CD plan**

---

## 2. `agent`

`agent` tells Jenkins where the pipeline should run.

```groovy
agent any
```

This means Jenkins can use any available agent.

You can also define an agent for a specific environment when needed.

Simple idea:

```text
Jenkins Controller
       ↓
   chooses Agent
       ↓
Pipeline runs
```

---

## 3. `stages`

`stages` contains the major sections of the pipeline.

```groovy
stages {
    stage('Build') { ... }
    stage('Test') { ... }
    stage('Deploy') { ... }
}
```

Think:

**stages = pipeline's major steps**

---

## 4. `stage`

A `stage` represents one logical part of the pipeline.

```groovy
stage('Test') {
    steps {
        sh 'npm test'
    }
}
```

Common stages:

```text
Install → Test → Build → Deploy
```

Stages also make the Jenkins UI easier to understand.

---

## 5. `steps`

`steps` contains the commands Jenkins executes.

```groovy
steps {
    sh 'npm ci'
    sh 'npm test'
}
```

On a Linux agent, `sh` can execute a shell command.

Example:

```groovy
steps {
    sh 'npm run build'
}
```

Remember:

**Stage = logical section**

**Step = actual work/command**

---

## 6. `environment`

`environment` is used to define environment variables for the pipeline or a particular stage.

Pipeline-level example:

```groovy
pipeline {
    agent any

    environment {
        APP_ENV = 'production'
    }

    stages {
        stage('Build') {
            steps {
                sh 'echo $APP_ENV'
            }
        }
    }
}
```

A variable can also be limited to a stage.

```groovy
stage('Build') {
    environment {
        APP_ENV = 'production'
    }
    steps {
        sh 'npm run build'
    }
}
```

Important:

Do not put real passwords or secrets directly in the Jenkinsfile.

---

## 7. `parameters`

`parameters` allows a user to provide values when starting a build.

Example:

```groovy
parameters {
    choice(
        name: 'ENVIRONMENT',
        choices: ['staging', 'production'],
        description: 'Where should we deploy?'
    )
}
```

Then the pipeline can use the selected value.

```text
Build Now
   ↓
Choose environment
   ↓
staging / production
   ↓
Pipeline runs
```

Parameters are useful when we want the same pipeline to behave differently based on user input.

---

## 8. `when`

`when` controls whether a stage should run.

For example, deploy only from the `main` branch:

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

This is useful for branch-based CI/CD rules.

---

## 9. `post`

`post` defines actions that happen after a pipeline or stage finishes.

Example:

```groovy
post {
    success {
        echo 'Build successful'
    }
    failure {
        echo 'Build failed'
    }
}
```

Common conditions include:

- `success`
- `failure`
- `always`
- `unstable`

For example, notifications or cleanup can be placed in `post`.

---

## 10. `input`

`input` pauses the pipeline and asks for human approval.

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
Production deployment
```

This is useful for production approval gates.

---

## 11. `options`

`options` lets us configure pipeline behavior.

For example, a timeout:

```groovy
options {
    timeout(time: 10, unit: 'MINUTES')
}
```

If the pipeline takes longer than the configured limit, Jenkins can stop it.

Another useful option is keeping a limited number of builds:

```groovy
options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
}
```

Simple idea:

**options = pipeline behavior/configuration**

---

## 12. `tools`

`tools` can tell Jenkins which configured build tools should be available to the pipeline.

For example, an organization may configure a Node.js installation in Jenkins and reference it from the pipeline.

The exact tool names depend on the Jenkins setup and installed/configured tools.

Simple idea:

```text
Jenkins tool configuration
          ↓
       tools block
          ↓
Pipeline uses tool
```

---

## 13. `triggers`

`triggers` can define how Jenkins should automatically start a pipeline.

For example, Jenkins can be configured to poll a Git repository periodically.

```groovy
triggers {
    pollSCM('H/5 * * * *')
}
```

However, in modern GitHub CI/CD setups, webhooks are commonly preferred because Jenkins can be notified immediately when an event happens.

Simple comparison:

```text
Polling
Jenkins → Is there a change?
Jenkins → Is there a change?
Jenkins → Is there a change?

Webhook
GitHub → Something changed!
```

---

## Putting the important blocks together

```groovy
pipeline {
    agent any

    environment {
        APP_ENV = 'staging'
    }

    parameters {
        choice(name: 'DEPLOY', choices: ['yes', 'no'])
    }

    options {
        timeout(time: 10, unit: 'MINUTES')
    }

    stages {
        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
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

    post {
        success {
            echo 'Pipeline successful'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
```

Do not try to memorize the entire example.

Understand what each block controls.

---

## 🧠 Easy Mental Model

```text
pipeline
   │
   ├── agent       → Where?
   ├── environment → Variables
   ├── parameters  → User input
   ├── options     → Pipeline behavior
   ├── tools       → Build tools
   ├── triggers    → How it starts
   │
   ├── stages
   │    └── stage
   │         └── steps → What to execute
   │
   ├── when        → Should this stage run?
   ├── input       → Human approval
   └── post        → What happens after?
```

---

## ⭐ Interview Points

### What is `environment`?
> It defines environment variables that can be used by the pipeline or a stage.

### What is `parameters`?
> It allows users to provide input when starting a build.

### What is `when`?
> It defines conditions that determine whether a stage should execute.

### What is `post`?
> It defines actions that run after a pipeline or stage completes, such as success, failure, cleanup, or notifications.

### What is `input`?
> It pauses the pipeline and waits for human input or approval.

### What is `options`?
> It configures pipeline behavior, such as timeouts and build retention.

### What is `triggers`?
> It defines automatic conditions that can start a pipeline, such as SCM polling. Webhooks are commonly used with GitHub integrations.

---

## Final takeaway

A Jenkinsfile is not just a list of commands.

It describes **where the pipeline runs, when it runs, what it does, under which conditions it runs, and what happens after it finishes.**