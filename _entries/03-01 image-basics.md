---
sectionid: imagebasics
sectionclass: h2
title: Image Management Basics
parent-id: images
---

### Understanding Container Images

A container image is a **read-only template** containing:
- Application code
- Runtime environment
- Libraries and dependencies
- Configuration files
- Metadata (environment variables, exposed ports, entrypoint)

**Image Registries:** Centralized repositories for storing and distributing images

- **Docker Hub** - hub.docker.com (default registry)
- **Quay.io** - Red Hat's registry
- **GitHub Container Registry** - ghcr.io
- **Private registries** - ACR (Azure), ECR (AWS), GCR (Google Cloud)

---

### Pulling Images

```bash
# Pull latest version
podman pull nginx

# Pull specific version (tag)
podman pull nginx:1.25-alpine

# Pull from different registry
podman pull quay.io/podman/hello

# Pull specific platform (for M1/M2 Macs or ARM)
podman pull --platform linux/amd64 nginx
```

### Listing Images

```bash
# List all images
podman images

# List with digests
podman images --digests

# Filter images
podman images nginx
podman images "nginx:*alpine"
```

### Inspecting Images

```bash
# Get detailed JSON information
podman inspect nginx:alpine

# Extract specific fields
podman inspect nginx:alpine --format='{{.Architecture}}'
podman inspect nginx:alpine --format='{{.Config.Env}}'
podman inspect nginx:alpine --format='{{.Config.ExposedPorts}}'

# View image history (layers)
podman history nginx:alpine
```

### Tagging Images

Tags help version and organize images. Use semantic versioning!

```bash
# Tag format: registry/repository:tag
# Examples:
#   nginx:1.25
#   myapp:v2.3.1
#   ghcr.io/username/app:latest

# Create a tag
podman tag nginx:alpine my-nginx:v1.0

# Tag for different environments
podman tag myapp:latest myapp:dev
podman tag myapp:latest myapp:prod-v1.2.0

# Tag for registry
podman tag myapp:latest ghcr.io/username/myapp:1.0.0
```

### Removing Images

```bash
# Remove specific image
podman rmi nginx:alpine

# Remove by ID (first 3 characters are usually enough)
podman rmi a1b2c3

# Remove all unused images
podman image prune

# Remove all dangling images (untagged)
podman image prune -a

# Force remove (even if containers exist)
podman rmi -f nginx
```

### Saving and Loading Images

Useful for air-gapped environments or sharing images without a registry.

```bash
# Save image to tar file
podman save nginx:alpine -o nginx-alpine.tar

# Load image from tar file
podman load -i nginx-alpine.tar

# Save multiple images
podman save nginx:alpine redis:alpine -o my-images.tar
```

---

### Exercise: Image Management

{% collapsible %}

**Task 1: Pull and Inspect Different Base Images**

```bash
# Pull three common base images
podman pull alpine:latest
podman pull python:3.12-slim
podman pull ubuntu:22.04

# Compare their sizes
podman images

# Inspect Alpine
podman inspect alpine:latest --format='{{.Size}}' | numfmt --to=iec

# View layer history
podman history alpine:latest
podman history python:3.12-slim
podman history ubuntu:22.04
```

**Expected Results:**
- Alpine: ~5-7 MB (minimal!)
- Python Slim: ~150-180 MB
- Ubuntu: ~70-80 MB

**Task 2: Tag Practice**

```bash
# Pull nginx
podman pull nginx:alpine

# Create multiple tags simulating different environments
podman tag nginx:alpine myapp:latest
podman tag nginx:alpine myapp:dev
podman tag nginx:alpine myapp:v1.0.0
podman tag nginx:alpine myapp:prod-20240213

# List all tags
podman images myapp
```

**Task 3: Image Cleanup**

```bash
# See current disk usage
podman system df

# Remove unused images
podman image prune

# Check disk usage again
podman system df
```

{% endcollapsible %}

---

### Choosing the Right Base Image

**Selection criteria:**

| Base Image | Size | Use Case | Security | Compatibility |
|------------|------|----------|----------|---------------|
| **scratch** | 0 MB | Compiled binaries (Go) | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **alpine** | ~5 MB | Microservices, APIs | ⭐⭐⭐⭐ | ⭐⭐⭐ (musl libc) |
| **distroless** | ~20 MB | Production apps | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **slim variants** | ~80 MB | General purpose | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **full distros** | 200+ MB | Development, complex deps | ⭐⭐ | ⭐⭐⭐⭐⭐ |

**Recommendations:**

```dockerfile
# ❌ Don't use
FROM ubuntu:latest

# ✅ Do use - specify exact version
FROM ubuntu:22.04

# ✅ Better - use slim variant
FROM python:3.12-slim

# ✅ Best - use alpine for production microservices
FROM python:3.12-alpine

# ✅ Best - use distroless for maximum security
FROM gcr.io/distroless/python3
```

---

### Vulnerability Scanning

Always scan images for known security vulnerabilities!

**Install Trivy:**

```bash
# macOS
brew install trivy

# Linux
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

**Scan images:**

```bash
# Scan an image
trivy image nginx:latest

# Scan with severity filter
trivy image --severity HIGH,CRITICAL nginx:latest

# Scan local image
trivy image myapp:latest

# Generate JSON report
trivy image -f json -o report.json nginx:alpine
```

> **Resources**
>
> * [Docker Hub](https://hub.docker.com/)
> * [Podman Images Documentation](https://docs.podman.io/en/latest/markdown/podman-images.1.html)
> * [Trivy Vulnerability Scanner](https://trivy.dev/)
> * [Base Image Security Guide](https://snyk.io/blog/choosing-the-best-base-image-for-containers/)
