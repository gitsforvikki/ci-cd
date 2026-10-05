# Lesson 22 — Parallel & Distributed Builds

## What are Parallel and Distributed Builds?

Jenkins can run independent work at the same time instead of running everything one after another.

~~~text
Sequential:
Lint → Test → Security → Build

Parallel:
        ┌→ Lint ───────┐
Start ──┼→ Test ───────┼→ Build
        └→ Security ───┘
~~~

Parallel execution can reduce total pipeline time.

Distributed builds use different Jenkins agents or machines for different workloads.

~~~text
Jenkins Controller
        ↓
 ┌──────┼──────┐
 ↓      ↓      ↓
Agent1 Agent2 Agent3
Linux   Docker Windows
~~~

---

## 1. Parallel Stages

Independent stages can run at the same time.

~~~groovy
stage('Validation') {
    parallel {
        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Security Scan') {
            steps {
                sh './security-scan.sh'
            }
        }
    }
}
~~~

Instead of:

~~~text
Lint → Test → Security
~~~

we can run:

~~~text
Lint ─────┐
Test ─────┼→ Continue
Security ─┘
~~~

---

## 2. When to Use Parallel Execution

Parallel execution is useful when tasks:

- Do not depend on each other
- Can safely use available resources
- Take significant time
- Produce independent results

Example:

~~~text
Lint ───────┐
Unit Test ──┼→ Build
SAST ───────┘
~~~

---

## 3. When NOT to Use Parallel Execution

Dependent stages should remain sequential.

~~~text
Build
  ↓
Deploy
  ↓
Smoke Test
~~~

Smoke tests cannot run before deployment.

Ask:

> Does this stage need the output of another stage?

If yes, it probably needs to remain sequential.

---

## 4. Multiple Jenkins Agents

A Jenkins agent is a machine or execution environment where Jenkins runs pipeline work.

~~~text
Jenkins Controller
       │
       ├── Agent 1 → Linux / Node.js
       ├── Agent 2 → Linux / Docker
       └── Agent 3 → Windows
~~~

Different workloads can run on different agents.

---

## 5. Why Use Multiple Agents?

Multiple agents are useful when:

- Different operating systems are required
- Different tools are required
- Workloads are resource-heavy
- More builds need to run simultaneously
- One machine would otherwise become a bottleneck

Example:

~~~text
Agent 1 → Frontend tests
Agent 2 → Docker build
Agent 3 → Windows-specific tests
~~~

---

## 6. Executors

An executor is a Jenkins execution slot on an agent.

Think:

> One executor = one workload that can run at that time.

Example:

~~~text
Agent
 ├── Executor 1 → Build A
 ├── Executor 2 → Build B
 └── Executor 3 → Build C
~~~

If an agent has one executor:

~~~text
Build A → Running
Build B → Waiting
Build C → Waiting
~~~

Executors directly affect concurrency.

---

## 7. Build Queue

If Jenkins needs to run a build but no suitable executor is available, the build waits in the queue.

~~~text
Build Request
      ↓
Jenkins Queue
      ↓
Executor Available?
      ↓
     YES
      ↓
Run on Agent
~~~

A growing queue can indicate:

- Not enough agents
- Too few executors
- Long-running builds
- Resource bottlenecks
- Too many builds being triggered

---

## 8. Agent vs Executor

These concepts are commonly confused.

**Agent = where the work runs.**

**Executor = a slot on that agent where one workload can run.**

~~~text
Agent
   │
   ├── Executor 1
   ├── Executor 2
   └── Executor 3
~~~

---

## 9. Build Concurrency

Build concurrency means allowing multiple builds to execute at the same time.

Example:

~~~text
Commit A → Build #101
Commit B → Build #102
Commit C → Build #103
~~~

If enough resources are available, they may execute concurrently.

But concurrency can create problems.

---

## 10. Concurrency Problems

### Resource contention

Multiple builds may compete for:

- CPU
- RAM
- Disk
- Network
- Docker resources

~~~text
Build A ─┐
Build B ─┼→ Same Agent
Build C ─┘
          ↓
       CPU / RAM
          ↓
      Slow builds
~~~

### Deployment conflicts

Two production deployments can race with each other.

~~~text
Build A → Deploy v1
Build B → Deploy v2
             ↓
          Conflict
~~~

Production deployment concurrency should therefore be controlled.

---

## 11. Resource Management

Parallel execution does not mean:

> Run everything at the same time.

We must consider available resources.

~~~text
Agent
 ├── CPU: 4 cores
 ├── RAM: 8 GB
 └── Disk: 50 GB
~~~

Five heavy Docker builds may overload the machine.

Good CI/CD design balances:

~~~text
Pipeline Speed
      ↕
Available Resources
~~~

---

## 12. Labels and Agent Selection

Jenkins agents can have labels.

Example:

~~~text
Agent A → linux docker
Agent B → linux node
Agent C → windows
~~~

A stage can request an appropriate label.

~~~groovy
stage('Docker Build') {
    agent {
        label 'docker'
    }
    steps {
        sh 'docker build -t myapp:$BUILD_NUMBER .'
    }
}
~~~

The stage then runs on a matching agent.

---

## 13. Multiple Agents in One Pipeline

Different stages can use different agents.

~~~text
Pipeline
   │
   ├── Lint → Node Agent
   ├── Docker Build → Docker Agent
   └── Windows Test → Windows Agent
~~~

Example:

~~~groovy
pipeline {
    agent none

    stages {
        stage('Lint') {
            agent { label 'node' }
            steps {
                sh 'npm run lint'
            }
        }

        stage('Docker Build') {
            agent { label 'docker' }
            steps {
                sh 'docker build -t myapp:$BUILD_NUMBER .'
            }
        }
    }
}
~~~

