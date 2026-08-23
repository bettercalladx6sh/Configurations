# Node.js CI/CD with Docker and AWS EC2

This README explains a GitHub Actions workflow that:

1. Runs when code is pushed to `main` or a Pull Request targets `main`.
2. Sets up Node.js.
3. Installs dependencies.
4. Builds the application.
5. Builds a Docker image.
6. Pushes the image to Docker Hub.
7. Connects to an AWS EC2 instance through SSH.
8. Pulls the latest Docker image.
9. Stops the old container.
10. Starts a new container.

---

# Complete Workflow

Create:

```text
.github/workflows/node-ci.yml
```

```yaml
name: Node.js CI

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [22.x]

    steps:
      - uses: actions/checkout@v4

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"

      - run: npm ci
      - run: npm run build

      # Docker Workflow
      - name: Login to Docker Hub
        uses: docker/login-action@v4
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v4

      - name: Build and push
        uses: docker/build-push-action@v7
        with:
          push: true
          tags: adx6sh/adx6sh.sh:latest

  deploy:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Deploy to Instance
        uses: appleboy/ssh-action@v1.2.2
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            docker pull adx6sh/adx6sh.sh:latest

            docker stop adx6sh-app || true
            docker rm adx6sh-app || true

            docker run -d \
              --name adx6sh-app \
              -p 3000:3000 \
              --restart unless-stopped \
              adx6sh/adx6sh.sh:latest
```

---

# Line-by-Line Explanation

## 1. Workflow Name

```yaml
name: Node.js CI
```

`name` defines the name of the GitHub Actions workflow.

GitHub will show this name in the **Actions** tab.

For example:

```text
Actions
 └── Node.js CI
```

You can change it to something like:

```yaml
name: Node.js CI/CD
```

---

# 2. Workflow Triggers

```yaml
on:
```

`on` defines **when GitHub Actions should run the workflow**.

Think of it as:

> "What event should trigger this workflow?"

---

## Push Trigger

```yaml
push:
```

This tells GitHub to run the workflow when someone pushes code to the repository.

For example:

```bash
git push origin main
```

---

## Branch Filter

```yaml
branches: ["main"]
```

This limits the push trigger to the `main` branch.

So:

```text
push to main
      ↓
workflow runs
```

But:

```text
push to development
      ↓
workflow does NOT run
```

---

# 3. Pull Request Trigger

```yaml
pull_request:
```

This tells GitHub to run the workflow when a Pull Request event occurs.

For example:

```text
feature branch
      ↓
Pull Request
      ↓
main
```

---

```yaml
branches: ["main"]
```

This means the Pull Request must target the `main` branch.

For example:

```text
feature/login → main
```

will trigger the workflow.

But:

```text
feature/login → development
```

will not trigger this workflow.

---

# 4. Jobs

```yaml
jobs:
```

`jobs` contains the different jobs that GitHub Actions needs to execute.

This workflow has two jobs:

```text
jobs
 ├── build
 └── deploy
```

The `build` job builds and pushes the Docker image.

The `deploy` job deploys that image to EC2.

---

# 5. Build Job

```yaml
build:
```

This creates a job named `build`.

You can name it anything:

```yaml
build:
```

or:

```yaml
docker:
```

or:

```yaml
build-and-push:
```

---

# 6. Runner

```yaml
runs-on: ubuntu-latest
```

This tells GitHub which machine should execute the job.

GitHub creates a temporary Ubuntu virtual machine for the workflow.

Conceptually:

```text
GitHub
   │
   ▼
Ubuntu VM
   │
   ├── Node.js
   ├── npm
   ├── Docker
   └── Git
```

The VM is used to execute your workflow.

---

# 7. Strategy

```yaml
strategy:
```

`strategy` allows you to control how a job is executed, especially when using a matrix.

---

# 8. Matrix

```yaml
matrix:
```

A matrix allows you to run the same job with different configurations.

For example:

```yaml
matrix:
  node-version: [20.x, 22.x]
```

would run the job twice:

```text
Node.js 20
Node.js 22
```

Your workflow currently has:

```yaml
matrix:
  node-version: [22.x]
```

So it only runs Node.js 22.

---

# 9. Steps

```yaml
steps:
```

`steps` contains the individual actions/commands that the job executes.

Think of it as a list:

```text
Step 1
Step 2
Step 3
Step 4
...
```

Each step runs in order.

---

# 10. Checkout Repository

```yaml
- uses: actions/checkout@v4
```

This downloads your GitHub repository into the GitHub Actions runner.

Without this, the runner doesn't have your project files.

