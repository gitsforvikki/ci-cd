# Lesson 17 — Deployment Strategies

## What is Deployment?

Deployment means taking a tested application version and making it available in a target environment such as staging or production.

~~~text
GitHub
   ↓
Jenkins
   ↓
Build / Test
   ↓
Artifact or Docker Image
   ↓
Deployment
   ↓
Production
~~~

Different deployment strategies decide how Jenkins transfers and starts the application on the production server.

---

## 1. SSH Deployment

SSH (Secure Shell) allows Jenkins or an administrator to securely connect to a remote Linux server.

Example:

~~~bash
ssh user@server
~~~

Jenkins can use SSH to:

- Connect to a server
- Run deployment commands
- Restart an application
- Check application status

~~~text
Jenkins
   ↓
SSH
   ↓
Production Server
   ↓
Deployment Commands
~~~

**Interview point:** SSH is a secure remote access mechanism. It is commonly used as the transport/control mechanism for deployment, not a complete deployment strategy by itself.

---

## 2. SCP / rsync

### SCP

SCP copies files over SSH.

~~~bash
scp app.zip user@server:/opt/app/
~~~

### rsync

rsync synchronizes files efficiently and usually works over SSH.

~~~bash
rsync -avz ./dist/ user@server:/var/www/app/
~~~

Simple difference:

~~~text
SCP
→ Simple file copy

rsync
→ Efficient synchronization
~~~

For repeated deployments, rsync can avoid transferring files that have not changed.

---

## 3. Docker Deployment

Instead of copying application files directly to the server, Jenkins can build a Docker image and deploy that image.

~~~text
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
Docker Pull
   ↓
Container
~~~

Example:

~~~bash
docker pull codebuddy:9ab42ef

docker run -d   --name codebuddy   -p 3000:3000   codebuddy:9ab42ef
~~~

Important principle:

**Build once → Deploy the same image**

---

## 4. Nginx

Nginx is commonly used as a reverse proxy in front of an application.

Without a reverse proxy:

~~~text
User
  ↓
Node.js :3000
~~~

With Nginx:

~~~text
User
  ↓
HTTPS
  ↓
Nginx :443
  ↓
Node.js :3000
~~~

Nginx can handle:

- HTTPS/TLS termination
- Reverse proxying
- Domain routing
- Static files
- Load balancing
- Request forwarding

Example:

~~~text
example.com
     ↓
   Nginx
     ↓
localhost:3000
     ↓
Node.js
~~~

**Interview point:** Nginx commonly sits in front of the application. It does not normally replace the application server.

---

## 5. PM2

PM2 is a process manager commonly used for Node.js applications.

Without a process manager:

~~~text
Node.js Process
     ↓
Process crashes
     ↓
Application stops
~~~

With PM2:

~~~text
PM2
 ↓
Node.js
 ↓
Application
~~~

PM2 can help with:

- Starting applications
- Restarting crashed processes
- Managing multiple Node.js processes
- Application logs
- Startup configuration

Examples:

~~~bash
pm2 start server.js --name codebuddy
pm2 status
pm2 restart codebuddy
~~~

---

## 6. systemd

systemd is the service manager commonly used by modern Linux distributions.

Conceptually:

~~~text
systemd
   ↓
Node.js Application
   ↓
Application Process
~~~

A systemd service can provide:

- Automatic startup
- Automatic restart
- Service management
- Dependency ordering
- Service status

Useful commands:

~~~bash
systemctl status codebuddy
systemctl restart codebuddy
systemctl start codebuddy
systemctl stop codebuddy
~~~

### PM2 vs systemd

~~~text
PM2
→ Node.js-focused process manager

systemd
→ Linux system/service manager
~~~

---

## 7. Container Deployment

In a container-based deployment, the application runs inside a container.

~~~text
Docker Image
     ↓
Container
     ↓
Application
~~~

The container can be placed behind Nginx:

~~~text
User
 ↓
Nginx
 ↓
Docker Container
 ↓
Node.js
 ↓
Database
~~~

Important distinction:

**Docker image = packaged application**

**Container = running instance of the image**

---

## 8. Deployment Scripts

Instead of manually typing deployment commands, create a script.

Example:

~~~bash
#!/bin/bash

set -e

docker pull codebuddy:9ab42ef

docker stop codebuddy || true
docker rm codebuddy || true

docker run -d   --name codebuddy   -p 3000:3000   codebuddy:9ab42ef
~~~

Jenkins can execute the script:

~~~text
Jenkins
   ↓
Deployment Script
   ↓
Server
   ↓
Application
~~~

### Why use scripts?

- Repeatability
- Fewer manual mistakes
- Easier automation
- Easier troubleshooting
- Deployment logic can be version controlled

A production script should also handle configuration, health checks, logging, and safe failure behavior.

---

## 9. Restarting Applications

After deploying new code, the application process may need to restart.

### PM2

~~~bash
pm2 restart codebuddy
~~~

### systemd

~~~bash
systemctl restart codebuddy
~~~

### Docker

A container can be replaced with a new container using the new image.

~~~text
Old Container
     ↓
Stop / Replace
     ↓
New Container
~~~

A simple stop-and-replace deployment may cause downtime.