`agent none` means the pipeline does not reserve one global agent. Individual stages can choose their own agents.

---

## 14. Parallel + Multiple Agents

Parallel stages can also use different agents.

~~~text
                 Jenkins
                    ↓
              Validation
                    ↓
       ┌────────────┼────────────┐
       ↓            ↓            ↓
   Node Agent   Docker Agent  Security Agent
       ↓            ↓            ↓
      Test         Build         Scan
       └────────────┼────────────┘
                    ↓
                 Continue
~~~

This can be a powerful production pattern.

---

## 15. Build Queue and Resource Planning

Suppose:

~~~text
10 builds requested
2 executors available
~~~

Then approximately:

~~~text
2 → Running
8 → Queue
~~~

As executors become available:

~~~text
Queue
 ↓
Build
 ↓
Executor
 ↓
Complete
 ↓
Next queued build
~~~

---

## 16. Controlling Build Concurrency

Not every pipeline should allow unlimited concurrent builds.

For example, two production deployments could conflict.

Jenkins Declarative Pipeline can use:

~~~groovy
options {
    disableConcurrentBuilds()
}
~~~

Conceptually:

~~~text
Build #101 → Running
Build #102 → Wait
~~~

This can be useful for deployment pipelines where ordering matters.

Do not disable concurrency everywhere. Independent CI validation can often run concurrently.

---

## 17. Workspace Considerations

Each build needs an isolated workspace.

Jenkins normally manages separate workspaces for concurrent executions.

Problems can still happen when builds use:

- Shared directories
- Global files
- Shared Docker resources
- External mutable state
- Custom scripts writing outside the workspace

Avoid unsafe shared state.

---

## 18. Parallel Stage Failure

Suppose three validation stages run in parallel:

~~~text
Lint      → PASS
Test      → FAIL
Security  → PASS
~~~

The validation group should normally be considered failed.

~~~text
Lint ───── PASS ──┐
Test ───── FAIL ──┼→ Validation FAIL
Security ─ PASS ──┘
~~~

Required validation must pass before deployment continues.

---

## 19. Practical CI Pipeline

A useful pattern is:

~~~text
                 Checkout
                    ↓
                 Install
                    ↓
             ┌──────┼──────┐
             ↓      ↓      ↓
           Lint    Test   SAST
             ↓      ↓      ↓
             └──────┼──────┘
                    ↓
                  Build
                    ↓
                 Deploy
~~~

Lint, tests, and security scanning can run in parallel when independent.

---

## 20. Distributed Build Architecture

A larger Jenkins setup may look like:

~~~text
                    Jenkins Controller
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
     Linux Agent      Docker Agent      Windows Agent
          │                │                │
       Tests             Build          Windows Tests
          └────────────────┼────────────────┘
                           ↓
                       Artifact
                           ↓
                        Registry
                           ↓
                       Deployment
~~~

The controller coordinates the work while agents execute it.

---

## 21. Resource Management Best Practices

### Match workloads to agents

Use labels to select suitable machines.

### Avoid excessive concurrency

More executors do not automatically mean faster builds.

### Monitor CPU and memory

Watch for overloaded agents.

### Clean workspaces

Old files can consume disk space.

### Use caching carefully

Caching can speed builds, but poorly managed shared caches can introduce problems.

### Separate production resources

Production deployment should not unnecessarily compete with heavy development builds.

---

## ⭐ Interview Points

### What is a parallel stage?

> A parallel stage allows independent pipeline tasks to execute at the same time, reducing total pipeline duration.

### What is a Jenkins agent?

> An agent is a machine or execution environment where Jenkins runs pipeline steps.

### What is an executor?

> An executor is an execution slot on an agent that allows one workload to run at a time.

### Agent vs executor?

> An agent is the machine; an executor is a slot on that machine.

### What is the Jenkins build queue?

> It is the waiting area for builds that cannot start because a suitable executor or resource is not currently available.

### Why use multiple agents?

> To distribute workloads, support different environments, increase concurrency, and prevent one machine from becoming a bottleneck.

### Why not run every stage in parallel?

> Some stages depend on previous stages, and excessive parallelism can cause resource contention or deployment conflicts.

### How do you prevent two deployments from running together?

> Use concurrency controls such as disabling concurrent builds or appropriate deployment locking mechanisms.

### What happens when all executors are busy?

> New builds wait in the Jenkins queue until a suitable executor becomes available.

### Does adding executors always make builds faster?

> No. CPU, memory, disk, network, and workload characteristics can become bottlenecks.

---

## 🧠 Remember

~~~text
                  Jenkins Controller
                         ↓
                    Build Queue
                         ↓
               Available Executors
                         ↓
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
     Agent 1          Agent 2          Agent 3
        ↓                ↓                ↓
      Lint             Test             Scan
        └────────────────┼────────────────┘
                         ↓
                       Build
                         ↓
                      Deploy
~~~

### Golden rules

**1. Parallelize independent work.**

**2. Keep dependent stages sequential.**

**3. Agent = where the work runs.**

**4. Executor = how many workloads can run on an agent simultaneously.**

**5. Queue = work waiting for resources.**

**6. More concurrency is not always better.**

**7. Control production deployment concurrency.**

**8. Match workloads to suitable agents and monitor resources.**

### Final interview-ready summary

> Parallel builds reduce pipeline time by running independent work simultaneously. Distributed builds use multiple Jenkins agents to execute workloads across different machines or environments. Executors determine how many workloads an agent can run concurrently, while the build queue holds work waiting for available resources. Good CI/CD design balances speed, correctness, and resource usage.