Conceptually:

```text
GitHub Repository
       │
       │ checkout
       ▼
Ubuntu Runner
       │
       └── your project files
```

`actions/checkout@v4` is a reusable GitHub Action provided for checking out repositories.

---

# 11. Node.js Setup

```yaml
- name: Use Node.js ${{ matrix.node-version }}
```

`name` gives this step a readable name.

The expression:

```text
${{ matrix.node-version }}
```

gets the Node.js version from the matrix.

Since the matrix contains:

```yaml
node-version: [22.x]
```

the step effectively becomes:

```text
Use Node.js 22.x
```

---

```yaml
uses: actions/setup-node@v4
```

This uses GitHub's official Node.js setup action.

It installs/configures the requested Node.js version on the runner.

---

# 12. With

```yaml
with:
```

`with` provides configuration options to the action.

In this case, we configure Node.js.

---

```yaml
node-version: ${{ matrix.node-version }}
```

This tells `setup-node` which Node.js version to use.

The matrix contains:

```yaml
node-version: [22.x]
```

Therefore:

```text
Node.js 22.x
```

is configured.

---

# 13. npm Cache

```yaml
cache: "npm"
```

This enables npm dependency caching.

When dependencies have already been downloaded in previous workflow runs, GitHub can reuse cached files.

This can make subsequent workflow runs faster.

Conceptually:

```text
First run
   ↓
Download dependencies
   ↓
Cache dependencies

Next run
   ↓
Restore cache
   ↓
Faster npm install
```

---

# 14. Install Dependencies

```yaml
- run: npm ci
```

`run` executes a shell command on the GitHub Actions runner.

The command is:

```bash
npm ci
```

`npm ci` installs dependencies from `package-lock.json`.

It is commonly preferred in CI environments because it performs a clean, reproducible installation.

Conceptually:

```text
package.json
     +
package-lock.json
     ↓
npm ci
     ↓
node_modules/
```

---

# 15. Build Application

```yaml
- run: npm run build
```

This executes the `build` script from your `package.json`.

For example:

```json
{
  "scripts": {
    "build": "next build"
  }
}
```

Then:

```bash
npm run build
```

executes:

```bash
next build
```

If the build fails, the workflow stops here.

---

# Docker Workflow

```yaml
# Docker Workflow
```

This is a YAML comment.

Comments start with:

```text
#
```

GitHub ignores this line.

It is only there to make the workflow easier for humans to understand.

---

# 16. Docker Hub Login

```yaml
- name: Login to Docker Hub
```

This creates a workflow step named:

```text
Login to Docker Hub
```

---

```yaml
uses: docker/login-action@v4
```

This uses Docker's login GitHub Action.

It authenticates the GitHub Actions runner with Docker Hub.

---

# 17. Docker Username

```yaml
with:
  username: ${{ secrets.DOCKERHUB_USERNAME }}
```

This provides the Docker Hub username.

The value comes from GitHub Secrets.

You should create:

```text
DOCKERHUB_USERNAME
```

inside:

```text
Repository
 → Settings
 → Secrets and variables
 → Actions
```

The actual username isn't written directly into the workflow.

---

# 18. Docker Token

```yaml
password: ${{ secrets.DOCKERHUB_TOKEN }}
```

This provides the Docker Hub access token.

Again, it is stored securely inside GitHub Secrets.

You should create:

```text
DOCKERHUB_TOKEN
```

The workflow can access it through:

```text
${{ secrets.DOCKERHUB_TOKEN }}
```

This is safer than writing your Docker password directly in the YAML file.

---

# 19. QEMU

```yaml
- name: Set up QEMU
```

This gives the step a name.

---

```yaml
uses: docker/setup-qemu-action@v4
```

QEMU is used for CPU architecture emulation.

It is useful when building Docker images for architectures different from the runner's architecture.

For example:

```text
amd64
arm64
```

If you are only building for the same architecture, QEMU may not always be necessary, but it is commonly included in Docker build workflows.

---

# 20. Docker Buildx

```yaml
- name: Set up Docker Buildx
```

This creates a step for configuring Docker Buildx.

---

```yaml
uses: docker/setup-buildx-action@v4
```

Buildx is Docker's modern build system.

It provides features such as:

- Advanced Docker builds
- Build cache
- Multi-platform builds
- Better build features

---

# 21. Build and Push Docker Image

```yaml
- name: Build and push
```

This gives the step its name.

---

```yaml
uses: docker/build-push-action@v7
```

This action builds the Docker image and can push it to a container registry.

In this workflow, it does both.

---

