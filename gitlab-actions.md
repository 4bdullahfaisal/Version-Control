# GitHub Actions — CI/CD Basics

---

## What is GitHub Actions?

GitHub Actions is a CI/CD (Continuous Integration / Continuous Deployment) tool built into GitHub.

**Analogy:** GitHub Actions is like a robot that automatically runs tasks when you push code.

| Concept | Meaning |
|---------|---------|
| CI | Continuous Integration — automatically build and test code on every push |
| CD | Continuous Deployment — automatically deploy code after tests pass |
| Workflow | The automated process (defined in a YAML file) |
| Runner | The machine that executes the workflow |

---

## GitHub Actions Core Concepts

| Term | What it means |
|------|---------------|
| **Workflow** | A YAML file that defines the automation process |
| **Event** | What triggers the workflow (push, pull_request, schedule) |
| **Job** | A set of steps that run on the same runner |
| **Step** | A single task (run a command or use an action) |
| **Action** | A reusable piece of code (like a plugin) |
| **Runner** | The server that runs the workflow (GitHub-hosted or self-hosted) |

---

## Workflow File Location

```
.github/workflows/ci.yml
```

---

## Basic Workflow Example

### File: `.github/workflows/ci.yml`

```yaml
name: CI Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run a simple command
        run: echo "Hello from GitHub Actions!"

      - name: List files
        run: ls -la
```

---

## Workflow Breakdown

| Line | Meaning |
|------|---------|
| `name:` | Name of the workflow |
| `on:` | Trigger event (push, pull_request, schedule) |
| `jobs:` | Group of jobs to run |
| `runs-on:` | Runner type (ubuntu-latest, windows-latest, macos-latest) |
| `steps:` | List of tasks to execute |
| `uses:` | Use a pre-built action (like `actions/checkout`) |
| `run:` | Run a shell command |

---

## Common GitHub Actions Events

```yaml
# Trigger on push to main branch
on:
  push:
    branches: [ main ]

# Trigger on pull request to main
on:
  pull_request:
    branches: [ main ]

# Trigger on schedule (cron)
on:
  schedule:
    - cron: '0 0 * * *'   # Daily at midnight

# Trigger on tag creation
on:
  push:
    tags:
      - 'v*'

# Trigger manually from GitHub UI
on:
  workflow_dispatch:
```

---

## Docker Workflow Example

### Build and push Docker image to Docker Hub

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker image
        run: docker build -t my-app .

      - name: Tag image
        run: docker tag my-app username/my-app:latest

      - name: Push image
        run: docker push username/my-app:latest
```

---

## Deploy to AWS EC2 Example

```yaml
name: Deploy to EC2

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /app
            git pull
            docker compose down
            docker compose up -d --build
```

---

## Using Secrets in GitHub Actions

Secrets store sensitive data (passwords, API keys, SSH keys).

### Add a Secret:
1. Go to your GitHub repo
2. Settings → Secrets and variables → Actions
3. Click **New repository secret**
4. Add secret name and value

### Use Secrets in Workflow:
```yaml
${{ secrets.SECRET_NAME }}
```

---

## Matrix Builds (Test on Multiple Versions)

```yaml
name: Test on Multiple Versions

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18, 20]

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Run tests
        run: |
          npm install
          npm test
```

---

## Common GitHub Actions (Useful for DevOps)

| Action | Purpose |
|--------|---------|
| `actions/checkout` | Clone the repository |
| `actions/setup-node` | Install Node.js |
| `actions/setup-python` | Install Python |
| `docker/login-action` | Log in to Docker Hub / ECR |
| `docker/build-push-action` | Build and push Docker image |
| `appleboy/ssh-action` | Run commands on remote server (SSH) |
| `actions/upload-artifact` | Upload files for later use |
| `actions/download-artifact` | Download uploaded files |

---

## GitHub Actions vs Jenkins

| Feature | GitHub Actions | Jenkins |
|---------|----------------|---------|
| Hosting | GitHub-hosted (free for public) | Self-hosted |
| Setup | Zero setup | Needs installation |
| YAML vs UI | YAML-based | UI + Groovy |
| Integration | Native with GitHub | Plugin-based |

---
