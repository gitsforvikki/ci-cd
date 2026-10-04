# Lesson 5 — Jenkinsfile & Declarative Pipeline

## What is a Jenkinsfile?

A **Jenkinsfile** is a text file that tells Jenkins **what to do** during a CI/CD pipeline.

Instead of configuring every step manually in Jenkins, we write the pipeline as code and keep the Jenkinsfile in our Git repository.

```text
Git Repository
      ↓
Jenkinsfile
      ↓
Jenkins reads it
      ↓
Pipeline runs
```

## Why do we use a Jenkinsfile?

- Pipeline configuration is stored with the code.
- Changes can be tracked with Git.
- The pipeline can be reviewed through pull requests.
- The same pipeline can be recreated easily.

This is called **Pipeline as Code**.

## Declarative Pipeline

For beginners, the important Jenkins pipeline style is **Declarative Pipeline**.

It uses a clear and structured format:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building application...'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
            }
        }
    }
}
```

Think of it like:

```text
pipeline
   │
   ├── agent
   │
   └── stages
        │
        ├── Build
        │    └── steps
        ├── Test
        │    └── steps
        └── Deploy
             └── steps
```

## Important Jenkinsfile Parts

### 1. pipeline

`pipeline { ... }` is the main block of a Declarative Pipeline.

### 2. agent

`agent any` tells Jenkins to run the pipeline on any available agent.

```text
Jenkins Controller
       ↓
   assigns Agent
       ↓
Agent runs commands
```

### 3. stages

`stages` contains the major parts of the pipeline, such as Build, Test, and Deploy.

### 4. stage

A `stage` represents one logical part of the pipeline.

Examples: Build, Test, Deploy.

### 5. steps

`steps` contains the actual commands Jenkins executes.

For example:

```groovy
steps {
    sh 'npm ci'
    sh 'npm run build'
}
```

Remember:

**Stage = what part of the pipeline?**

**Steps = what commands should run?**

## Simple Real Example

For a Node.js project:

```groovy
pipeline {
    agent any

    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}
```

Flow:

```text
Jenkins
   ↓
Install dependencies
   ↓
Run tests
   ↓
Build application
   ↓
Success / Failure
```

If the test stage fails, Jenkins normally stops the later stages:

```text
Install ✅
   ↓
Test ❌
   ↓
Build ⛔
```

## Jenkinsfile vs Jenkins

**Jenkins** is the automation server.

**Jenkinsfile** is the file that defines the pipeline instructions.

```text
Jenkins
  │
  └── reads Jenkinsfile
           ↓
       executes pipeline
```

## Why keep Jenkinsfile in Git?

A common project structure is:

```text
my-project/
├── app/
├── package.json
├── src/
└── Jenkinsfile
```

Then:

```text
Developer
   ↓
git push
   ↓
GitHub
   ↓
Jenkins
   ↓
reads Jenkinsfile
   ↓
runs pipeline
```

This keeps application code and its CI/CD instructions together.

## ⭐ Interview Points

**What is a Jenkinsfile?**

> A Jenkinsfile is a file stored in a source-control repository that defines the Jenkins CI/CD pipeline as code.

**What is a Declarative Pipeline?**

> Declarative Pipeline is a structured way of defining Jenkins pipelines using blocks such as `pipeline`, `agent`, `stages`, `stage`, and `steps`.

**What is Pipeline as Code?**

> Pipeline as Code means storing CI/CD pipeline configuration in a version-controlled file such as a Jenkinsfile.

**Difference between stage and step?**

> A **stage** represents a logical section of the pipeline, while **steps** are the actual commands executed inside that stage.

## 🧠 Remember

```text
Jenkinsfile
     ↓
pipeline
     ↓
agent
     ↓
stages
     ↓
stage
     ↓
steps
     ↓
commands
```

**Jenkinsfile = instructions for Jenkins about how to run our CI/CD pipeline.**