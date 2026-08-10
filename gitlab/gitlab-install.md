# GitLab — Installation on Windows (Docker)

---

## What is GitLab?

GitLab is a complete DevOps platform that provides source code management (Git), CI/CD, issue tracking, and more in a single application. It is self-hosted, giving you full control over your development infrastructure.


| Feature | Description |
|---------|--------------|
| Source Code Management | Git repositories |
| CI/CD | Built-in pipelines |
| Container Registry | Store Docker images |
| Issue Tracking | Project management |
| Self-Hosted | Run on your own infrastructure |
| Security | Built-in vulnerability scanning |

---

## Installation on Windows Using Docker

### Prerequisites
- Docker Desktop installed and running
- Windows 10/11
---

## Folder Structure

```bash
gitlab/
│
├── docker-compose.yml
│
├── runner/
│   └── dockerfile
```

## Step 1: Create Data Directory

```bash
mkdir gitlab

cd gitlab
```

---

## Step 2: Docker Compose File

### `gitlab/docker-compose.yml`

```yaml
version: "3.8"

services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    hostname: gitlab
    restart: always
    ports:
      - "8929:8929"
      - "443:443"
      - "22:22"
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.local:8929'
        gitlab_rails['gitlab_shell_ssh_port'] = 22
    volumes:
      - gitlab_config:/etc/gitlab
      - gitlab_logs:/var/log/gitlab
      - gitlab_data:/var/opt/gitlab
    networks:
      gitlab-net:
        ipv4_address: 172.28.0.10
        aliases:
          - gitlab.local

networks:
  gitlab-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16

volumes:
  gitlab_config:
  gitlab_logs:
  gitlab_data:
```

## Port Mapping Explanation

| Port | Purpose |
|------|---------|
| `-p 8929:8929` | Web UI (http://gitlab.local:8929) |
| `-p 443:443` | HTTPS (SSL) |
| `-p 22:22` | SSH for Git operations |

---

## Step 3: Update Windows Hosts File

Add this entry to `C:\Windows\System32\drivers\etc\hosts`:

```
127.0.0.1 gitlab.local
```

**How to edit:**
1. Open Notepad as Administrator
2. File → Open → `C:\Windows\System32\drivers\etc\hosts`
3. Add the line above
4. Save and close

---

## Step 4: Start GitLab

```bash
# Start GitLab container
docker compose up -d

# Check if container is running
docker compose ps

# View logs (wait 2-5 minutes for GitLab to fully start)
docker compose logs -f gitlab

# First start takes 3–5 minutes.

# Wait until you see logs with reconfigure complete.
```
---

## Step 5: Get Root Password

```bash
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

Copy the password shown.

---

## Step 6: Access GitLab Web UI

Open browser: `http://gitlab.local:8929/`

Login:
- **Username:** `root`
- **Password:** (from step 4)

---

## Step 7: Change Password (First Time)

After login, set a new password for `root`.

---

## Common Commands

```bash
# Start containers
docker compose up -d

# Stop containers
docker compose down

# Stop and remove volumes (clean start)
docker compose down -v

# Check if GitLab is running
docker compose ps

# View GitLab logs
docker compose logs -f gitlab

# Get root password again
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password

# Enter runner container
docker exec -it runner bash

# Restart runner
docker restart runner
```

---

## Troubleshooting

### gitlab.local doesn't resolve
- Add `127.0.0.1 gitlab.local` to `C:\Windows\System32\drivers\etc\hosts`
- Restart browser or flush DNS: `ipconfig /flushdns`

### Docker not running
- Open Docker Desktop manually
- Wait for whale icon to show "running"

### Port already in use
Change host port:
```yaml
ports:
  - "8083:8929"  # Access at http://gitlab.local:8083
```

### Container fails to start
```bash
docker logs gitlab
```

### Runner can't connect to GitLab
- Check network: `docker network ls`
- Verify runner is on same network: `docker inspect runner`
- Use `http://gitlab.local:8929` not `localhost`

### Clean reinstall
```bash
docker compose down -v
docker system prune -f
# Then start fresh
```

---

## GitLab Runner

```bash
├── runner/
│   └── dockerfile
```

## Dockerfile

```
FROM oraclelinux:9

RUN dnf install -y curl git openssh-clients tar which shadow-utils && \
    curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | bash && \
    dnf install -y gitlab-runner && \
    dnf clean all

WORKDIR /home/gitlab-runner

ENTRYPOINT ["gitlab-runner"]
CMD ["run", "--working-directory=/home/gitlab-runner", "--config=/etc/gitlab-runner/config.toml"]
```

To run CI/CD pipelines, install GitLab Runner:

```bash
# Build image
docker build -t runner ./runner

# Run it 
docker run -d --name gitlab-runner \
  --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /c/gitlab-runner/config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest
```

Register runner using token from GitLab UI (Projects → Settings → CI/CD → Runners).

---
