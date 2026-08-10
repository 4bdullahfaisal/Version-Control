# GitLab Runners Setup — Oracle

---

## What is a GitLab Runner?

A GitLab Runner is a lightweight agent that executes CI/CD jobs defined in `.gitlab-ci.yml`. It picks up jobs from GitLab, runs them in an isolated environment, and sends results back.

### Runner Types

| Type | Description |
|------|-------------|
| **Shell** | Runs jobs directly on the runner's shell |
| **Docker** | Runs jobs inside Docker containers |
| **SSH** | Runs jobs on a remote server via SSH |
| **Kubernetes** | Runs jobs in a Kubernetes cluster |

---

## Step 1: Folder Structure

```
runner/
├── dockerfile
```

---

## Step 2: Dockerfile

### `runner/dockerfile`

```dockerfile
FROM oraclelinux:9

RUN dnf install -y curl git openssh-clients tar which shadow-utils && \
    curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | bash && \
    dnf install -y gitlab-runner && \
    dnf clean all

WORKDIR /home/gitlab-runner

ENTRYPOINT ["gitlab-runner"]
CMD ["run", "--working-directory=/home/gitlab-runner", "--config=/etc/gitlab-runner/config.toml"]
```


## Step 3: Start Containers

```bash
# Start Oracle runner
cd runner
docker compose up -d

# Verify running
docker ps
```

---

## Step 4: Install GitLab Runner on Oracle Linux

```bash
# Enter container
docker exec -it gitlab-runner bash

# Update and install dependencies
yum update -y
yum install -y curl wget git

# Download GitLab Runner
curl -LJO "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64"

# Make executable and move to PATH
chmod +x gitlab-runner-linux-amd64
mv gitlab-runner-linux-amd64 /usr/local/bin/gitlab-runner

# Create gitlab-runner user
useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash

# Install and start as service
gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
gitlab-runner start

# Verify
gitlab-runner --version
```

---

## Step 6: Register Runners with GitLab

### 6.1 Get GitLab Registration Token

1. Open GitLab: `http://localhost:8082`
2. Go to **Admin** → **Runners**
3. Copy the registration token (starts with `GR134894...`)

### 6.2 Register Oracle Runner

```bash
docker exec -it gitlab-runner bash

gitlab-runner register \
  --url http://host.docker.internal:8082 \
  --registration-token YOUR_TOKEN \
  --executor shell \
  --description "Oracle Linux Runner" \
  --tag-list "oracle,linux"
```

---

## Step 7: Verify Runners

```bash
# List all runners
gitlab-runner list

# Check runner status
gitlab-runner verify
```

---

## Step 8: Test with a Pipeline

### Create `.gitlab-ci.yml` in your GitLab project:

```yaml
stages:
  - test

oracle-job:
  stage: test
  tags:
    - oracle
  script:
    - echo "Running on Oracle Linux"
    - cat /etc/os-release
    - whoami
    - pwd

```

### Push the file and check pipeline status in GitLab.

---

## Commands Summary

```bash
# Start containers
docker compose up -d

# Stop containers
docker compose down

# Check logs
docker compose logs -f

# Enter Oracle container
docker exec -it gitlab-runner bash

# Register runner
gitlab-runner register --url URL --registration-token TOKEN --executor shell

# List runners
gitlab-runner list

# Verify runners
gitlab-runner verify
```

---

## Troubleshooting

### Runner not showing up in GitLab UI
- Make sure GitLab is running: `http://localhost:8082`
- Token must match exactly
- Restart runner: `gitlab-runner restart`

### Runner registration fails
- Use `host.docker.internal` for Docker-to-host communication
- Or use your machine's local IP address

### Container not starting
- Check dockerfile syntax: `docker compose config`
---