---

## 10. Zero / Minimal Downtime Basics

A simple deployment can cause downtime:

~~~text
Users
  ↓
Old App
  ↓
STOP
  ↓
Deploy New App
  ↓
START
  ↓
New App
~~~

The goal of zero/minimal downtime deployment is:

~~~text
Users
  ↓
Running Application
     ↓
Deploy New Version
     ↓
Health Check
     ↓
Switch Traffic
     ↓
New Version
~~~

### Rolling Deployment

Replace instances gradually.

~~~text
v1  v1  v1
 ↓
v2  v1  v1
 ↓
v2  v2  v1
 ↓
v2  v2  v2
~~~

### Blue-Green Deployment

Keep two environments.

~~~text
Users
  ↓
Blue (v1) ← Current

Green (v2) ← New
~~~

After testing green, switch traffic:

~~~text
Users
  ↓
Green (v2)
~~~

Rollback can be as simple as switching traffic back to blue.

### Canary Deployment

Send a small percentage of traffic to the new version first.

~~~text
Users
   │
   ├── 95% → v1
   └── 5%  → v2
~~~

If v2 is healthy, increase traffic gradually.

---

## 11. Deployment Methods Together

~~~text
SSH + SCP/rsync
→ Copy application files to server

Docker
→ Package application as an image

PM2
→ Manage Node.js processes

systemd
→ Manage Linux services

Nginx
→ Reverse proxy / HTTPS / routing

Container
→ Isolated application runtime

Deployment Script
→ Automate repeatable deployment steps
~~~

These technologies can be combined.

Example:

~~~text
Jenkins
   ↓
SSH
   ↓
Production Server
   ↓
Docker
   ↓
Container
   ↓
Nginx
   ↓
Users
~~~

---

## 12. Example Production Flows

### Docker-based deployment

~~~text
Developer
    ↓
GitHub
    ↓
Jenkins
    ↓
Test
    ↓
Docker Build
    ↓
Docker Image
    ↓
Registry
    ↓
SSH / Deployment Automation
    ↓
Production Server
    ↓
Docker Pull
    ↓
New Container
    ↓
Health Check
    ↓
Nginx
    ↓
Users
~~~

### PM2-based deployment

~~~text
Developer
    ↓
GitHub
    ↓
Jenkins
    ↓
Build
    ↓
SCP / rsync
    ↓
Production Server
    ↓
PM2 Restart
    ↓
Nginx
    ↓
Users
~~~

---

## 13. Important Production Considerations

### Do not blindly restart production

A restart can cause downtime.

Prefer:

- Health checks
- Rolling deployment
- Blue-green deployment
- Canary deployment
- Load balancing
- Automatic rollback

### Keep deployment repeatable

The same deployment should produce the expected result every time.

### Keep versions identifiable

Prefer:

~~~text
codebuddy:9ab42ef
~~~

over relying only on:

~~~text
codebuddy:latest
~~~

### Keep secrets outside the application package

Use environment configuration and proper secret-management mechanisms.

---

## ⭐ Interview Points

### What is SSH deployment?

> SSH deployment uses a secure remote connection to access a target server and execute deployment commands.

### What is the difference between SCP and rsync?

> SCP is primarily a straightforward file-copy mechanism, while rsync synchronizes files efficiently and can avoid transferring unchanged data.

### Why use Docker for deployment?

> Docker packages the application and runtime into a consistent image, reducing environment differences between systems.

### What is Nginx's role?

> Nginx commonly acts as a reverse proxy in front of the application, handling HTTPS, routing, and request forwarding.

### What is PM2?

> PM2 is a Node.js process manager that helps start, monitor, and restart Node.js applications.

### What is systemd?

> systemd is the Linux service manager used to manage long-running services, including starting them at boot and restarting them when configured to do so.

### PM2 vs systemd?

> PM2 is focused on Node.js process management, while systemd is the operating system's service manager.

### What is a deployment script?

> A deployment script is a repeatable set of commands used to automate application deployment and reduce manual errors.

### How can you achieve minimal downtime?

> Use strategies such as rolling, blue-green, or canary deployments, combined with health checks and controlled traffic switching.

### Does Docker automatically provide zero downtime?

> No. Docker provides containerization. Zero/minimal downtime requires an appropriate deployment strategy and traffic management.

---

## 🧠 Remember

~~~text
SSH
→ Secure remote access

SCP / rsync
→ Transfer application files

Docker
→ Package application

PM2
→ Manage Node.js process

systemd
→ Manage Linux service

Nginx
→ Reverse proxy / HTTPS / routing

Deployment Script
→ Automate deployment

Rolling / Blue-Green / Canary
→ Reduce deployment downtime
~~~

### Most important deployment mental model

~~~text
                 Jenkins
                    ↓
             Deployment Method
              /       |                    ↓        ↓        ↓
           SSH      Docker    Kubernetes
             ↓        ↓
          Server   Container
             ↓        ↓
             └── Nginx ──┘
                    ↓
                  Users
~~~

**Interview takeaway:**

> Deployment is not just copying code to a server. A production deployment needs a reliable way to transfer the version, start/manage the application, expose it through the network, verify its health, and minimize downtime during the transition.
