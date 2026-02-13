---
sectionid: architecture
sectionclass: h2
title: Container Architecture Internals
parent-id: concepts
---

### How Containers Actually Work

Containers aren't magic - they're built on three fundamental Linux kernel features:

1. **Namespaces** - Isolation ("what you can see")
2. **cgroups** - Resource limits ("what you can use")
3. **Layered filesystems** - Efficient storage ("copy-on-write")

---

### 1. Linux Namespaces: The Isolation Illusion

Namespaces make each container think it's running on its own dedicated machine.

**Types of Namespaces:**

**PID Namespace** - Process isolation
```bash
# On host: your app might be PID 42
# Inside container: same app appears as PID 1

# Try it:
podman run --rm alpine sh -c "ps aux"
# Notice: the shell process is PID 1!
```

**Network Namespace** - Network isolation
```bash
# Each container gets its own:
# - Network interfaces
# - IP addresses
# - Routing tables
# - Firewall rules

# Demo:
podman run --rm alpine ip addr show
# You'll see a unique network interface
```

**Mount Namespace** - Filesystem isolation
```bash
# Each container sees its own filesystem
# Changes don't affect the host or other containers

podman run --rm -v /:/host alpine ls /host
# The container can access host files only if explicitly mounted
```

**UTS Namespace** - Hostname isolation
```bash
# Each container can have its own hostname
podman run --rm --hostname my-container alpine hostname
# Output: my-container
```

**IPC Namespace** - Inter-Process Communication isolation

**User Namespace** - User ID isolation (rootless containers)

### Exercise: Exploring Namespaces

{% collapsible %}

**1. PID Namespace Demonstration:**

```bash
# Terminal 1: Run a container
podman run -it --name test-namespace alpine sh

# Inside container:
ps aux
# Notice: your shell is PID 1!

# Terminal 2: Check from host
podman top test-namespace
# The same process has a completely different PID on the host!
```

**2. Network Namespace:**

```bash
# Run two containers and compare their network configs
podman run --rm alpine ip addr show
podman run --rm alpine ip addr show
# Different IP addresses!
```

**3. Hostname Namespace:**

```bash
# Create containers with different hostnames
podman run --rm --hostname webapp alpine hostname
podman run --rm --hostname database alpine hostname
```

{% endcollapsible %}

---

### 2. cgroups: Resource Control

**cgroups (Control Groups)** ensure containers can't monopolize system resources.

**What cgroups limit:**

**CPU Limits:**
```bash
# Limit container to 50% of one CPU core
podman run --cpus=0.5 nginx

# Limit to 2 full cores
podman run --cpus=2 nginx
```

**Memory Limits:**
```bash
# Limit to 512MB RAM
podman run --memory=512m nginx

# With memory + swap limit
podman run --memory=512m --memory-swap=1g nginx

# What happens when limit is exceeded? The container is killed (OOM - Out Of Memory)
```

**Disk I/O Limits:**
```bash
# Limit read/write speed
podman run --device-read-bps /dev/sda:10mb nginx
```

**PID Limits:**
```bash
# Prevent fork bombs (malicious process creation)
podman run --pids-limit 100 nginx
```

### Exercise: Testing Resource Limits

{% collapsible %}

**Test 1: Memory Limit with OOM Kill**

Create a file `memory-test.py`:
```python
# This will consume memory until killed
data = []
while True:
    data.append(' ' * 10**6)  # Allocate 1MB chunks
    print(f"Allocated {len(data)} MB")
```

Run with limit:
```bash
# This container will be killed when it exceeds 100MB
podman run --rm --memory=100m -v $(pwd):/app python:alpine python /app/memory-test.py
```

**Test 2: CPU Limits**

```bash
# Unlimited CPU
podman run --rm alpine sh -c "while true; do :; done" &

# Limited to 10% CPU
podman run --rm --cpus=0.1 alpine sh -c "while true; do :; done" &

# Monitor with:
podman stats
```

Press `Ctrl+C` to stop monitoring, then clean up:
```bash
podman stop $(podman ps -q)
```

{% endcollapsible %}

---

### 3. Image Layers and Copy-on-Write

**How it works:**

- Container images are built in **layers**
- Each layer is **read-only**
- Layers are **stacked** on top of each other
- Containers add a **writable layer** on top
- Multiple containers **share the same base layers**

**Efficiency Example:**

```
Base Image: Alpine (5MB)
   ├─ Layer 1: OS files (5MB)

Application Image: my-app (50MB total)
   ├─ Layer 1: Alpine (5MB) ← SHARED
   ├─ Layer 2: Python (30MB)
   ├─ Layer 3: Dependencies (10MB)
   └─ Layer 4: App code (5MB)

Running 100 containers = 50MB + (100 × writable layers)
NOT 100 × 50MB = 5GB!
```

### Exercise: Visualizing Layers

{% collapsible %}

**1. Inspect Image Layers:**

```bash
# Pull an image
podman pull nginx:alpine

# View layer history
podman history nginx:alpine

# Detailed layer information
podman inspect nginx:alpine | grep -A 20 RootFS
```

**2. Compare Image Sizes:**

```bash
# Pull different base images
podman pull alpine:latest
podman pull ubuntu:latest
podman pull debian:latest

# Compare sizes
podman images
```

**3. Demonstrate Copy-on-Write:**

```bash
# Run 3 containers from same image
podman run -d --name web1 nginx:alpine
podman run -d --name web2 nginx:alpine
podman run -d --name web3 nginx:alpine

# Check disk usage
podman system df

# Notice: 3 containers but minimal extra disk usage!

# Cleanup
podman rm -f web1 web2 web3
```

{% endcollapsible %}

---

### Summary

**Namespaces** = Isolation (PID, Network, Mount, UTS, IPC, User)

**cgroups** = Resource limits (CPU, Memory, I/O, PIDs)

**Layers** = Efficient storage (Copy-on-Write, shared base images)

These three technologies combine to create the container abstraction: lightweight, isolated, and efficient application environments.

> **Resources**
>
> * [Linux Namespaces Documentation](https://man7.org/linux/man-pages/man7/namespaces.7.html)
> * [cgroups Documentation](https://man7.org/linux/man-pages/man7/cgroups.7.html)
> * [Container Storage Documentation](https://docs.podman.io/en/latest/markdown/podman-info.1.html)
