# Lesson 16 — Docker + Jenkins

## What is Docker + Jenkins?

Jenkins automates the CI/CD process.

Docker packages the application and its runtime into a Docker image.

Together:

~~~text
GitHub
   ↓
Jenkins
   ↓
Docker Build
   ↓
Docker Image
   ↓
Registry
   ↓
Production Server
   ↓
Container
~~~

The important idea is:

**Jenkins automates the process; Docker packages and runs the application.**

---

## 1. Docker Build Inside Jenkins

After Jenkins checks out the source code, it can build a Docker image.

Example:

~~~bash
docker build -t codebuddy:9ab42ef .
~~~

Here:

- **docker build** → builds the image
- **-t** → gives the image a name and tag
- **codebuddy** → image name
- **9ab42ef** → image tag
- **.** → current directory is the Docker build context

Typical Jenkins flow:

~~~text
Checkout
   ↓
Install / Test
   ↓
Docker Build
   ↓
Docker Image
~~~

Jenkins is executing the Docker command on the Jenkins agent.

---

## 2. Jenkins Agent Needs Docker

The Docker command must run somewhere.

Usually:

~~~text
Jenkins Controller
       ↓
Jenkins Agent
       ↓
Docker
       ↓
Docker Image
~~~

The Jenkins agent must have access to the required Docker tooling and daemon.

If Docker is not available, Jenkins may fail with an error such as:

~~~text
Cannot connect to the Docker daemon
~~~

So:

**Jenkins does not magically provide Docker. The execution environment must have the required Docker capability.**

---

## 3. Docker Image Tagging

A Docker image has a name and tag.

Example:

~~~text
codebuddy:9ab42ef
          ↑
         tag
~~~

A useful production tag is the Git commit SHA.

Example:

~~~text
Git commit
9ab42ef
   ↓
Docker image
codebuddy:9ab42ef
~~~

This gives us traceability.

We can understand:

~~~text
Git SHA
   ↓
Jenkins Build
   ↓
Docker Image
   ↓
Production
~~~

### Why not use only `latest`?

Example:

~~~text
codebuddy:latest
~~~

The problem is that `latest` does not clearly tell us which source version is running.

Prefer immutable or unique tags such as:

~~~text
codebuddy:9ab42ef
codebuddy:build-104
codebuddy:v1.2.0
~~~

For CI/CD, Git SHA tags are especially useful because they directly identify the source commit.

---

## 4. Docker Registry

A Docker registry stores and distributes Docker images.

Simple flow:

~~~text
Jenkins
   ↓
Docker Build
   ↓
Docker Image
   ↓
Docker Registry
   ↓
Production Server
~~~

Examples of registries include:

- Docker Hub
- GitHub Container Registry (GHCR)
- Amazon ECR
- Google Artifact Registry
- Azure Container Registry

The registry acts as the central place from which deployment servers can pull the required image.

---

## 5. Docker Login

If the registry is private, Jenkins needs authentication before pushing the image.

Conceptually:

~~~text
Jenkins
   ↓
Registry Credentials
   ↓
Docker Login
   ↓
Authenticated
~~~

Example:

~~~bash
docker login
~~~

In a real Jenkins pipeline, credentials should come from the Jenkins Credentials Store.

Do **not** write passwords directly inside the Jenkinsfile.

Bad:

~~~groovy
sh 'docker login -u myuser -p mypassword'
~~~

Better concept:

~~~text
Jenkins Credentials Store
          ↓
     credentialsId
          ↓
      Docker Login
~~~

The exact implementation depends on the registry and Jenkins setup.

---

## 6. Push the Docker Image

After building and logging in, Jenkins pushes the image to the registry.

Example:

~~~bash
docker push codebuddy:9ab42ef
~~~

The complete flow:

~~~text
Source Code
    ↓
Jenkins
    ↓
Docker Build
    ↓
codebuddy:9ab42ef
    ↓
Docker Login
    ↓
Docker Push
    ↓
Docker Registry
~~~

Now the image is available to deployment infrastructure.

---

## 7. Pull the Image on the Server

The production server does not need to build the application again.

It can pull the exact image from the registry.

Example:

~~~bash
docker pull codebuddy:9ab42ef
~~~

Flow:

~~~text
Docker Registry
      ↓
docker pull
      ↓
Production Server
      ↓
Docker Image
~~~

This is one of the most important CI/CD ideas:

**Build the image once, then deploy that same image.**

---

## 8. Container Deployment

An image is not the same as a running container.

Think:

~~~text
Docker Image
     ↓
 docker run
     ↓
Container
~~~

For example:

~~~bash
docker run -d \
  --name codebuddy \
  -p 3000:3000 \
  codebuddy:9ab42ef
~~~

Here:

- **-d** → run in detached mode
- **--name codebuddy** → container name
- **-p 3000:3000** → map host port to container port
- **codebuddy:9ab42ef** → exact image to run

The important distinction:

**Image = packaged application**

**Container = running instance of an image**

---

## 9. Complete Jenkins + Docker Flow

A simple production pipeline looks like this:

~~~text
Developer
    ↓
GitHub
    ↓
Webhook
    ↓
Jenkins
    ↓
Checkout
    ↓
Test
    ↓
Build
    ↓
Docker Build
    ↓
Image: codebuddy:GIT_SHA
    ↓
Docker Login
    ↓
Docker Push
    ↓
Docker Registry
    ↓
Production Server
    ↓
Docker Pull
    ↓
Docker Run
    ↓
Container
~~~

This is the complete mental model for this lesson.

---

## 10. Example Jenkins Pipeline

A simplified example:

~~~groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'codebuddy'
        IMAGE_TAG = '9ab42ef'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh 'npm ci'
                sh 'npm test'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Docker Login') {
            steps {
                // Use Jenkins-managed credentials here.
                sh 'docker login'
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker pull $IMAGE_NAME:$IMAGE_TAG'
                sh 'docker stop codebuddy || true'
                sh 'docker rm codebuddy || true'
                sh 'docker run -d --name codebuddy -p 3000:3000 $IMAGE_NAME:$IMAGE_TAG'
            }
        }
    }
}
~~~

This is intentionally simplified.

In a production pipeline, credentials, registry URLs, environment configuration, health checks, rollback, and deployment safety need to be handled properly.

---

## 11. Build Once → Deploy the Same Image

This is one of the most important CI/CD principles.

### Bad approach

~~~text
Source
  ↓
Build for Staging
  ↓
Staging

Source
  ↓
Build again for Production
  ↓
Production
~~~

The two builds could differ.

### Better approach

~~~text
Source
  ↓
Build
  ↓
Docker Image
codebuddy:9ab42ef
  ↓
Registry
  ↓
Staging
  ↓
Testing
  ↓
Production
~~~

The same image is promoted.

Therefore:

**Build once → Test → Deploy the same image.**

---

## 12. Why Docker Helps CI/CD

Without Docker, the application may depend on the server's installed environment.

For example:

~~~text
Server
 ├── Node 18
 ├── npm version X
 ├── OS dependencies
 └── Other software
~~~

Another server may have:

~~~text
Server
 ├── Node 22
 ├── npm version Y
 ├── Different OS packages
 └── Different configuration
~~~

This can create:

**"Works on my machine"**

Docker packages the application with its required runtime environment.

~~~text
Application
    +
Runtime
    +
Dependencies
    ↓
Docker Image
    ↓
Container
~~~

This reduces environment mismatch.

---

## 13. Registry vs Server

These are different things.

### Registry

Stores images.

~~~text
Docker Registry
    ↓
codebuddy:9ab42ef
~~~

### Server

Runs containers.

~~~text
Production Server
    ↓
Container
    ↓
Application
~~~

So:

**Registry stores the image; the server pulls and runs the image.**

---

## 14. Docker + Jenkins Responsibilities

Keep these responsibilities clear.

### Jenkins

- Checkout source
- Run tests
- Build application
- Build Docker image
- Authenticate with registry
- Push image
- Trigger deployment
- Verify deployment

### Docker

- Build image
- Package application
- Run container
- Provide isolated runtime environment

### Registry

- Store images
- Version images
- Distribute images

### Production Server

- Pull image
- Run container
- Provide application runtime

Simple mental model:

~~~text
Jenkins
   ↓
"Automate the process"

Docker
   ↓
"Package and run the application"

Registry
   ↓
"Store and distribute the image"

Server
   ↓
"Run the container"
~~~

---

## 15. Important Production Considerations

A basic deployment might use:

~~~bash
docker stop codebuddy
docker rm codebuddy
docker run ...
~~~

This can cause downtime.

For production systems, more advanced deployment strategies can be used:

- Rolling deployment
- Blue-green deployment
- Canary deployment
- Load balancer based switching

These will be covered in deployment strategy lessons.

Also remember:

**Do not bake production secrets into the Docker image.**

Use runtime configuration and proper secret management.

---

## 16. Complete Docker + Jenkins Architecture

~~~text
                         Developer
                             ↓
                          GitHub
                             ↓
                          Webhook
                             ↓
                         Jenkins
                             ↓
                   ┌─────────┴─────────┐
                   ↓                   ↓
                Checkout             Tests
                   │
                   └─────────┬─────────┘
                             ↓
                       Docker Build
                             ↓
                  codebuddy:9ab42ef
                             ↓
                       Docker Login
                             ↓
                       Docker Push
                             ↓
                    Docker Registry
                             ↓
                       Docker Pull
                             ↓
                     Production Server
                             ↓
                       Docker Run
                             ↓
                         Container
                             ↓
                       Node.js App
~~~

---

## ⭐ Interview Points

### What is the role of Docker in Jenkins?

> Jenkins automates the CI/CD process, while Docker packages the application into an image and provides a consistent runtime environment.

### Where does Docker build happen?

> The Docker build runs on the Jenkins agent that has the required Docker capabilities.

### Why tag Docker images?

> Tags identify different image versions. Using a Git commit SHA makes the deployed image traceable to a specific source commit.

### Why use a Docker registry?

> A registry provides centralized storage and distribution of Docker images so deployment servers can pull the required version.

### Why should Jenkins not hard-code Docker credentials?

> Credentials should be stored securely in Jenkins Credentials Store or another secret manager, not committed to source code.

### What is the difference between an image and a container?

> An image is a packaged, immutable template; a container is a running instance of that image.

### Why use the same Docker image for staging and production?

> It ensures that the exact image tested in staging is the one deployed to production, reducing environment and build differences.

### What is the purpose of `docker push`?

> It uploads a Docker image to a registry so other systems can pull and deploy it.

### What is the purpose of `docker pull`?

> It downloads a specific image from the registry to the machine where the container will run.

### What happens after `docker pull`?

> The server can create a running container from that image using commands such as `docker run`.

---

## 🧠 Remember

~~~text
Jenkins = Automation
Docker = Package + Runtime
Registry = Store + Distribute
Server = Run Container
~~~

The most important production flow is:

~~~text
GitHub
   ↓
Jenkins
   ↓
Docker Build
   ↓
Image tagged with Git SHA
   ↓
Docker Login
   ↓
Docker Push
   ↓
Registry
   ↓
Server
   ↓
Docker Pull
   ↓
Docker Run
   ↓
Container
~~~

**Build once → Push once → Pull the same image → Deploy the same image.**

**Jenkins orchestrates the process; Docker packages and runs the application.**
