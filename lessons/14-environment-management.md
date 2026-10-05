# Lesson 14 — Environment Management

## What is Environment Management?

Most applications run in more than one environment.

~~~text
Development
     ↓
Staging
     ↓
Production
~~~

Each environment can have different configuration values.

The application code can be mostly the same, while the configuration changes.

---

## 1. Development Environment

Development is where developers work on the application.

~~~text
Developer
   ↓
Local Machine
   ↓
Development Environment
~~~

Typical values might be:

~~~text
DATABASE_URL=local-database
API_URL=http://localhost:3000
~~~

---

## 2. Staging Environment

Staging is a production-like environment used for testing before production.

~~~text
Development
    ↓
Staging
    ↓
Production
~~~

The team can test the application using staging configuration before releasing it to users.

---

## 3. Production Environment

Production is the environment used by real users.

~~~text
Users
  ↓
Production
  ↓
Production Database
~~~

Production values are sensitive.

Examples:

- Database passwords
- API keys
- JWT secrets
- Payment credentials

These should be stored securely.

---

## 4. Environment Variables

Environment variables allow configuration to be provided without hard-coding it into application code.

Instead of:

~~~js
const apiUrl = "https://api.example.com";
~~~

we can use:

~~~js
const apiUrl = process["env"].API_URL;
~~~

Simple idea:

~~~text
Application Code
      ↓
Environment Variable
      ↓
Actual value
~~~

The same code can then run in different environments with different values.

---

## 5. .env Files

During local development, we commonly use `.env` files.

Example:

~~~text
.env
----
API_URL=http://localhost:3000
DATABASE_URL=local-database
~~~

For example:

~~~text
.env.development
.env.staging
.env.production
~~~

The exact files supported depend on the framework and setup.

### Important

Do not commit sensitive `.env` files to Git.

Usually we add them to `.gitignore`.

---

## 6. Jenkins Environment Variables

Jenkins can also provide environment variables to a pipeline.

Example:

~~~groovy
pipeline {
    agent any

    environment {
        NODE_ENV = 'production'
    }

    stages {
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}
~~~

Simple flow:

~~~text
Jenkins
   ↓
Environment Variables
   ↓
Pipeline
   ↓
Application Build
~~~

---

## 7. Secrets Per Environment

Development, staging, and production should not normally share the same secrets.

~~~text
Development
   ↓
Dev Database Credentials

Staging
   ↓
Staging Database Credentials

Production
   ↓
Production Database Credentials
~~~

This limits the damage if one environment's credentials are exposed.

Never put production secrets directly inside the Jenkinsfile or source code.

---

## 8. Build-Time vs Runtime Variables

This is very important for modern applications.

### Build-time variable

A value is needed while the application is being built.

~~~text
Source Code
    ↓
Build
    ↑
Environment Variable
    ↓
Build Output
~~~

### Runtime variable

A value is provided when the application is running.

~~~text
Application
    ↓
Starts
    ↓
Reads Runtime Configuration
~~~

For Next.js, some environment values can become part of the browser bundle when intentionally exposed using the appropriate public prefix. Therefore, never treat a browser-exposed environment variable as a secret.

---

## 9. Why Build-Time vs Runtime Matters

Imagine we build an application for staging:

~~~text
Staging Variable
      ↓
Build
      ↓
Staging Build
~~~

If we want to use exactly the same build in production, we need to understand which values were already included during the build and which values can be supplied at runtime.

Important CI/CD principle:

**Build once, promote the same artifact when possible.**

---

## 10. Simple CI/CD Environment Flow

~~~text
GitHub
   ↓
Jenkins
   ↓
Build
   ↓
Artifact
   ↓
Staging
   ↓
Test
   ↓
Approval
   ↓
Production
~~~

Configuration changes by environment:

~~~text
             Same Application
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
   Development   Staging    Production
        │           │           │
      Dev Env    Stage Env    Prod Env
~~~

---

## 11. What Should Not Be Stored in Git?

Never commit sensitive values such as:

- Database passwords
- Private API keys
- JWT secrets
- Cloud credentials
- Production credentials

Bad:

~~~js
const password = "my-real-password";
~~~

Better:

~~~text
Git
 ↓
Code
 ↓
Jenkins / Secret Store
 ↓
Environment
 ↓
Application
~~~

---

## 12. Common Mistakes

### Mistake 1: Same secrets everywhere

Using production credentials in development is dangerous.

### Mistake 2: Committing `.env`

A secret committed to Git can remain in Git history even after the file is deleted.

### Mistake 3: Confusing public and private variables

Anything exposed to the browser should be considered public.

### Mistake 4: Rebuilding unnecessarily

If configuration can be supplied at runtime, rebuilding for every environment can make deployments less consistent.

---

## ⭐ Interview Points

### What are development, staging, and production?

> They are separate environments used for development, testing, and serving real users.

### Why use environment variables?

> They keep configuration outside application code and allow the same application to run with different environment-specific values.

### Should `.env` files be committed?

> Sensitive `.env` files should not be committed to Git. Secrets should be stored securely.

### What is the difference between build-time and runtime variables?

> Build-time variables are available while creating the application build, while runtime variables are provided when the application runs.

### Why should production secrets be different from staging secrets?

> Environment-specific secrets reduce the impact of a compromised environment and prevent accidental access to production resources.

### Why is build-time vs runtime important for CI/CD?

> It affects whether the same artifact can be promoted from staging to production without rebuilding it.

---

## 🧠 Remember

~~~text
Development
     ↓
Staging
     ↓
Production

Same Code
   +
Different Configuration
   ↓
Different Environment
~~~

And:

~~~text
Code
  ↓
Jenkins
  ↓
Environment Variables / Secrets
  ↓
Build or Runtime
  ↓
Application
~~~

**Keep configuration outside the code.**

**Keep secrets out of Git.**

**Use separate secrets for separate environments.**

**Understand whether a variable is needed at build time or runtime.**