# 22. Push

```yaml
with:
  push: true
```

This tells the Docker action:

> After building the image, push it to the registry.

If it were:

```yaml
push: false
```

the image would only be built on the GitHub Actions runner.

---

# 23. Docker Image Tag

```yaml
tags: adx6sh/adx6sh.sh:latest
```

This specifies the name and tag of the Docker image.

The structure is:

```text
USERNAME/IMAGE_NAME:TAG
```

So:

```text
adx6sh
```

is the Docker Hub username.

```text
adx6sh.sh
```

is the repository/image name.

```text
latest
```

is the image tag.

Therefore:

```text
adx6sh/adx6sh.sh:latest
```

identifies the Docker image.

---

# Deploy Job

```yaml
deploy:
```

This creates another job called `deploy`.

The workflow now looks like:

```text
jobs
 ├── build
 └── deploy
```

---

# 24. Deploy Runner

```yaml
runs-on: ubuntu-latest
```

The deployment job also runs on a temporary GitHub-hosted Ubuntu machine.

Important:

**This is not your EC2 server.**

It is a temporary GitHub Actions runner that will connect to your EC2 server.

The architecture is:

```text
GitHub Actions Runner
        │
        │ SSH
        ▼
     AWS EC2
```

---

# 25. Needs

```yaml
needs: build
```

This is very important.

It means:

> The `deploy` job must wait until the `build` job successfully finishes.

Without it, both jobs could potentially start independently.

With:

```yaml
needs: build
```

the flow becomes:

```text
build
  │
  │ success
  ▼
deploy
```

If `build` fails:

```text
build
  │
  │ failed
  X
deploy doesn't run
```

---

# 26. Deployment Steps

```yaml
steps:
```

This starts the list of steps for the deployment job.

---

# 27. Deployment Step Name

```yaml
- name: Deploy to Instance
```

This gives the deployment step a readable name.

---

# 28. SSH Action

```yaml
uses: appleboy/ssh-action@v1.2.2
```

This uses the `appleboy/ssh-action` GitHub Action.

Its purpose is to connect from the GitHub Actions runner to your EC2 server using SSH and execute commands there.

Conceptually:

```text
GitHub Actions
      │
      │ SSH
      ▼
AWS EC2
```

---

# 29. SSH Configuration

```yaml
with:
```

This provides the configuration for the SSH action.

---

# 30. EC2 Host

```yaml
host: ${{ secrets.EC2_HOST }}
```

This specifies the address of your EC2 instance.

For example, the secret could contain:

```text
13.234.xxx.xxx
```

or an EC2 public DNS name.

The actual value is stored as:

```text
EC2_HOST
```

inside GitHub Secrets.

---

# 31. EC2 Username

```yaml
username: ${{ secrets.EC2_USERNAME }}
```

This specifies the Linux username used for SSH.

For Ubuntu EC2 instances, it is commonly:

```text
ubuntu
```

For Amazon Linux, it is commonly:

```text
ec2-user
```

The value is stored as:

```text
EC2_USERNAME
```

---

# 32. SSH Private Key

```yaml
key: ${{ secrets.EC2_SSH_KEY }}
```

This provides the SSH private key used to authenticate with the EC2 instance.

The private key is stored inside GitHub Secrets as:

```text
EC2_SSH_KEY
```

You should **never commit your private key to GitHub**.

---

# 33. Remote Script

```yaml
script: |
```

This tells the SSH action:

> Run the following commands on the EC2 instance.

The `|` allows multiple lines of shell commands.

Everything below it is executed remotely on EC2.

---

# 34. Pull Latest Docker Image

```bash
docker pull adx6sh/adx6sh.sh:latest
```

This downloads the latest Docker image from Docker Hub.

The flow is:

```text
Docker Hub
    │
    │ docker pull
    ▼
EC2
    │
    ▼
Local Docker image
```

The EC2 machine now has:

```text
adx6sh/adx6sh.sh:latest
```

---

# 35. Stop Existing Container

```bash
docker stop adx6sh-app || true
```

This attempts to stop the existing container named:

```text
adx6sh-app
```

Normally:

```bash
docker stop adx6sh-app
```

would return an error if the container doesn't exist.

The:

```bash
|| true
```

means:

> If the command fails, don't fail the deployment.

For example:

```text
Container exists
     ↓
docker stop
     ↓
Success
```

or:

```text
Container doesn't exist
     ↓
docker stop fails
     ↓
true
     ↓
Continue deployment
```

This is useful during the first deployment when the container may not exist yet.

---

# 36. Remove Existing Container

