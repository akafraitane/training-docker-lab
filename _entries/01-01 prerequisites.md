---
sectionid: prereq
sectionclass: h2
title: Prerequisites
parent-id: intro
---

### Tools and Environment

This workshop can be completed on **Windows**, **macOS**, or **Linux**. You'll need to install Podman (recommended) or Docker Desktop.

### Installation Options

#### Windows

**Option 1: Podman Desktop (Recommended)**

1. Install WSL2:
   ```powershell
   wsl --install
   ```

2. Download and install [Podman Desktop](https://podman-desktop.io/downloads)

3. Initialize Podman machine:
   ```powershell
   podman machine init --cpus 4 --memory 8192 --disk-size 50
   podman machine start
   ```

**Option 2: Docker Desktop**

Download and install [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)

{% collapsible %}

After installation, verify with:

```sh
podman --version
# or
docker --version
```

{% endcollapsible %}

#### macOS

**Option 1: Podman (Recommended)**

```bash
# Using Homebrew
brew install podman

# Initialize machine
podman machine init --cpus 4 --memory 8192 --disk-size 50
podman machine start
```

**Option 2: Docker Desktop**

Download [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/)

{% collapsible %}

Verify installation:

```sh
podman --version
podman run hello-world
```

{% endcollapsible %}

#### Linux

**Ubuntu/Debian:**

```bash
sudo apt update
sudo apt install -y podman
```

**Fedora/RHEL:**

```bash
sudo dnf install -y podman
```

**Optional: Install Podman Desktop**

```bash
flatpak install flathub io.podman_desktop.PodmanDesktop
```

{% collapsible %}

Verify installation:

```sh
podman --version
podman info
```

{% endcollapsible %}

### Verification Script

Run this verification script to ensure your environment is properly configured:

```bash
# Test 1: Version check
echo "Testing Podman/Docker version..."
podman --version || docker --version

# Test 2: Run a simple container
echo "Testing container execution..."
podman run --rm hello-world || docker run --rm hello-world

# Test 3: Port mapping
echo "Testing port mapping..."
podman run -d --name test-nginx -p 8080:80 nginx:alpine
curl localhost:8080
podman stop test-nginx
podman rm test-nginx

# Test 4: Volume mounting
echo "Testing volume mounts..."
podman run --rm -v $(pwd):/data alpine ls /data

echo "✅ All tests passed! Your environment is ready."
```

### Why Podman vs Docker?

This workshop primarily uses **Podman** because:

- **Daemonless architecture** - No root daemon running continuously
- **Rootless by default** - Enhanced security model
- **100% Docker CLI compatible** - All Docker commands work with Podman
- **Open source** - No licensing concerns
- **Native Kubernetes pods support** - Better cloud-native integration

> **Note:** All commands in this workshop work with both `podman` and `docker`. Simply replace `podman` with `docker` if you're using Docker.

### Text Editor

You'll need a text editor for creating Dockerfiles and configuration files:

- **VS Code** (recommended) - [Download here](https://code.visualstudio.com/)
- **Vim**, **Nano**, or any text editor of your choice

### Optional Tools

- **Trivy** - For vulnerability scanning (we'll cover this later)
- **Git** - For cloning sample applications
