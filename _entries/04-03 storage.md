---
sectionid: storage
sectionclass: h2
title: Persistent Storage
parent-id: running
---

### The Problem: Container Filesystem is Ephemeral

**By default, when you delete a container, all its data is lost!**

```bash
# Create container and write data
podman run -d --name temp-db postgres:15-alpine

# Data is stored... but where?
podman exec temp-db psql -U postgres -c "CREATE DATABASE myapp;"

# Delete container
podman rm -f temp-db

# Data is gone! 💀
```

**Solution:** Use volumes or bind mounts to persist data outside the container.

---

### Storage Types

| Type | Description | Use Case | Path |
|------|-------------|----------|------|
| **Volume** | Managed by Podman | Databases, production data | `/var/lib/containers/storage/volumes/` |
| **Bind Mount** | Direct host path | Development, config files | Any host path (`/home/user/data`) |
| **tmpfs** | In-memory (RAM) | Temporary data, secrets | Memory only (not persisted) |

---

### Named Volumes (Recommended for Production)

**Advantages:**
- Managed by Podman
- Portable across containers
- Can be backed up easily
- Work on all platforms (Linux, macOS, Windows)
- Better performance than bind mounts

**Create and use volumes:**

```bash
# Create named volume
podman volume create mydata

# List volumes
podman volume ls

# Inspect volume
podman volume inspect mydata

# Use volume in container
podman run -d \
  --name postgres \
  -v mydata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15-alpine

# Volume persists data even after container is removed!
podman rm -f postgres

# Start new container with same volume - data is still there!
podman run -d \
  --name postgres-new \
  -v mydata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15-alpine

# Remove volume (only when no containers use it)
podman volume rm mydata

# Force remove
podman volume rm -f mydata

# Remove all unused volumes
podman volume prune
```

**Anonymous volumes:**

```bash
# Podman creates anonymous volume automatically
podman run -d -v /var/lib/postgresql/data postgres:15-alpine

# List (shows random name like "4f8a2c...")
podman volume ls
```

---

### Bind Mounts (Development)

**Mount host directory directly into container:**

```bash
# Bind mount current directory
podman run -d \
  -v $(pwd)/data:/app/data \
  nginx:alpine

# Absolute paths
podman run -d \
  -v /home/user/myapp:/app \
  myapp:latest

# Read-only bind mount
podman run -d \
  -v $(pwd)/config:/app/config:ro \
  myapp:latest

# Bind mount with SELinux label (Linux only)
podman run -d \
  -v $(pwd)/data:/app/data:z \
  myapp:latest
```

**Common use cases:**

**Development (live code reload):**
```bash
# Code changes on host immediately reflect in container
podman run -d \
  -v $(pwd)/src:/app/src \
  -p 8000:8000 \
  python:3.12 \
  python -m http.server --directory /app/src
```

**Configuration files:**
```bash
# Mount config directory
podman run -d \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  nginx:alpine
```

**Database data (development):**
```bash
# Easy to backup/inspect
podman run -d \
  -v $(pwd)/pgdata:/var/lib/postgresql/data \
  postgres:15-alpine
```

---

### tmpfs (In-Memory Storage)

**Use for:**
- Sensitive temporary data (secrets, tokens)
- High-performance temporary files
- Data that doesn't need to persist

```bash
# Create tmpfs mount
podman run -d \
  --tmpfs /tmp:rw,size=100m,mode=1777 \
  myapp:latest

# Multiple tmpfs mounts
podman run -d \
  --tmpfs /tmp \
  --tmpfs /var/cache \
  nginx:alpine
```

**Example: Secure container with read-only root filesystem:**

```bash
podman run -d \
  --read-only \
  --tmpfs /tmp \
  --tmpfs /var/run \
  --tmpfs /var/cache \
  nginx:alpine
```

---

### Volume Management

```bash
# Create volume with specific driver
podman volume create --driver local mydata

# Create volume with labels
podman volume create \
  --label env=production \
  --label app=myapp \
  proddata

# Backup volume
podman run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/mydata-backup.tar.gz /data

# Restore volume
podman run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/mydata-backup.tar.gz -C /

# Copy data between volumes
podman run --rm \
  -v oldvol:/from \
  -v newvol:/to \
  alpine sh -c "cp -av /from/. /to/"

# Inspect volume location
podman volume inspect mydata --format='{{.Mountpoint}}'

# View volume usage
podman system df -v
```

