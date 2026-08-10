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


## Step 3: build and run container

```bash
# Build the image
docker build -t runner .

# Run the container (with Docker socket mount for Docker executor)
docker run -d \
  --name gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /etc/gitlab-runner/config.toml:/etc/gitlab-runner/config.toml \
  runner

# Or for shell executor (simpler):
docker run -d --name gitlab-runner runner
```

---

## Step 4: Register Runners with GitLab

### 4.1 Get GitLab Registration Token

1. Open GitLab: `http://localhost:8082`
2. Go to **Admin** → **Runners**
3. Copy the registration token (starts with `GR134894...`)

### 4.2 Register Oracle Runner

```bash
# Enter container
docker exec -it gitlab-runner bash

# Register the runner
gitlab-runner register \
  --url http://host.docker.internal:8082 \
  --registration-token YOUR_TOKEN \
  --executor docker \
  --docker-image alpine:latest \
  --description "Oracle Linux Runner" \
  --tag-list "oracle,linux"

# Exit container
exit
```

> **Note:** For `--executor`, choose:
 - `shell` - runs directly on container (no Docker socket needed)
 - `docker` - needs `/var/run/docker.sock` mounted

**Alternative: Interactive Registration**

```bash
# Enter container
docker exec -it gitlab-runner bash

# Register interactively
gitlab-runner register
# Follow prompts
```

---

## Step 5: Verify Runners

```bash
# List all runners
gitlab-runner list

# Check runner status
gitlab-runner verify
```

---

## Step 6: Test with a Pipeline

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
# Build image
docker build -t runner .

# Start container
docker run -d --name gitlab-runner runner

# Enter container
docker exec -it gitlab-runner bash

# Register runner
gitlab-runner register --url URL --registration-token TOKEN --executor shell

# List runners
gitlab-runner list

# Verify runners
gitlab-runner verify

# Check logs
docker logs gitlab-runner

# Stop and remove
docker stop gitlab-runner
docker rm gitlab-runner
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
- Check logs : `docker logs gitlab-runner`
---
