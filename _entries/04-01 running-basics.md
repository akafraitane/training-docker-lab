---
sectionid: runbasics
sectionclass: h2
title: Container Lifecycle Management
parent-id: running
---

### Basic Container Operations

**Run a container:**

```bash
# Simple run
podman run nginx

# Run in background (-d = detached)
podman run -d nginx

# Run with name
podman run -d --name my-nginx nginx

# Run interactively (-it = interactive + TTY)
podman run -it alpine sh

# Run and remove after exit
podman run --rm alpine echo "Hello World"
```

**List containers:**

```bash
# List running containers
podman ps

# List all containers (including stopped)
podman ps -a

# List with custom format
podman ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

**Stop containers:**

```bash
# Stop gracefully (sends SIGTERM, then SIGKILL after 10s)
podman stop my-nginx

# Stop immediately (sends SIGKILL)
podman kill my-nginx

# Stop all running containers
podman stop $(podman ps -q)
```

**Start/Restart containers:**

```bash
# Start a stopped container
podman start my-nginx

# Restart a container
podman restart my-nginx

# Start and attach to container
podman start -a my-nginx
```

**Remove containers:**

```bash
# Remove stopped container
podman rm my-nginx

# Force remove (even if running)
podman rm -f my-nginx

# Remove all stopped containers
podman container prune

# Remove all containers (including running)
podman rm -f $(podman ps -aq)
```

---

### Production Container Configuration

**Complete production run command:**

```bash
podman run -d \
  --name myapp \
  --hostname myapp-prod \
  --memory=512m \
  --memory-reservation=256m \
  --cpus=1.5 \
  --pids-limit=100 \
  --network mynetwork \
  --ip 10.88.0.10 \
  -p 8080:8000 \
  -e FLASK_ENV=production \
  -e DATABASE_URL=postgresql://db/myapp \
  --env-file .env.prod \
  -v $(pwd)/data:/app/data:ro \
  -v myapp-logs:/var/log \
  --user 1000:1000 \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --security-opt=no-new-privileges \
  --read-only \
  --tmpfs /tmp \
  --restart=unless-stopped \
  --health-cmd='curl -f http://localhost:8000/health || exit 1' \
  --health-interval=30s \
  --health-timeout=3s \
  --health-retries=3 \
  --health-start-period=10s \
  myapp:v1.2.0
```

**Let's break this down:**

**Resource Limits:**
```bash
--memory=512m              # Hard memory limit
--memory-reservation=256m  # Soft limit (prefer to stay under)
--cpus=1.5                # 1.5 CPU cores max
--pids-limit=100          # Max 100 processes
```

**Networking:**
```bash
--network mynetwork       # Connect to custom network
--hostname myapp-prod     # Set container hostname
--ip 10.88.0.10          # Static IP in network
-p 8080:8000             # Port mapping (host:container)
```

**Environment Variables:**
```bash
-e FLASK_ENV=production           # Individual var
-e DATABASE_URL=postgresql://...  # Database connection
--env-file .env.prod             # Load from file
```

**Storage:**
```bash
-v $(pwd)/data:/app/data:ro  # Read-only bind mount
-v myapp-logs:/var/log       # Named volume
--tmpfs /tmp                  # In-memory temporary filesystem
--read-only                   # Root filesystem is read-only
```

**Security:**
```bash
--user 1000:1000                     # Run as non-root user
--cap-drop=ALL                       # Drop all Linux capabilities
--cap-add=NET_BIND_SERVICE           # Add only needed capability
--security-opt=no-new-privileges     # Prevent privilege escalation
```

**Reliability:**
```bash
--restart=unless-stopped  # Auto-restart policy
--health-cmd='...'       # Health check command
--health-interval=30s    # Check every 30 seconds
--health-retries=3       # Fail after 3 failed checks
```

---

### Restart Policies

```bash
# Never restart (default)
podman run --restart=no nginx

# Always restart (even after reboot)
podman run --restart=always nginx

# Restart only on failure
podman run --restart=on-failure nginx

# Restart unless manually stopped
podman run --restart=unless-stopped nginx

# Restart max 5 times on failure
podman run --restart=on-failure:5 nginx
```

---

### Inspecting Running Containers

**View logs:**

```bash
# View logs
podman logs myapp

# Follow logs (like tail -f)
podman logs -f myapp

# Last 100 lines
podman logs --tail=100 myapp