---

### Exercise: Persistent Database

{% collapsible %}

**Task: Run PostgreSQL with persistent data**

**Step 1: Create volume and run database**

```bash
# Create volume
podman volume create pgdata

# Run PostgreSQL
podman run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=mysecret \
  -e POSTGRES_DB=myapp \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15-alpine

# Wait for database to start
sleep 5

# Create some data
podman exec postgres psql -U postgres -d myapp -c "
  CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
  );

  INSERT INTO users (name, email) VALUES
    ('Alice', 'alice@example.com'),
    ('Bob', 'bob@example.com');
"

# Verify data
podman exec postgres psql -U postgres -d myapp -c "SELECT * FROM users;"
```

**Step 2: Delete container and recreate**

```bash
# Delete container (scary!)
podman rm -f postgres

# Volume still exists
podman volume ls

# Start new container with same volume
podman run -d \
  --name postgres-new \
  -e POSTGRES_PASSWORD=mysecret \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15-alpine

# Wait for startup
sleep 5

# Data is still there! 🎉
podman exec postgres-new psql -U postgres -d myapp -c "SELECT * FROM users;"
```

**Step 3: Backup and restore**

```bash
# Backup volume
podman run --rm \
  -v pgdata:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/pgdata-backup.tar.gz -C /data .

# Create new volume
podman volume create pgdata-restored

# Restore backup
podman run --rm \
  -v pgdata-restored:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/pgdata-backup.tar.gz -C /data

# Verify restored data
podman run -d \
  --name postgres-restored \
  -e POSTGRES_PASSWORD=mysecret \
  -v pgdata-restored:/var/lib/postgresql/data \
  postgres:15-alpine

sleep 5

podman exec postgres-restored psql -U postgres -d myapp -c "SELECT * FROM users;"
```

**Cleanup:**

```bash
podman rm -f postgres-new postgres-restored
podman volume rm pgdata pgdata-restored
rm pgdata-backup.tar.gz
```

{% endcollapsible %}

---

### Exercise: Development with Bind Mounts

{% collapsible %}

**Task: Live-reload Python application**

**Step 1: Create application**

Create `app.py`:
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from live-reload container! v1.0'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000, debug=True)
```

Create `requirements.txt`:
```
flask==3.0.0
```

**Step 2: Run with bind mount**

```bash
# Run with live reload
podman run -d \
  --name dev-app \
  -v $(pwd):/app \
  -w /app \
  -p 8000:8000 \
  python:3.12-slim \
  sh -c "pip install -r requirements.txt && python app.py"

# Wait for startup
sleep 5

# Test
curl localhost:8000
```

**Step 3: Modify code (without restarting container!)**

Edit `app.py` and change the version to `v2.0`:
```python
return 'Hello from live-reload container! v2.0'
```

```bash
# Flask auto-reloads!
curl localhost:8000
# Should show v2.0!
```

**Cleanup:**

```bash
podman rm -f dev-app
```

{% endcollapsible %}

---

### Volume Best Practices

✅ **DO:**
- Use named volumes for production databases
- Use bind mounts for development
- Backup important volumes regularly
- Use read-only mounts for configs (`:ro`)
- Use tmpfs for sensitive temporary data

❌ **DON'T:**
- Store data in container without volumes
- Use bind mounts in production (portability issues)
- Mount entire host root (`-v /:/host`) without good reason
- Forget to backup volumes before deleting

---

### Volume Drivers

Podman supports custom volume drivers:

```bash
# Local driver (default)
podman volume create --driver local mydata

# NFS volume
podman volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/path/to/share \
  nfs-volume

# CIFS/SMB volume
podman volume create --driver local \
  --opt type=cifs \
  --opt o=username=user,password=pass \
  --opt device=//server/share \
  smb-volume
```

> **Resources**
>
> * [Podman Volume Documentation](https://docs.podman.io/en/latest/markdown/podman-volume.1.html)
> * [Manage Data in Containers](https://docs.docker.com/storage/)
> * [Volume Backup Strategies](https://docs.docker.com/storage/volumes/#back-up-restore-or-migrate-data-volumes)
