# GitLab — Installation on Windows (Docker)
---

## What is GitLab?

GitLab is a complete DevOps platform with:

| Feature | What it does |
|---------|--------------|
| Source Code Management | Git repositories |
| CI/CD | Built-in pipelines |
| Container Registry | Store Docker images |
| Issue Tracking | Project management |
| Self-Hosted | Run on your own infrastructure |

---

## Installation on Windows Using Docker

### Prerequisites
- Docker Desktop installed and running
- Windows 10/11
---

## Step 1: Create Data Directory

```bash
mkdir -p /c/gitlab
```

---

## Step 2: Run GitLab Container

```bash
docker run -d --name gitlab \
  --restart always \
  -p 8082:80 \
  -p 8443:443 \
  -p 8022:22 \
  -v /c/gitlab/config:/etc/gitlab \
  -v /c/gitlab/logs:/var/log/gitlab \
  -v /c/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

---

## Port Mapping Explanation

| Port | Purpose |
|------|---------|
| `-p 8082:80` | Web UI (http://localhost:8082) |
| `-p 8443:443` | HTTPS (SSL) |
| `-p 8022:22` | SSH for Git operations |

---

## Step 3: Wait for Startup

First start takes 3–5 minutes.

```bash
docker logs gitlab -f
```

Wait until you see logs with reconfigure complete.

---

## Step 4: Get Root Password

```bash
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

Copy the password shown.

---

## Step 5: Access GitLab Web UI

Open browser: `http://localhost:8082`

Login:
- **Username:** `root`
- **Password:** (from step 4)

---

## Step 6: Change Password (First Time)

After login, set a new password for `root`.

---

## Common Commands

```bash
# Check if GitLab is running
docker ps

# View logs
docker logs gitlab -f

# Stop GitLab
docker stop gitlab

# Start GitLab
docker start gitlab

# Restart GitLab
docker restart gitlab

# Get root password again
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password

# Update GitLab to latest
docker pull gitlab/gitlab-ce:latest
docker stop gitlab
docker rm gitlab
# Then re-run the docker run command above
```

---

## Troubleshooting

### Docker not running
- Open Docker Desktop manually
- Wait for whale icon to show "running"

### Port already in use
Change host port:
```bash
-p 8083:80
```
Then access at `http://localhost:8083`

### Container fails to start
```bash
docker logs gitlab
```

### Clean reinstall
```bash
docker stop gitlab
docker rm gitlab
docker volume prune -f
# Then re-run docker run command
```

---

## GitLab Runner (Optional)

To run CI/CD pipelines, install GitLab Runner:

```bash
docker run -d --name gitlab-runner \
  --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /c/gitlab-runner/config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest
```

Register runner using token from GitLab UI (Admin → Runners).

---
