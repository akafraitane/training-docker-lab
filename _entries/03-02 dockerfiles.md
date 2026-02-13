---
sectionid: dockerfiles
sectionclass: h2
title: Building Images with Dockerfiles
parent-id: images
---

### What is a Dockerfile?

A **Dockerfile** is a text file containing instructions to build a container image.

**Basic structure:**

```dockerfile
# Start from a base image
FROM python:3.12-slim

# Set working directory
WORKDIR /app

# Copy files
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 8000

# Default command
CMD ["python", "app.py"]
```

---

### Dockerfile Instructions

**FROM** - Base image (always first instruction)
```dockerfile
FROM alpine:3.19
FROM python:3.12-slim AS builder
```

**WORKDIR** - Set working directory
```dockerfile
WORKDIR /app
# All subsequent commands run from /app
```

**COPY** - Copy files from host to image
```dockerfile
COPY app.py /app/
COPY requirements.txt .
COPY src/ /app/src/
```

**ADD** - Like COPY but can also download URLs and extract archives (prefer COPY)
```dockerfile
ADD https://example.com/file.tar.gz /tmp/
```

**RUN** - Execute commands during build
```dockerfile
RUN apt-get update && apt-get install -y curl
RUN pip install flask
```

**ENV** - Set environment variables
```dockerfile
ENV FLASK_APP=app.py
ENV PORT=8000
```

**EXPOSE** - Document which ports the container listens on (doesn't actually publish)
```dockerfile
EXPOSE 8000
EXPOSE 443
```

**CMD** - Default command when container starts (can be overridden)
```dockerfile
CMD ["python", "app.py"]
CMD ["nginx", "-g", "daemon off;"]
```

**ENTRYPOINT** - Command that always runs (harder to override)
```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]  # Can override just the arguments
```

**USER** - Switch to non-root user
```dockerfile
USER appuser
```

**ARG** - Build-time variables
```dockerfile
ARG VERSION=1.0
RUN echo "Building version $VERSION"
```

**HEALTHCHECK** - Container health check
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8000/health || exit 1
```

---

### Building Images

```bash
# Basic build
podman build -t myapp:latest .

# Build with custom Dockerfile name
podman build -f Dockerfile.prod -t myapp:prod .

# Build with build arguments
podman build --build-arg VERSION=2.0 -t myapp:2.0 .

# Build without cache (force rebuild)
podman build --no-cache -t myapp:latest .

# Build for specific platform
podman build --platform linux/amd64 -t myapp:latest .
```

---

### Multi-Stage Builds

**Problem:** Build tools bloat the final image.

**Solution:** Use multi-stage builds to separate build and runtime environments.

**Example: Python Application**

❌ **Single-stage (BAD):**

```dockerfile
FROM python:3.12
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "app.py"]
# Result: ~900 MB image!
```

✅ **Multi-stage (GOOD):**

```dockerfile
# Stage 1: Build
FROM python:3.12 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Stage 2: Runtime
FROM python:3.12-slim
WORKDIR /app
# Copy only what's needed from builder stage
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
# Result: ~180 MB image (5x smaller!)
```

**Example: Go Application**

```dockerfile
# Build stage
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Runtime stage
FROM alpine:3.19
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
CMD ["./main"]
# Result: ~45 MB (from 1.2 GB build image!)
```

---

### Dockerfile Best Practices

**1. Order Layers by Change Frequency**

❌ **Bad - cache invalidated on every code change:**
```dockerfile
FROM python:3.12-slim
COPY . .  # Code changes often
RUN pip install -r requirements.txt  # Rebuilds every time!
```

✅ **Good - cache dependencies:**
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt  # Cached!
COPY . .  # Only this layer rebuilds on code change
```

**2. Minimize Layers**

❌ **Bad - too many layers:**
```dockerfile
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN apt-get clean
```

✅ **Good - combined:**
```dockerfile
RUN apt-get update && \
    apt-get install -y curl git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**3. Use .dockerignore**

Create `.dockerignore` file in your project root:

```
# .dockerignore
node_modules
.git
.env
.vscode
*.log
*.md
Dockerfile
.dockerignore
*.pyc
__pycache__
```

**4. Don't Run as Root**

❌ **Bad:**
```dockerfile
FROM python:3.12-slim
COPY app.py .
CMD ["python", "app.py"]  # Runs as root!
```

✅ **Good:**
```dockerfile
FROM python:3.12-slim
RUN useradd -m -u 1000 appuser
USER appuser
COPY app.py /home/appuser/
WORKDIR /home/appuser
CMD ["python", "app.py"]
```

**5. Use Specific Image Tags**

❌ **Bad:**
```dockerfile
FROM python:latest  # What version? Changes over time!
```

✅ **Good:**
```dockerfile
FROM python:3.12-slim  # Specific, reproducible
```

**6. Add Health Checks**

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1
```

**7. Use COPY Instead of ADD**

```dockerfile
# ❌ ADD can download URLs and extract archives (side effects!)
ADD app.tar.gz /app/

# ✅ COPY is explicit and predictable
COPY app/ /app/
```

---

### Exercise: Building Optimized Images

{% collapsible %}

**Task 1: Single-stage vs Multi-stage Build**

Create `app.py`:
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from optimized container!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
```

Create `requirements.txt`:
```
flask==3.0.0
```

Create `Dockerfile.single`:
```dockerfile
FROM python:3.12
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
```

Create `Dockerfile.multi`:
```dockerfile
# Build stage
FROM python:3.12 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Runtime stage
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY app.py .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

Build both:
```bash
# Single-stage
podman build -f Dockerfile.single -t myapp:single .

# Multi-stage
podman build -f Dockerfile.multi -t myapp:multi .

# Compare sizes
podman images | grep myapp
```

**Expected Result:** Multi-stage should be ~5-6x smaller!

**Task 2: Layer Caching**

Create `Dockerfile.optimized`:
```dockerfile
FROM python:3.12-slim

# Create non-root user
RUN useradd -m -u 1000 appuser

WORKDIR /app

# Install dependencies FIRST (cached unless requirements change)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy code LAST (changes frequently)
COPY app.py .

USER appuser

HEALTHCHECK --interval=30s CMD curl -f http://localhost:8000/ || exit 1

CMD ["python", "app.py"]
```

Build twice and notice the cache:
```bash
# First build
podman build -f Dockerfile.optimized -t myapp:opt .

# Modify app.py (not requirements.txt)
echo "# comment" >> app.py

# Second build - notice the cache hits!
podman build -f Dockerfile.optimized -t myapp:opt .
```

{% endcollapsible %}

---

### Production Dockerfile Template

```dockerfile
# Multi-stage production Dockerfile with best practices

# ===== Build Stage =====
FROM python:3.12-slim AS builder

# Install build dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Install Python dependencies
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# ===== Runtime Stage =====
FROM python:3.12-slim

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Install runtime dependencies only
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy dependencies from builder
COPY --from=builder /root/.local /root/.local

# Copy application code
COPY --chown=appuser:appuser . .

# Set PATH and environment
ENV PATH=/root/.local/bin:$PATH \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# Switch to non-root user
USER appuser

# Expose port (documentation only)
EXPOSE 8000

# Start application
CMD ["python", "app.py"]
```

> **Resources**
>
> * [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
> * [Multi-stage Builds](https://docs.docker.com/build/building/multi-stage/)
> * [Podman Build Documentation](https://docs.podman.io/en/latest/markdown/podman-build.1.html)
