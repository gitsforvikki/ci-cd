# Lesson 7 — Jenkins Agent Architecture

## What is a Jenkins Agent?

A **Jenkins Agent** is a machine that actually runs the commands of our pipeline.

For example: `npm ci`, `npm test`, `npm run build`, and Docker commands.

The Jenkins Controller manages the pipeline, while an Agent does the actual work.

```text
Jenkins Controller
       │
       │ assigns work
       ↓
Jenkins Agent
       │
       ├── npm ci
       ├── npm test
       └── npm run build
```

## Controller vs Agent

Think of Jenkins like a manager and worker:

```text
Jenkins Controller
       │
       │ manages and schedules
       ↓
Jenkins Agent
       │
       └── executes the work
```

### Jenkins Controller

The Controller is responsible for things like:
- Managing Jenkins
- Receiving build requests
- Reading pipeline configuration
- Deciding where a build should run
- Managing jobs and build information

### Jenkins Agent

The Agent is responsible for:
- Running pipeline commands
- Providing a workspace
- Building the application
- Running tests
- Running deployment commands when required

## Why do we need Agents?

If everything runs on one machine, it can become slow and overloaded.

With multiple agents, work can be distributed:

```text
              Jenkins Controller
                /      |      \
               /       |       \
          Agent 1   Agent 2   Agent 3
             ↓         ↓         ↓
           Build     Test      Deploy
```

## What is an Executor?

An **executor** is a slot on an Agent that can run one build or task at a time.

```text
Agent 1
├── Executor 1 → Build A
├── Executor 2 → Build B
└── Executor 3 → Build C
```

If an agent has 3 executors, it can normally run up to 3 tasks at the same time.

More executors do not automatically mean better performance. The machine must have enough CPU, memory, and other resources.

## What is an Agent Workspace?

A **workspace** is the directory on the Agent where Jenkins works with the project files.

```text
Jenkins Agent
      ↓
  Workspace
      ↓
Git checkout
      ↓
Project files
      ↓
Build / Test
```

Example:

```text
workspace/
├── src/
├── package.json
├── package-lock.json
└── Jenkinsfile
```

## How does a Pipeline use an Agent?

A simple Declarative Pipeline can use:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }
    }
}
```

`agent any` tells Jenkins that the pipeline can run on any available agent.

## Can different stages use different Agents?

Yes. This is useful when different stages need different environments.

```text
Build
  ↓
Node.js Agent

Test
  ↓
Testing Agent

Deploy
  ↓
Deployment Agent
```

## Complete Jenkins Architecture

```text
                  Jenkins Controller
                         │
              manages / schedules
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
      Agent 1         Agent 2         Agent 3
          │              │              │
      Workspace       Workspace       Workspace
          │              │              │
       Build           Test           Deploy
```

The Controller coordinates the work. The Agents perform the work.

## What happens when a build starts?

```text
1. Build is triggered
        ↓
2. Controller receives the build
        ↓
3. Jenkins finds an available Agent
        ↓
4. Agent gets a workspace
        ↓
5. Source code is checked out
        ↓
6. Jenkinsfile stages run
        ↓
7. Agent executes commands
        ↓
8. Build result is reported
```

## Why are Agents important in real projects?

Different projects may need different environments.

```text
Node.js Agent
    ↓
Next.js application

Docker Agent
    ↓
Docker image build

Linux Agent
    ↓
Deployment
```

Agents allow Jenkins to scale and run different workloads in suitable environments.

## ⭐ Interview Points

### What is a Jenkins Agent?
> A Jenkins Agent is a machine that executes pipeline tasks assigned by the Jenkins Controller.

### What is the difference between Controller and Agent?
> The Controller manages and schedules Jenkins work, while Agents execute the actual pipeline commands.

### What is an Executor?
> An Executor is a slot on an Agent that can execute one build or task at a time.

### What is a Workspace?
> A Workspace is the directory on a Jenkins Agent where the source code and build files are available during pipeline execution.

### Why use multiple Agents?
> Multiple Agents allow Jenkins to distribute workloads, run builds in different environments, and avoid putting all work on a single machine.

## 🧠 Remember

```text
Controller
    ↓
"Who should do the work?"
    ↓
Agent
    ↓
"Run the work"
    ↓
Executor
    ↓
"Run this task"
    ↓
Workspace
    ↓
"Files needed for the task"
```

**Controller = manages**

**Agent = executes**

**Executor = execution slot**

**Workspace = place where the project files are available**