# Logs since timestamp
podman logs --since="2024-02-13T10:00:00" myapp

# Logs with timestamps
podman logs -t myapp
```

**Exec into running container:**

```bash
# Run shell in running container
podman exec -it myapp sh

# Run specific command
podman exec myapp ls -la /app

# Run as different user
podman exec --user root myapp apt-get update
```

**Inspect container details:**

```bash
# Full JSON output
podman inspect myapp

# Specific field
podman inspect myapp --format='{{.State.Status}}'
podman inspect myapp --format='{{.NetworkSettings.IPAddress}}'
podman inspect myapp --format='{{range .Config.Env}}{{println .}}{{end}}'
```

**Resource usage:**

```bash
# Real-time stats
podman stats

# Stats for specific container
podman stats myapp

# One-time stats (no stream)
podman stats --no-stream
```

**Process information:**

```bash
# Processes in container
podman top myapp

# With full command
podman top myapp aux
```

**Port mappings:**

```bash
# Show port mappings
podman port myapp

# Specific port
podman port myapp 8000
```

---

### Pause and Unpause

Freezes all processes in container (useful for debugging):

```bash
# Pause container
podman pause myapp

# Container is now frozen (but still running)
podman ps

# Unpause
podman unpause myapp
```

---

### Copy Files

```bash
# Copy from container to host
podman cp myapp:/app/logs/app.log ./local-logs/

# Copy from host to container
podman cp ./config.yml myapp:/app/config/

# Copy entire directory
podman cp myapp:/app/data ./backup/
```

---

### Exercise: Container Lifecycle

{% collapsible %}

**Task 1: Run Container with Resource Limits**

```bash
# Run nginx with limits
podman run -d \
  --name web \
  --memory=256m \
  --cpus=0.5 \
  -p 8080:80 \
  nginx:alpine

# Check it's running
podman ps

# View resource usage
podman stats --no-stream web

# Test it works
curl localhost:8080

# View logs
podman logs web

# Exec into container
podman exec -it web sh
# Inside container:
ps aux
top
exit

# Stop and remove
podman stop web
podman rm web
```

**Task 2: Environment Variables**

Create `.env` file:
```
DATABASE_HOST=db.example.com
DATABASE_PORT=5432
API_KEY=secret123
```

Run with env file:
```bash
podman run -d \
  --name myapp \
  --env-file .env \
  -e APP_ENV=production \
  alpine sleep 3600

# Verify env vars
podman exec myapp env | grep -E 'DATABASE|API|APP'

# Cleanup
podman rm -f myapp
```

**Task 3: Restart Policies**

```bash
# Run with auto-restart
podman run -d \
  --name crashapp \
  --restart=on-failure:3 \
  alpine sh -c "sleep 5; exit 1"

# Watch it restart
watch podman ps -a

# Check restart count
podman inspect crashapp --format='{{.RestartCount}}'

# Cleanup
podman rm -f crashapp
```

**Task 4: Health Checks**

```bash
# Run nginx with health check
podman run -d \
  --name healthy-web \
  -p 8080:80 \
  --health-cmd='curl -f http://localhost:80 || exit 1' \
  --health-interval=10s \
  --health-retries=3 \
  nginx:alpine

# Check health status
watch podman ps

# View health check logs
podman inspect healthy-web --format='{{json .State.Health}}' | jq

# Cleanup
podman rm -f healthy-web
```

{% endcollapsible %}

---

### Common Patterns

**Temporary debug container:**
```bash
podman run --rm -it alpine sh
```

**One-off task:**
```bash
podman run --rm -v $(pwd):/work alpine sh -c "ls -la /work"
```

**Database container:**
```bash
podman run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=mysecret \
  -e POSTGRES_DB=myapp \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  --restart=unless-stopped \
  postgres:15-alpine
```

**Application container:**
```bash
podman run -d \
  --name api \
  -e DATABASE_URL=postgresql://postgres:mysecret@postgres:5432/myapp \
  --network app-network \
  -p 8000:8000 \
  --restart=unless-stopped \
  --read-only \
  --tmpfs /tmp \
  --user 1000:1000 \
  myapp:latest
```

> **Resources**
>
> * [Podman Run Documentation](https://docs.podman.io/en/latest/markdown/podman-run.1.html)
> * [Container Restart Policies](https://docs.podman.io/en/latest/markdown/podman-run.1.html#restart)
> * [Health Checks](https://docs.docker.com/engine/reference/builder/#healthcheck)
