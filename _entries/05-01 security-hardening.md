---
sectionid: securityhardening
sectionclass: h2
title: Container Security Hardening
parent-id: security
---

### Container Security Principles

**Defense in Depth:** Multiple layers of security
**Least Privilege:** Run with minimum required permissions
**Immutability:** Containers should be disposable and replaceable
**Zero Trust:** Verify everything, trust nothing

---

### Image Security

**1. Use Trusted Base Images**

✅ **DO:**
```dockerfile
# Official images from verified publishers
FROM nginx:1.25-alpine
FROM python:3.12-slim
FROM postgres:15-alpine

# Specific digests (immutable)
FROM nginx:alpine@sha256:a1b2c3d4...
```

❌ **DON'T:**
```dockerfile
# Untrusted random images
FROM random-user/suspicious-image

# Latest tag (changes over time)
FROM ubuntu:latest

# Unknown sources
FROM hub.example.com/untrusted/image
```

**2. Scan for Vulnerabilities**

```bash
# Install Trivy scanner
brew install trivy  # macOS
# or download from https://trivy.dev

# Scan image
trivy image nginx:alpine

# Show only HIGH and CRITICAL
trivy image --severity HIGH,CRITICAL nginx:alpine

# Scan local Dockerfile
trivy config Dockerfile

# Fail build if vulnerabilities found
trivy image --exit-code 1 --severity CRITICAL myapp:latest
```

**3. Use Minimal Base Images**

```dockerfile
# ❌ Full Ubuntu: ~70MB, lots of attack surface
FROM ubuntu:22.04

# ✅ Alpine: ~5MB, minimal attack surface
FROM alpine:3.19

# ✅ Distroless: ~20MB, no shell, no package manager!
FROM gcr.io/distroless/python3
```

**4. Never Store Secrets in Images**

❌ **DON'T:**
```dockerfile
# NEVER do this!
ENV DATABASE_PASSWORD=mysecret
COPY .env /app/.env
RUN echo "API_KEY=secret123" > /app/config
```

✅ **DO:**
```bash
# Pass secrets at runtime
podman run -e DATABASE_PASSWORD=secret myapp

# Use env files
podman run --env-file .env.prod myapp

# Use secret management
podman run -e DATABASE_PASSWORD=$(cat /run/secrets/db_pass) myapp
```

**5. Use Specific Tags**

```dockerfile
# ❌ Changes over time
FROM python:latest

# ✅ Specific version
FROM python:3.12-slim

# ✅ Immutable digest
FROM python:3.12-slim@sha256:abc123...
```

---

### Runtime Security

**1. Run as Non-Root User**

❌ **BAD - Runs as root:**
```dockerfile
FROM python:3.12-slim
COPY app.py /app/
CMD ["python", "/app/app.py"]
```

✅ **GOOD - Non-root user:**
```dockerfile
FROM python:3.12-slim

# Create user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Set ownership
COPY --chown=appuser:appuser app.py /app/

# Switch user
USER appuser

CMD ["python", "/app/app.py"]
```

```bash
# Or specify at runtime
podman run --user 1000:1000 myapp
```

**2. Read-Only Root Filesystem**

```bash
# Make root filesystem read-only
podman run -d \
  --read-only \
  --tmpfs /tmp \
  --tmpfs /var/run \
  --tmpfs /var/cache \
  nginx:alpine
```

**Dockerfile with read-only support:**
```dockerfile
FROM nginx:alpine

# Application doesn't write to disk
# All writes go to tmpfs mounts
VOLUME /tmp
VOLUME /var/cache/nginx
```

**3. Drop Linux Capabilities**

```bash
# Drop ALL capabilities, add only what's needed
podman run -d \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  -p 80:80 \
  nginx:alpine

# Common capabilities:
# NET_BIND_SERVICE - bind to ports < 1024
# CHOWN - change file ownership
# SETUID/SETGID - change user/group IDs
```

**4. Prevent Privilege Escalation**

```bash
podman run -d \
  --security-opt=no-new-privileges \
  myapp:latest
```

**5. Resource Limits (Prevent DoS)**

```bash
podman run -d \
  --memory=512m \
  --memory-swap=512m \
  --cpus=1.0 \
  --pids-limit=100 \
  myapp:latest
```

**6. Rootless Containers (Advanced)**

Podman can run completely without root:

```bash
# Run as regular user (no sudo needed!)
podman run -d nginx:alpine

# Check rootless mode
podman info | grep rootless
```

---

### Network Security

**1. Custom Networks (Isolation)**

```bash
# Create isolated network
podman network create --internal backend-net

# Containers can't reach internet, only each other
podman run -d --network backend-net postgres:15-alpine
```

**2. Restrict Port Exposure**

```bash
# ❌ Exposed to all interfaces
podman run -d -p 8080:80 nginx

# ✅ Only localhost
podman run -d -p 127.0.0.1:8080:80 nginx
```

**3. Use TLS for Inter-Container Communication**

For production, enable TLS between containers (beyond scope of this workshop, but important!)

---

### Complete Security Checklist

#### Image Security
- [ ] Use official or verified images
- [ ] Use specific tags, not `:latest`
- [ ] Use minimal base images (Alpine, Distroless)
- [ ] Scan images with Trivy
- [ ] No secrets in images
- [ ] Multi-stage builds to minimize size

#### Runtime Security
- [ ] Run as non-root user
- [ ] Read-only root filesystem
- [ ] Drop all capabilities, add only required
- [ ] Enable no-new-privileges
- [ ] Set memory/CPU limits
- [ ] Set PID limits