```bash
docker rm adx6sh-app || true
```

This removes the stopped container.

Again:

```bash
|| true
```

prevents the deployment from failing if the container doesn't exist.

The sequence is:

```text
docker stop
     ↓
docker rm
```

---

# 37. Start New Container

```bash
docker run -d \
```

`docker run` creates and starts a new container.

The `-d` means:

```text
detached mode
```

The container runs in the background.

Without `-d`, the terminal would remain attached to the container's output.

---

# 38. Container Name

```bash
  --name adx6sh-app \
```

This gives the container the name:

```text
adx6sh-app
```

So instead of dealing with a randomly generated container name, you can use:

```bash
docker stop adx6sh-app
```

```bash
docker logs adx6sh-app
```

```bash
docker restart adx6sh-app
```

---

# 39. Port Mapping

```bash
  -p 3000:3000 \
```

This maps:

```text
HOST_PORT:CONTAINER_PORT
```

So:

```text
3000:3000
```

means:

```text
EC2 port 3000
      │
      ▼
Container port 3000
```

If your application listens on port `3000` inside the container, users can access it through port `3000` on the EC2 server.

For example:

```text
http://EC2_PUBLIC_IP:3000
```

---

# 40. Restart Policy

```bash
  --restart unless-stopped \
```

This tells Docker to automatically restart the container if it stops unexpectedly.

For example:

```text
Application crashes
       ↓
Docker restarts container
```

It also helps when the EC2 server restarts.

The `unless-stopped` policy means Docker will restart the container unless you intentionally stopped it.

---

# 41. Docker Image

```bash
  adx6sh/adx6sh.sh:latest
```

This is the image used to create the container.

The complete command is effectively:

```bash
docker run \
  -d \
  --name adx6sh-app \
  -p 3000:3000 \
  --restart unless-stopped \
  adx6sh/adx6sh.sh:latest
```

Docker takes:

```text
adx6sh/adx6sh.sh:latest
```

and creates a running container from it.

---

# Complete CI/CD Flow

The complete process is:

```text
Developer
    │
    │ git push
    ▼
GitHub
    │
    ▼
GitHub Actions
    │
    ├── Checkout code
    │
    ├── Setup Node.js 22
    │
    ├── npm ci
    │
    ├── npm run build
    │
    ├── Login to Docker Hub
    │
    ├── Setup QEMU
    │
    ├── Setup Docker Buildx
    │
    ├── Build Docker image
    │
    └── Push image
             │
             ▼
        Docker Hub
             │
             │ docker pull
             ▼
           EC2
             │
             ├── Stop old container
             │
             ├── Remove old container
             │
             └── Start new container
```

---

# GitHub Secrets

The workflow expects these five secrets:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN

EC2_HOST
EC2_USERNAME
EC2_SSH_KEY
```

They should be configured under:

```text
GitHub Repository
        ↓
Settings
        ↓
Secrets and variables
        ↓
Actions
        ↓
New repository secret
```

## Why use Secrets?

You should not write sensitive information directly into the workflow.

Bad:

```yaml
username: myusername
password: mypassword
```

Better:

```yaml
username: ${{ secrets.DOCKERHUB_USERNAME }}
password: ${{ secrets.DOCKERHUB_TOKEN }}
```

GitHub stores the sensitive values separately from your workflow code.

---

# CI vs CD

This workflow contains both **CI** and **CD**.

## Continuous Integration

These parts are CI:

```text
Checkout
   ↓
Setup Node.js
   ↓
npm ci
   ↓
npm run build
```

The purpose is to verify that the application can be installed and built successfully.

---

## Continuous Deployment

These parts are CD:

```text
Build Docker image
       ↓
Push to Docker Hub
       ↓
SSH to EC2
       ↓
Pull image
       ↓
Stop old container
       ↓
Start new container
```

The application is automatically deployed after a successful build.

---

# Overall Architecture

```text
                  GitHub
                    │
                    │ Push to main
                    ▼
             GitHub Actions
                    │
            ┌───────┴───────┐
            │               │
            ▼               │
       Build Application    │
            │               │
            ▼               │
       Docker Build         │
            │               │
            ▼               │
        Docker Hub          │
            │               │
            │               │
            ▼               │
           EC2 ◄────────────┘
            │
            ▼
       Docker Container
            │
            │ :3000
            ▼
       Web Application
```

The important idea is:

```text
Code
 ↓
GitHub
 ↓
CI
 ↓
Docker Image
 ↓
Docker Hub
 ↓
EC2
 ↓
Docker Container
 ↓
Application
```
