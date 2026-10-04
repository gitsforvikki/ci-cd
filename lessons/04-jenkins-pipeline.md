# Lesson 4 — What Actually Happens When You Click "Build Now"?

## Simple idea

When you click **Build Now**, Jenkins starts one complete build process.

Conceptually:

~~~text
Build Now
   ↓
Jenkins creates a build
   ↓
Agent is selected
   ↓
Workspace is prepared
   ↓
Code is checked out
   ↓
Jenkinsfile is read
   ↓
Stages run
   ↓
Build / Test / Deploy
   ↓
Success or Failure
~~~

## 1. You click "Build Now"

Jenkins creates a new **build**.

For example:

~~~text
Build #1
Build #2
Build #3
Build #4
~~~

Each execution gets a build number.

The build number helps Jenkins keep track of each run.

## 2. Jenkins chooses an Agent

Jenkins does not always run the work directly on the Controller.

Usually, Jenkins gives the work to an **Agent**.

~~~text
Jenkins Controller
       ↓
     Agent
       ↓
   Build work
~~~

The agent is the machine where commands such as `npm ci`, tests, and builds actually run.

## 3. Jenkins creates a Workspace

The agent provides a **workspace**.

A workspace is simply a directory where Jenkins keeps the project while the build is running.

~~~text
Agent
  │
  └── Workspace
        ├── source code
        ├── node_modules
        ├── build files
        └── other temporary files
~~~

## 4. Jenkins gets the code

Jenkins checks out the required Git commit/branch into the workspace.

~~~text
GitHub
   ↓
git checkout
   ↓
Jenkins Workspace
~~~

Now Jenkins has the source code needed for the build.

## 5. Jenkins reads the Jenkinsfile

If the repository contains a Jenkinsfile, Jenkins uses it to know what to do.

For example:

~~~text
Jenkinsfile
    ↓
Install dependencies
    ↓
Run tests
    ↓
Build application
    ↓
Deploy
~~~

The Jenkinsfile is basically the **instructions for the pipeline**.

We will learn Jenkinsfile syntax properly in Lesson 5.

## 6. Jenkins runs Stages

A pipeline is normally divided into **stages**.

Example:

~~~text
Stage 1       Stage 2       Stage 3
Install   →   Test      →   Build
~~~

A stage groups related work.

Inside a stage are **steps**.

Example:

~~~text
Stage: Build
   ↓
Step: `npm run build`
~~~

### Stage vs Step vs Command

**Stage** = a logical part of the pipeline.

**Step** = an action Jenkins performs.

**Command** = the actual shell command that runs on the agent.

Example:

~~~text
Stage: Build
      ↓
Step: Run shell command
      ↓
Command: `npm run build`
~~~

## 7. Install dependencies

For a Node.js project, Jenkins may run:

~~~bash
`npm ci`
~~~

This installs the dependencies defined by the lock file.

## 8. Run tests

Jenkins can run tests before creating a production build.

Example:

~~~bash
`npm test`
~~~

If the tests fail, the pipeline can stop.

~~~text
Install
  ↓
Test ❌
  ↓
Pipeline stops
~~~

This prevents bad code from moving further through the pipeline.

## 9. Build the application

If tests pass, Jenkins can build the application.

For a Next.js application:

~~~bash
`npm run build`
~~~

The build creates the files needed to run the application.

## 10. Artifact or Docker image

After the build, Jenkins may produce something that can be deployed.

This could be:

- A build artifact
- A Docker image

Conceptually:

~~~text
Source Code
    ↓
Build
    ↓
Artifact / Docker Image
    ↓
Deployment
~~~

The exact output depends on the deployment architecture.

## 11. Credentials may be used

A deployment may need credentials, such as:

- Git credentials
- SSH credentials
- Docker registry credentials
- Cloud credentials

These should be stored securely in **Jenkins Credentials**, not hard-coded in the Jenkinsfile.

## 12. Deployment

If the pipeline contains a deployment stage, Jenkins can deploy the application.

For a Docker-based deployment:

~~~text
Jenkins
   ↓
Build Docker image
   ↓
Push image to registry
   ↓
Server pulls image
   ↓
Container starts
   ↓
Application is LIVE
~~~

## 13. Health check

After deployment, the pipeline can verify that the application is working.

For example:

~~~text
Deploy
  ↓
GET /health
  ↓
HTTP 200
  ↓
Deployment successful
~~~

If the health check fails, Jenkins can mark the build as failed and a rollback strategy can be used.

# What if something fails?

A pipeline can stop when an important step fails.

Example:

~~~text
Install
  ↓
Test
  ↓
Build ❌
  ↓
Pipeline FAILED
~~~

Jenkins records the failure so the developer can inspect the build logs.

# Build Time vs Runtime

This is very important.

### Build time

The application is being prepared.

~~~text
Install
  ↓
Test
  ↓
Build
  ↓
Artifact / Image
~~~

### Runtime

The application is actually running.

~~~text
Server
  ↓
Container / Process
  ↓
Next.js Application
  ↓
Users
~~~

So:

**Jenkins mainly handles the build/deployment process.**

The deployed application then runs on the server.

# Complete mental model

~~~text
             CLICK "BUILD NOW"
                     │
                     ↓
              Jenkins creates
                 Build #N
                     │
                     ↓
              Selects Agent
                     │
                     ↓
                 Workspace
                     │
                     ↓
               Checkout Code
                     │
                     ↓
                Jenkinsfile
                     │
                     ↓
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
    Install         Test          Build
       │             │             │
       └─────────────┴─────────────┘
                     ↓
              Artifact / Image
                     ↓
                 Deploy
                     ↓
               Health Check
                     ↓
             SUCCESS / FAILURE
~~~

# Important interview concepts

### What is Build Now?

It manually starts a Jenkins build.

### Where does the build actually run?

Usually on a Jenkins **Agent**.

### What is a workspace?

The directory on the agent where Jenkins checks out the source code and performs the build.

### What is a Jenkinsfile?

A file that defines the Jenkins pipeline instructions.

### What is a stage?

A logical section of the pipeline, such as Build, Test, or Deploy.

### What happens when a step fails?

The pipeline can stop or handle the error according to its configuration, and Jenkins records the build result.

# Remember

~~~text
Build Now
   ↓
Build
   ↓
Agent
   ↓
Workspace
   ↓
Checkout
   ↓
Jenkinsfile
   ↓
Stages
   ↓
Build / Test / Deploy
   ↓
Success / Failure
~~~

The main idea is:

> **Clicking "Build Now" starts a Jenkins build, which gets a workspace, checks out the code, executes the Jenkinsfile, runs its stages, and finally produces a success or failure result.**

## What you've learned so far

~~~text
Lesson 1
What are CI/CD and Deployment?
        ↓
Lesson 2
Code → LIVE
        ↓
Lesson 3
How Jenkins knows about changes
        ↓
Lesson 4
What Jenkins actually does during a build
~~~

Next, we learn **Lesson 5 — Jenkinsfile & Declarative Pipeline**.