#### Network Security
- [ ] Use custom networks for isolation
- [ ] Expose ports only to localhost when possible
- [ ] Use network policies
- [ ] Enable TLS between services

#### Storage Security
- [ ] Use volumes, not bind mounts in production
- [ ] Encrypt sensitive data at rest
- [ ] Limit volume permissions

#### Monitoring
- [ ] Enable health checks
- [ ] Collect and analyze logs
- [ ] Monitor resource usage
- [ ] Set up alerts for anomalies

---

### Exercise: Production-Hardened Container

{% collapsible %}

**Task: Build and run a security-hardened Flask application**

**Step 1: Create application**

Create `app.py`:
```python
from flask import Flask, jsonify
app = Flask(__name__)

@app.route('/')
def index():
    return jsonify({"status": "healthy", "version": "1.0"})

@app.route('/health')
def health():
    return jsonify({"status": "ok"}), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
```

Create `requirements.txt`:
```
flask==3.0.0
gunicorn==21.2.0
```

**Step 2: Create hardened Dockerfile**

Create `Dockerfile`:
```dockerfile
# Build stage
FROM python:3.12-slim AS builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Runtime stage
FROM python:3.12-slim

# Security: Create non-root user
RUN groupadd -r appuser && \
    useradd -r -g appuser -u 1000 appuser

WORKDIR /app

# Copy dependencies from builder
COPY --from=builder /root/.local /root/.local

# Copy application code
COPY --chown=appuser:appuser app.py .

# Set PATH
ENV PATH=/root/.local/bin:$PATH \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')" || exit 1

# Security: Switch to non-root user
USER appuser

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "2", "app:app"]
```

**Step 3: Build and scan**

```bash
# Build image
podman build -t secure-app:1.0 .

# Scan for vulnerabilities
trivy image secure-app:1.0

# Scan Dockerfile
trivy config Dockerfile
```

**Step 4: Run with security hardening**

```bash
# Run with all security features
podman run -d \
  --name secure-app \
  --hostname secure-app \
  `# Resource limits` \
  --memory=256m \
  --memory-reservation=128m \
  --cpus=0.5 \
  --pids-limit=50 \
  `# Security options` \
  --read-only \
  --tmpfs /tmp \
  --cap-drop=ALL \
  --security-opt=no-new-privileges \
  `# Network` \
  -p 127.0.0.1:8000:8000 \
  `# Reliability` \
  --restart=unless-stopped \
  `# Image` \
  secure-app:1.0

# Wait for startup
sleep 5

# Test
curl http://localhost:8000/
curl http://localhost:8000/health

# Check health status
podman inspect secure-app --format='{{.State.Health.Status}}'

# Verify non-root user
podman exec secure-app whoami
# Should show: appuser

# Try to write to filesystem (should fail)
podman exec secure-app touch /test.txt
# Error: Read-only file system

# But tmpfs works
podman exec secure-app sh -c "echo test > /tmp/test.txt && cat /tmp/test.txt"
```

**Step 5: Verify security**

```bash
# Check user
podman exec secure-app id
# Should show uid=1000(appuser)

# Check capabilities
podman inspect secure-app --format='{{.HostConfig.CapDrop}}'
# Should show: [ALL]

# Check filesystem
podman inspect secure-app --format='{{.HostConfig.ReadonlyRootfs}}'
# Should show: true

# View resource limits
podman stats --no-stream secure-app
```

**Cleanup:**

```bash
podman rm -f secure-app
podman rmi secure-app:1.0
```

{% endcollapsible %}

---

### Security Tools and Resources

**Scanning Tools:**
- **Trivy** - Vulnerability scanner
- **Syft** - SBOM (Software Bill of Materials) generator
- **Grype** - Vulnerability scanner
- **Clair** - Static analysis of vulnerabilities

**Runtime Security:**
- **Falco** - Runtime threat detection
- **AppArmor** / **SELinux** - Mandatory access control
- **Seccomp** - System call filtering

**Best Practices References:**

```bash
# Docker Bench Security
git clone https://github.com/docker/docker-bench-security.git
cd docker-bench-security
sudo sh docker-bench-security.sh

# CIS Kubernetes Benchmark
# https://www.cisecurity.org/benchmark/kubernetes
```

---

### Production Deployment Checklist

Before deploying to production:

**Images:**
- [ ] Scanned for vulnerabilities (CRITICAL/HIGH fixed)
- [ ] Using specific tags or digests
- [ ] Multi-stage build to minimize size
- [ ] Minimal base image
- [ ] No secrets embedded

**Containers:**
- [ ] Running as non-root
- [ ] Read-only root filesystem
- [ ] Capabilities dropped
- [ ] Resource limits set
- [ ] Health checks enabled
- [ ] Restart policy configured

**Networking:**
- [ ] Proper network isolation
- [ ] TLS enabled for external communication
- [ ] Minimal port exposure

**Monitoring:**
- [ ] Logging configured
- [ ] Metrics collected
- [ ] Alerts configured

**Compliance:**
- [ ] GDPR/HIPAA/PCI compliance verified
- [ ] Security audit passed
- [ ] Incident response plan ready

> **Resources**
>
> * [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
> * [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
> * [Trivy Documentation](https://trivy.dev/)
> * [Podman Security Guide](https://docs.podman.io/en/latest/markdown/podman-run.1.html#security-options)
