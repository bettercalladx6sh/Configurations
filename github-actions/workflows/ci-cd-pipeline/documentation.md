# Node.js CI/CD Pipeline with Docker & AWS EC2

A complete, practical CI/CD pipeline for a Node.js application using **GitHub Actions**, **Docker**, **Docker Hub**, **SSH**, and **AWS EC2**.

This repository demonstrates how source code can move automatically from a Git push to a running Docker container on an EC2 server.

> **Pipeline:** Code → GitHub → GitHub Actions → Node.js Build → Docker Image → Docker Hub → SSH → EC2 → Docker Container

---

## Table of Contents

- [1. Overview](#1-overview)
- [2. What This Project Does](#2-what-this-project-does)
- [3. Technology Stack](#3-technology-stack)
- [4. Architecture](#4-architecture)
- [5. End-to-End Flow](#5-end-to-end-flow)
- [6. Repository Structure](#6-repository-structure)
- [7. Prerequisites](#7-prerequisites)
- [8. Dockerfile](#8-dockerfile)
- [9. GitHub Actions Workflow](#9-github-actions-workflow)
- [10. Workflow Line-by-Line](#10-workflow-line-by-line)
- [11. GitHub Secrets](#11-github-secrets)
- [12. Docker Hub Setup](#12-docker-hub-setup)
- [13. AWS EC2 Setup](#13-aws-ec2-setup)
- [14. SSH Authentication](#14-ssh-authentication)
- [15. Docker Port Mapping](#15-docker-port-mapping)
- [16. Deployment Lifecycle](#16-deployment-lifecycle)
- [17. First Deployment](#17-first-deployment)
- [18. Subsequent Deployments](#18-subsequent-deployments)
- [19. CI vs CD](#19-ci-vs-cd)
- [20. Failure Behavior](#20-failure-behavior)
- [21. Security](#21-security)
- [22. Troubleshooting](#22-troubleshooting)
- [23. Useful Commands](#23-useful-commands)
- [24. Production Improvements](#24-production-improvements)
- [25. Mental Model](#25-mental-model)
- [26. Final Checklist](#26-final-checklist)
- [27. References](#27-references)

---

# 1. Overview

This project implements a basic but complete deployment pipeline for a Node.js application.

A developer only needs to push code:

```bash
git push origin main
```

GitHub Actions then automatically:

1. Checks out the source code.
2. Installs Node.js 22.
3. Restores npm cache.
4. Installs dependencies using `npm ci`.
5. Builds the application using `npm run build`.
6. Authenticates with Docker Hub.
7. Configures Docker Buildx and QEMU.
8. Builds a Docker image.
9. Pushes the image to Docker Hub.
10. Connects to AWS EC2 over SSH.
11. Pulls the latest Docker image.
12. Stops the previous container.
13. Removes the previous container.
14. Starts a new container.
15. Exposes the application through port `3000`.

The result is an automated deployment pipeline.

---

# 2. What This Project Does

The pipeline has two major stages:

```text
┌──────────────────────────────┐
│            BUILD             │
│                              │
│ Checkout                     │
│ Setup Node.js                │
│ Install dependencies         │
│ Build application            │
│ Login to Docker Hub          │
│ Build Docker image           │
│ Push Docker image            │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│           DEPLOY             │
│                              │
│ SSH into EC2                 │
│ Pull latest image            │
│ Stop old container           │
│ Remove old container         │
│ Start new container          │
└──────────────────────────────┘
```

The `deploy` job depends on the successful completion of the `build` job.

```yaml
needs: build
```

GitHub Actions uses job dependencies to control execution order. If a required job fails, dependent jobs are skipped by default. citeturn0search2turn0search1

---

# 3. Technology Stack

| Technology | Purpose |
|---|---|
| Node.js | Application runtime and build environment |
| npm | Dependency management |
| GitHub | Source-code hosting |
| GitHub Actions | CI/CD automation |
| Docker | Application containerization |
| Docker Hub | Docker image registry |
| Docker Buildx | Modern Docker image builder |
| QEMU | CPU architecture emulation |
| SSH | Remote EC2 access |
| AWS EC2 | Deployment server |

Docker provides official GitHub Actions for logging into registries, configuring Buildx/QEMU, and building/pushing images. citeturn0search0

---

# 4. Architecture

## High-Level Architecture

```text
                         Developer
                             │
                             │ git push origin main
                             ▼
                    ┌──────────────────┐
                    │ GitHub Repository │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │  GitHub Actions  │
                    │                  │
                    │      BUILD       │
                    │                  │
                    │ checkout         │
                    │ Node.js 22       │
                    │ npm ci           │
                    │ npm run build    │
                    │ Docker build     │
                    │ Docker push      │
                    └────────┬─────────┘
                             │
                             │ push image
                             ▼
                    ┌──────────────────┐
                    │    Docker Hub    │
                    │                  │
                    │ adx6sh/adx6sh.sh│
                    │      :latest     │
                    └────────┬─────────┘
                             │
                             │ docker pull
                             ▼
                    ┌──────────────────┐
                    │     AWS EC2      │
                    │                  │
                    │ Docker Engine    │
                    │       │          │
                    │       ▼          │
                    │  adx6sh-app      │
                    │  :3000 → :3000   │
                    └────────┬─────────┘
                             │
                             ▼
                         Application
```

---

# 5. End-to-End Flow

The complete deployment can be understood as six phases.

## Phase 1 — Source Code

The developer changes the application and pushes to `main`.

```bash
git add .
git commit -m "update application"
git push origin main
```

GitHub receives the push event.

---

## Phase 2 — Continuous Integration

GitHub Actions starts the workflow.

```text
GitHub
  ↓
Checkout
  ↓
Node.js setup
  ↓
npm ci
  ↓
npm run build
```

If the application cannot be installed or built, deployment stops.

---

## Phase 3 — Containerization

After the application build succeeds:

```text
Dockerfile
    +
Application source
    ↓
Docker build
    ↓
Docker image
```

The resulting image is tagged:

```text
adx6sh/adx6sh.sh:latest
```

---

## Phase 4 — Image Registry

The image is pushed to Docker Hub:

```text
GitHub Actions
      │
      │ docker push
      ▼
Docker Hub
```

Docker Hub becomes the image distribution point.

---

## Phase 5 — Remote Deployment

GitHub Actions connects to EC2:

```text
GitHub Actions
      │
      │ SSH
      ▼
AWS EC2
```

The EC2 server pulls the image:

```bash
docker pull adx6sh/adx6sh.sh:latest
```

---

## Phase 6 — Container Replacement

The old application container is replaced:

```text
Pull image
    ↓
Stop old container
    ↓
Remove old container
    ↓
Start new container
```

The new container runs:

```text
adx6sh-app
```

and maps:

```text
EC2:3000 → Container:3000
```

---

# 6. Repository Structure

Recommended structure:

```text
project/
├── .github/
│   └── workflows/
│       └── node-ci.yml
│
├── public/
│
├── src/
│
├── package.json
├── package-lock.json
├── Dockerfile
├── .dockerignore
└── README.md
```

The workflow must live inside:

```text
.github/workflows/
```

GitHub Actions workflows are defined as YAML files in this directory. citeturn0search4

---

# 7. Prerequisites

Before configuring the pipeline, you need:

## Local Project

- Node.js installed
- npm installed
- `package.json`
- `package-lock.json`
- A working `npm run build`
- A working Dockerfile

Test locally:

```bash
npm ci
npm run build
```

---

## GitHub

You need:

- A GitHub repository
- `main` branch
- GitHub Actions enabled

---

## Docker Hub

You need:

- Docker Hub account
- Docker Hub repository
- Docker Hub access token

---

## AWS

You need:

- AWS account
- EC2 instance
- Public IP or DNS
- SSH access
- Docker installed on EC2
- Correct security-group rules

---

# 8. Dockerfile

The GitHub Actions workflow expects a Dockerfile in the project root.

Example Node.js Dockerfile:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm ci

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

## Dockerfile Flow

```text
FROM
 ↓
WORKDIR
 ↓
COPY package files
 ↓
npm ci
 ↓
COPY application
 ↓
npm run build
 ↓
EXPOSE
 ↓
CMD
```

## Important

`EXPOSE 3000` does **not** publish port `3000` to the EC2 host.

It documents the port the application expects to use inside the container.

The actual host-to-container mapping is created later using:

```bash
-p 3000:3000
```

---

# 9. GitHub Actions Workflow

Create:

```text
.github/workflows/node-ci.yml
```

Use:

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

# 10. Workflow Line-by-Line

## `name`

```yaml
name: Node.js CI
```

Sets the workflow name displayed in GitHub Actions.

---

## `on`

```yaml
on:
```

Defines the events that trigger the workflow.

---

## Push

```yaml
  push:
```

Runs the workflow when code is pushed.

---

## Branch Filter

```yaml
    branches: ["main"]
```

Limits the push trigger to `main`.

```text
push → main
     ↓
workflow runs
```

---

## Pull Request

```yaml
  pull_request:
```

Triggers the workflow for Pull Request events.

---

```yaml
    branches: ["main"]
```

Only Pull Requests targeting `main` trigger this workflow.

---

## Jobs

```yaml
jobs:
```

Defines the jobs in the workflow.

This workflow contains:

```text
build
deploy
```

---

## Build Job

```yaml
  build:
```

Creates the `build` job.

---

## Runner

```yaml
    runs-on: ubuntu-latest
```

Runs the job on a GitHub-hosted Ubuntu runner.

This is a temporary runner and is separate from the EC2 server.

---

## Strategy

```yaml
    strategy:
```

Defines how the job should be executed.

---

## Matrix

```yaml
      matrix:
        node-version: [22.x]
```

Creates a matrix containing Node.js 22.x.

A matrix becomes more useful when multiple versions are required:

```yaml
node-version: [20.x, 22.x]
```

---

## Steps

```yaml
    steps:
```

Contains the operations executed by the job.

---

## Checkout

```yaml
      - uses: actions/checkout@v4
```

Checks the repository source code out onto the GitHub runner.

Without this step, the runner would not have your project files.

---

## Node.js Setup

```yaml
      - name: Use Node.js ${{ matrix.node-version }}
```

Creates a readable name for the Node.js setup step.

`${{ ... }}` is GitHub Actions expression syntax.

---

```yaml
        uses: actions/setup-node@v4
```

Installs/configures Node.js on the runner.

---

```yaml
        with:
```

Provides configuration to the action.

---

```yaml
          node-version: ${{ matrix.node-version }}
```

Uses the Node.js version defined by the matrix.

Currently:

```text
22.x
```

---

```yaml
          cache: "npm"
```

Enables npm dependency caching.

---

## Install Dependencies

```yaml
      - run: npm ci
```

Runs:

```bash
npm ci
```

This performs a clean dependency installation based on the lockfile.

---

## Build

```yaml
      - run: npm run build
```

Executes the project's `build` script.

Example:

```json
{
  "scripts": {
    "build": "next build"
  }
}
```

If the build fails, the job fails and deployment does not proceed.

---

# Docker Authentication

## Docker Login Step

```yaml
      - name: Login to Docker Hub
```

Names the Docker authentication step.

---

```yaml
        uses: docker/login-action@v4
```

Uses Docker's login action to authenticate with Docker Hub.

---

```yaml
        with:
```

Provides the login configuration.

---

```yaml
          username: ${{ secrets.DOCKERHUB_USERNAME }}
```

Reads the Docker Hub username from a GitHub Secret.

---

```yaml
          password: ${{ secrets.DOCKERHUB_TOKEN }}
```

Reads the Docker Hub access token from a GitHub Secret.

GitHub's `secrets` context provides secrets to workflows, and GitHub recommends using secrets for sensitive values rather than hard-coding them. citeturn0search1turn0search3

---

# QEMU

```yaml
      - name: Set up QEMU
```

Names the QEMU step.

---

```yaml
        uses: docker/setup-qemu-action@v4
```

Installs/configures QEMU support.

This is useful for architecture emulation and multi-platform image builds. Docker documents its QEMU action as part of its official GitHub Actions tooling. citeturn0search0

---

# Buildx

```yaml
      - name: Set up Docker Buildx
```

Names the Buildx step.

---

```yaml
        uses: docker/setup-buildx-action@v4
```

Creates/configures a BuildKit builder through Docker Buildx.

---

# Build and Push

```yaml
      - name: Build and push
```

Names the image build/publish step.

---

```yaml
        uses: docker/build-push-action@v7
```

Uses Docker's official build-and-push action.

Docker documents this action for building and pushing container images from GitHub Actions. citeturn0search0

---

## Push

```yaml
        with:
          push: true
```

Tells the action to push the image to the authenticated registry after building.

---

## Image Tag

```yaml
          tags: adx6sh/adx6sh.sh:latest
```

Defines the image reference:

```text
USERNAME/IMAGE:TAG
```

Therefore:

```text
adx6sh
```

is the Docker Hub username.

```text
adx6sh.sh
```

is the image/repository name.

```text
latest
```

is the tag.

Final image:

```text
adx6sh/adx6sh.sh:latest
```

---

# Deploy Job

```yaml
  deploy:
```

Creates the deployment job.

---

## Runner

```yaml
    runs-on: ubuntu-latest
```

The deployment job starts on another GitHub-hosted Ubuntu runner.

That runner will connect to EC2.

---

## Dependency

```yaml
    needs: build
```

Makes `deploy` depend on `build`.

Therefore:

```text
build
  │
  ├── success → deploy
  │
  └── failure → deployment skipped
```

This is a critical CI/CD control. citeturn0search2

---

# SSH Deployment

```yaml
      - name: Deploy to Instance
```

Names the deployment step.

---

```yaml
        uses: appleboy/ssh-action@v1.2.2
```

Uses an SSH action to execute remote commands on EC2.

---

## Host

```yaml
          host: ${{ secrets.EC2_HOST }}
```

Reads the EC2 public IP address or public DNS name from:

```text
EC2_HOST
```

---

## Username

```yaml
          username: ${{ secrets.EC2_USERNAME }}
```

Reads the EC2 Linux username from:

```text
EC2_USERNAME
```

---

## Private Key

```yaml
          key: ${{ secrets.EC2_SSH_KEY }}
```

Reads the SSH private key from:

```text
EC2_SSH_KEY
```

Never commit this private key to Git.

---

# Remote Script

```yaml
          script: |
```

Everything below this line is executed remotely on EC2.

---

## Pull Image

```bash
docker pull adx6sh/adx6sh.sh:latest
```

Downloads the latest image from Docker Hub to EC2.

---

## Stop Old Container

```bash
docker stop adx6sh-app || true
```

Attempts to stop the current application container.

The `|| true` prevents a missing container from causing the deployment to fail.

---

## Remove Old Container

```bash
docker rm adx6sh-app || true
```

Removes the previous container.

Again, `|| true` allows deployment to continue if the container does not exist.

---

## Run New Container

```bash
docker run -d \
```

Creates and starts a new container.

`-d` means detached mode.

---

## Container Name

```bash
  --name adx6sh-app \
```

Names the container:

```text
adx6sh-app
```

This makes later management easier:

```bash
docker logs adx6sh-app
docker stop adx6sh-app
docker restart adx6sh-app
```

---

## Port Mapping

```bash
  -p 3000:3000 \
```

Maps:

```text
HOST_PORT:CONTAINER_PORT
```

Therefore:

```text
EC2 port 3000
      │
      ▼
Container port 3000
```

---

## Restart Policy

```bash
  --restart unless-stopped \
```

Tells Docker to restart the container automatically after unexpected termination and after Docker/host restarts, unless the container has been intentionally stopped.

---

## Image

```bash
  adx6sh/adx6sh.sh:latest
```

Specifies the image from which the new container is created.

---

# 11. GitHub Secrets

Go to:

```text
GitHub Repository
        ↓
Settings
        ↓
Secrets and variables
        ↓
Actions
```

Create these repository secrets:

| Secret | Value |
|---|---|
| `DOCKERHUB_USERNAME` | Docker Hub username |
| `DOCKERHUB_TOKEN` | Docker Hub access token |
| `EC2_HOST` | EC2 public IP/DNS |
| `EC2_USERNAME` | EC2 SSH username |
| `EC2_SSH_KEY` | EC2 private SSH key |

---

## Why Secrets?

Never do this:

```yaml
username: myusername
password: mypassword
```

Instead:

```yaml
username: ${{ secrets.DOCKERHUB_USERNAME }}
password: ${{ secrets.DOCKERHUB_TOKEN }}
```

Secrets are designed for sensitive values. GitHub also automatically redacts secrets in workflow logs in supported cases. citeturn0search3

---

# 12. Docker Hub Setup

Create a Docker Hub repository corresponding to:

```text
adx6sh/adx6sh.sh
```

The workflow will publish:

```text
adx6sh/adx6sh.sh:latest
```

The image lifecycle is:

```text
Dockerfile
    ↓
docker/build-push-action
    ↓
Docker image
    ↓
Docker Hub
```

Docker's official GitHub Actions support authentication and image build/push workflows. citeturn0search0

---

# 13. AWS EC2 Setup

The EC2 server needs Docker.

After connecting to EC2, verify:

```bash
docker --version
```

Then:

```bash
docker ps
```

If Docker is working, the commands should return Docker information rather than a daemon error.

---

## Security Group

For this example, the EC2 security group needs to allow:

| Port | Protocol | Purpose |
|---|---|---|
| 22 | TCP | SSH |
| 3000 | TCP | Application |

For production, exposing application port `3000` directly to the internet is often not ideal.

A more typical architecture is:

```text
Internet
   ↓
80/443
   ↓
Nginx / Load Balancer
   ↓
Application :3000
```

---

# 14. SSH Authentication

The GitHub runner needs to authenticate to EC2.

The workflow provides:

```yaml
host: ${{ secrets.EC2_HOST }}
username: ${{ secrets.EC2_USERNAME }}
key: ${{ secrets.EC2_SSH_KEY }}
```

Conceptually:

```text
GitHub Actions Runner
        │
        │ Private key
        │
        │ SSH
        ▼
     AWS EC2
```

The private key should be stored only in the GitHub Secret.

---

# 15. Docker Port Mapping

This is one of the most important concepts in the pipeline.

The Dockerfile contains:

```dockerfile
EXPOSE 3000
```

This indicates that the application uses port `3000` inside the container.

The deployment command contains:

```bash
-p 3000:3000
```

The syntax is:

```text
-p HOST_PORT:CONTAINER_PORT
```

Therefore:

```text
-p 3000:3000
```

means:

```text
EC2
Port 3000
   │
   ▼
Docker
Port 3000
   │
   ▼
Application
```

You could use a different host port:

```bash
-p 80:3000
```

Then:

```text
EC2 port 80
     ↓
Container port 3000
```

The application still listens on `3000` inside the container.

---

# 16. Deployment Lifecycle

## Before Deployment

```text
EC2
 └── adx6sh-app
       └── old image
```

---

## Step 1 — Pull

```bash
docker pull adx6sh/adx6sh.sh:latest
```

```text
Docker Hub
    ↓
EC2
```

---

## Step 2 — Stop

```bash
docker stop adx6sh-app
```

```text
Running container
       ↓
Stopped container
```

---

## Step 3 — Remove

```bash
docker rm adx6sh-app
```

```text
Stopped container
       ↓
Removed
```

---

## Step 4 — Run

```bash
docker run ...
```

```text
Docker image
     ↓
New container
     ↓
Running application
```

---

# 17. First Deployment

On the first deployment, the container may not exist.

The workflow handles this using:

```bash
docker stop adx6sh-app || true
docker rm adx6sh-app || true
```

The first command may fail because there is no container.

`|| true` allows the script to continue.

Then:

```bash
docker run -d ...
```

creates the first container.

---

# 18. Subsequent Deployments

Suppose version 1 is currently running:

```text
Docker Hub
    │
    ▼
Image v1
    │
    ▼
adx6sh-app
```

You make a change and push:

```bash
git push origin main
```

The pipeline runs again:

```text
New source code
      ↓
Build
      ↓
New Docker image
      ↓
Docker Hub
      ↓
EC2
      ↓
Pull latest image
      ↓
Stop old container
      ↓
Remove old container
      ↓
Start new container
```

The result:

```text
Image v2
   ↓
adx6sh-app
```

---

# 19. CI vs CD

## Continuous Integration — CI

CI is responsible for verifying that the application can be built.

In this pipeline:

```text
Checkout
   ↓
Node.js setup
   ↓
npm ci
   ↓
npm run build
```

This is the integration/build portion.

---

## Continuous Deployment — CD

CD is responsible for delivering the application to the server.

In this pipeline:

```text
Docker build
   ↓
Docker push
   ↓
SSH to EC2
   ↓
Docker pull
   ↓
Replace container
```

---

## Complete Pipeline

```text
                 CI
                  │
Code ──→ Install ──→ Build
                         │
                         ▼
                     Dockerize
                         │
                         ▼
                 ─────── CD ───────
                         │
                         ▼
                    Docker Hub
                         │
                         ▼
                       EC2
                         │
                         ▼
                     Container
```

---

# 20. Failure Behavior

A good CI/CD pipeline should stop when critical steps fail.

## Scenario 1 — Dependency Installation Fails

```text
npm ci
  ↓
FAIL
  ↓
Build job fails
  ↓
Deploy skipped
```

---

## Scenario 2 — Application Build Fails

```text
npm run build
  ↓
FAIL
  ↓
Build job fails
  ↓
Deploy skipped
```

---

## Scenario 3 — Docker Push Fails

```text
Docker build
    ↓
Success
    ↓
Docker push
    ↓
FAIL
    ↓
Build job fails
    ↓
Deploy skipped
```

---

## Scenario 4 — EC2 Deployment Fails

```text
Build
  ↓
Success
  ↓
Deploy
  ↓
SSH/Docker command fails
  ↓
Deployment job fails
```

At this point the image may already exist in Docker Hub, but EC2 did not successfully complete the deployment.

---

# 21. Security

Security is critical because the workflow handles:

- Docker credentials
- SSH credentials
- EC2 access
- Production deployment

## Never Commit Secrets

Do not put:

```text
password
token
private key
AWS credentials
```

directly in the repository.

Use GitHub Secrets.

---

## Protect the Main Branch

Consider requiring:

- Pull Requests
- Reviews
- Successful checks
- No direct pushes

before merging into `main`.

---

## Use Least Privilege

The deployment credentials should have only the permissions they need.

---

## Protect SSH

Restrict SSH access where practical.

Avoid exposing port `22` to the entire internet unless required.

---

## Consider GitHub Environments

For a production deployment, consider:

```text
GitHub
  ↓
Actions
  ↓
production environment
  ↓
approval
  ↓
EC2
```

This can provide an additional deployment-control layer.

---

# 22. Troubleshooting

## Workflow Does Not Trigger

Check:

```yaml
on:
  push:
    branches: ["main"]
```

Make sure you are actually pushing to:

```text
main
```

Check:

```bash
git branch
```

and:

```bash
git push origin main
```

---

## `npm ci` Fails

Check:

```text
package.json
package-lock.json
```

Make sure the lockfile is committed.

Run locally:

```bash
npm ci
```

---

## `npm run build` Fails

Run locally:

```bash
npm run build
```

Fix the application/build error before debugging Docker.

---

## Docker Login Fails

Check:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
```

Make sure the token is valid and the Docker Hub repository is accessible.

---

## Docker Push Fails

Check the image tag:

```text
adx6sh/adx6sh.sh:latest
```

Make sure:

- Docker Hub repository exists.
- Username is correct.
- Token has the required access.
- The repository is writable.

---

## SSH Fails

Check:

```text
EC2_HOST
EC2_USERNAME
EC2_SSH_KEY
```

Also verify:

- EC2 instance is running.
- Public IP/DNS is correct.
- Security group allows SSH.
- Correct Linux username is used.
- The private key corresponds to the EC2 instance's configured SSH access.

---

## Docker Not Found on EC2

Run:

```bash
docker --version
```

If Docker is unavailable, install and configure Docker on the EC2 instance before running the deployment.

---

## Container Is Not Running

Run:

```bash
docker ps
```

Then:

```bash
docker ps -a
```

Inspect logs:

```bash
docker logs adx6sh-app
```

---

## Application Not Reachable

Check the container:

```bash
docker ps
```

Check port mapping:

```bash
docker port adx6sh-app
```

Expected:

```text
3000/tcp -> 0.0.0.0:3000
```

Also check:

- EC2 security group
- Application listening address
- Docker port mapping
- EC2 public IP
- Application logs

---

# 23. Useful Commands

## Git

```bash
git status
git add .
git commit -m "deploy update"
git push origin main
```

---

## Docker Images

List images:

```bash
docker images
```

Pull image:

```bash
docker pull adx6sh/adx6sh.sh:latest
```

Remove image:

```bash
docker rmi IMAGE_ID
```

---

## Docker Containers

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

Start:

```bash
docker start adx6sh-app
```

Stop:

```bash
docker stop adx6sh-app
```

Restart:

```bash
docker restart adx6sh-app
```

Remove:

```bash
docker rm adx6sh-app
```

---

## Logs

Follow application logs:

```bash
docker logs -f adx6sh-app
```

Show recent logs:

```bash
docker logs --tail 100 adx6sh-app
```

---

## Inspect Container

```bash
docker inspect adx6sh-app
```

---

## Check Port

```bash
docker port adx6sh-app
```

---

# 24. Production Improvements

This workflow is a strong learning and small-project deployment pipeline, but production systems can be improved considerably.

## 1. Stop Using `latest` as the Only Deployment Identifier

Current:

```text
adx6sh/adx6sh.sh:latest
```

A better production strategy is to also create immutable tags:

```text
adx6sh/adx6sh.sh:abc1234
```

or:

```text
adx6sh/adx6sh.sh:v1.4.0
```

This makes rollback much easier.

---

## 2. Add Tests

Before building:

```yaml
- run: npm test
```

Recommended flow:

```text
npm ci
   ↓
npm test
   ↓
npm run build
   ↓
Docker build
```

---

## 3. Add Docker Build Cache

Buildx supports caching strategies that can reduce build time.

Docker documents caching and other advanced BuildKit capabilities for GitHub Actions. citeturn0search0

---

## 4. Add Image Metadata

A production workflow can use Docker metadata tooling to generate tags and labels based on:

- Git commit
- Branch
- Git tag
- Pull Request
- Release

---

## 5. Add Image Security Scanning

Consider scanning images for known vulnerabilities before deployment.

---

## 6. Add SBOM and Provenance

Docker's build tooling supports SBOM and provenance attestations. These provide additional information about what went into an image and how it was built. citeturn0search17

---

## 7. Use a Reverse Proxy

Instead of:

```text
Internet
   ↓
EC2:3000
```

a more production-oriented architecture is:

```text
Internet
   ↓
HTTPS :443
   ↓
Nginx / Load Balancer
   ↓
Docker container :3000
```

---

## 8. Add HTTPS

Use TLS/HTTPS rather than exposing an application directly over plain HTTP.

---

## 9. Add Health Checks

A production deployment should verify that the new application is actually healthy.

Example concept:

```text
Deploy
  ↓
Start container
  ↓
Health check
  ↓
Healthy?
 ┌──────┴──────┐
Yes           No
 │             │
 ▼             ▼
Success      Rollback
```

---

## 10. Add Rollback

A production pipeline should have a clear rollback mechanism.

For example:

```text
v1.2.0
  ↓
Deploy
  ↓
Problem detected
  ↓
docker pull image:v1.1.0
  ↓
Restart
```

Immutable image tags make this much easier.

---

## 11. Pin Actions

For stronger supply-chain security, consider pinning actions to specific commit SHAs rather than relying only on mutable version tags.

---

# 25. Mental Model

The easiest way to understand this entire project is to remember five components.

## 1. GitHub

Stores the source code.

```text
Code
```

---

## 2. GitHub Actions

Automates the process.

```text
Automation
```

---

## 3. Docker

Packages the application.

```text
Application → Image
```

---

## 4. Docker Hub

Stores and distributes the image.

```text
Image Registry
```

---

## 5. EC2

Runs the application.

```text
Server
```

Together:

```text
GitHub
   │
   ▼
GitHub Actions
   │
   ▼
Docker Image
   │
   ▼
Docker Hub
   │
   ▼
AWS EC2
   │
   ▼
Docker Container
   │
   ▼
Application
```

---

# 26. Final Checklist

Before running the pipeline, verify every item.

## Application

- [ ] `package.json` exists
- [ ] `package-lock.json` exists
- [ ] `npm ci` works
- [ ] `npm run build` works
- [ ] Application listens on port `3000`

## Docker

- [ ] `Dockerfile` exists
- [ ] Docker image builds locally
- [ ] Docker Hub repository exists
- [ ] Image name is correct

Test locally:

```bash
docker build -t adx6sh/adx6sh.sh:latest .
```

Run locally:

```bash
docker run -d \
  --name adx6sh-app \
  -p 3000:3000 \
  adx6sh/adx6sh.sh:latest
```

---

## GitHub

- [ ] Workflow is inside `.github/workflows/`
- [ ] Workflow YAML is valid
- [ ] `main` branch exists
- [ ] GitHub Actions is enabled

---

## GitHub Secrets

- [ ] `DOCKERHUB_USERNAME`
- [ ] `DOCKERHUB_TOKEN`
- [ ] `EC2_HOST`
- [ ] `EC2_USERNAME`
- [ ] `EC2_SSH_KEY`

---

## EC2

- [ ] EC2 instance is running
- [ ] Docker is installed
- [ ] SSH access works
- [ ] Port 22 is reachable from the required source
- [ ] Application port is allowed
- [ ] Correct EC2 username is used
- [ ] Correct SSH key is configured

---

# 27. References

- GitHub Actions workflows: https://docs.github.com/en/actions/concepts/workflows-and-actions/workflows
- GitHub Actions jobs: https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-jobs
- GitHub Actions contexts and secrets: https://docs.github.com/en/actions/reference/workflows-and-actions/contexts
- GitHub Actions secrets: https://docs.github.com/en/actions/reference/security/secrets
- Docker GitHub Actions: https://docs.docker.com/build/ci/github-actions/
- Docker GitHub Actions guide: https://docs.docker.com/guides/gha/

---

# Final Architecture

```text
                         ┌────────────────────┐
                         │     DEVELOPER      │
                         │                    │
                         │  git push origin   │
                         │       main         │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │       GITHUB       │
                         │                    │
                         │    Repository      │
                         └─────────┬──────────┘
                                   │
                                   ▼
                    ┌────────────────────────────┐
                    │      GITHUB ACTIONS        │
                    │                            │
                    │  Checkout                  │
                    │      ↓                     │
                    │  Node.js 22                │
                    │      ↓                     │
                    │  npm ci                    │
                    │      ↓                     │
                    │  npm run build             │
                    │      ↓                     │
                    │  Docker build              │
                    │      ↓                     │
                    │  Docker push               │
                    └────────────┬───────────────┘
                                 │
                                 ▼
                    ┌────────────────────────────┐
                    │        DOCKER HUB           │
                    │                            │
                    │ adx6sh/adx6sh.sh:latest   │
                    └────────────┬───────────────┘
                                 │
                                 │ docker pull
                                 ▼
                    ┌────────────────────────────┐
                    │          AWS EC2            │
                    │                            │
                    │       Docker Engine        │
                    │             │              │
                    │             ▼              │
                    │       adx6sh-app            │
                    │             │              │
                    │      3000 → 3000           │
                    └─────────────┬──────────────┘
                                  │
                                  ▼
                           WEB APPLICATION
```

## The One-Line Summary

```text
git push → GitHub Actions → build → Docker image → Docker Hub → SSH → EC2 → new container → application
```

This is the core CI/CD pattern implemented by this project.
