---
sectionid: networking
sectionclass: h2
title: Container Networking
parent-id: running
---

### Container Network Basics

By default, containers are isolated. To communicate with the outside world or with each other, you need to understand container networking.

**Network Types:**

1. **Bridge (default)** - Isolated network with NAT
2. **Host** - Direct access to host network
3. **None** - Completely isolated (no network)
4. **Custom bridge** - User-defined isolated network with DNS

---

### Default Bridge Network

```bash
# Run container (uses default bridge automatically)
podman run -d --name web1 nginx:alpine

# Check IP address
podman inspect web1 --format='{{.NetworkSettings.IPAddress}}'

# Run another container
podman run -d --name web2 nginx:alpine
podman inspect web2 --format='{{.NetworkSettings.IPAddress}}'

# Containers can reach each other via IP (but NOT by name on default bridge!)
podman exec web1 ping -c 2 $(podman inspect web2 --format='{{.NetworkSettings.IPAddress}}')
```

---

### Custom Bridge Networks

**Custom networks provide:**
- Automatic DNS resolution (containers can reach each other by name!)
- Better isolation
- Network-level access control

```bash
# Create custom network
podman network create my-network

# List networks
podman network ls

# Inspect network
podman network inspect my-network

# Run containers on custom network
podman run -d --name db --network my-network postgres:15-alpine
podman run -d --name api --network my-network nginx:alpine

# Now containers can reach each other by NAME!
podman exec api ping -c 2 db
podman exec api nslookup db  # DNS works!
```

---

### Port Mapping

Expose container ports to the host:

```bash
# Map single port: -p HOST_PORT:CONTAINER_PORT
podman run -d -p 8080:80 nginx

# Multiple ports
podman run -d -p 8080:80 -p 8443:443 nginx

# Bind to specific interface
podman run -d -p 127.0.0.1:8080:80 nginx  # Only localhost
podman run -d -p 0.0.0.0:8080:80 nginx    # All interfaces

# Random host port
podman run -d -p 80 nginx

# UDP port
podman run -d -p 53:53/udp my-dns-server

# Expose all ports defined in Dockerfile
podman run -d -P nginx
```

**Check port mappings:**

```bash
# List ports for container
podman port nginx

# Check specific port
podman port nginx 80
```

---

### Host Network Mode

Container shares host's network stack (no isolation):

```bash
# Use host network
podman run -d --network=host nginx

# Container now uses host's IP directly
# Access via http://localhost:80 (no port mapping needed!)
```

**⚠️ Warning:** Host network mode reduces isolation and can cause port conflicts. Use only when necessary (performance-critical apps).

---

### Network-less Mode

Completely isolated (no network at all):

```bash
# Run without network
podman run --rm --network=none alpine ip addr show
# Only loopback interface!
```

---

### Connecting to Multiple Networks

```bash
# Create two networks
podman network create frontend
podman network create backend

# Run container on one network
podman run -d --name db --network backend postgres:15-alpine

# Connect to additional network
podman network connect frontend db

# Disconnect from network
podman network disconnect frontend db
```

---

### DNS and Service Discovery

On custom networks, containers automatically resolve each other's names:

```bash
podman network create mynet

# Run database
podman run -d --name postgres --network mynet \
  -e POSTGRES_PASSWORD=secret \
  postgres:15-alpine

# Run app that connects to database
podman run -d --name app --network mynet \
  -e DATABASE_URL=postgresql://postgres:secret@postgres:5432/mydb \
  myapp:latest

# DNS works!
podman exec app ping -c 2 postgres
podman exec app nslookup postgres
```

---

### Network Isolation Example

```bash
# Create separate networks for frontend and backend
podman network create frontend-net
podman network create backend-net

# Backend: database (only on backend network)
podman run -d --name db \
  --network backend-net \
  -e POSTGRES_PASSWORD=secret \
  postgres:15-alpine

# Middle tier: API (on BOTH networks)
podman run -d --name api \
  --network backend-net \
  -e DATABASE_URL=postgresql://postgres:secret@db:5432/mydb \
  myapi:latest

podman network connect frontend-net api

# Frontend: web server (only on frontend network)
podman run -d --name web \
  --network frontend-net \
  -p 8080:80 \
  -e API_URL=http://api:8000 \
  myweb:latest

# Result:
# - web can reach api ✅
# - web CANNOT reach db ✅ (security!)
# - api can reach db ✅
```

---

### Exercise: Multi-Container Networking

{% collapsible %}

**Task: Build a 3-tier application with proper network isolation**

**Setup:**

```bash
# Create networks
podman network create frontend
podman network create backend

# Start database (backend only)
podman run -d \
  --name postgres \
  --network backend \
  -e POSTGRES_PASSWORD=mysecret \
  -e POSTGRES_DB=appdb \
  postgres:15-alpine

# Start Redis cache (backend only)
podman run -d \
  --name redis \
  --network backend \
  redis:alpine

# Start API server (both networks)
podman run -d \
  --name api \
  --network backend \
  -e DATABASE_URL=postgresql://postgres:mysecret@postgres:5432/appdb \
  -e REDIS_URL=redis://redis:6379 \
  myapi:latest

podman network connect frontend api

# Start web frontend (frontend only, exposed to public)
podman run -d \
  --name web \
  --network frontend \
  -p 8080:80 \
  -e API_URL=http://api:8000 \
  nginx:alpine
```

**Test connectivity:**

```bash
# API can reach database
podman exec api ping -c 2 postgres

# API can reach Redis
podman exec api ping -c 2 redis

# Web can reach API
podman exec web ping -c 2 api

# Web CANNOT reach database (security!)
podman exec web ping -c 2 postgres
# This should fail!
```

**Inspect networks:**

```bash
# View containers in frontend network
podman network inspect frontend

# View containers in backend network
podman network inspect backend
```

**Cleanup:**

```bash
podman rm -f postgres redis api web
podman network rm frontend backend
```

{% endcollapsible %}

---

### Network Commands Reference

```bash
# Create network
podman network create [OPTIONS] NETWORK

# List networks
podman network ls

# Inspect network
podman network inspect NETWORK

# Remove network
podman network rm NETWORK

# Prune unused networks
podman network prune

# Connect container to network
podman network connect NETWORK CONTAINER

# Disconnect container from network
podman network disconnect NETWORK CONTAINER
```

---

### Custom Network with Specific Subnet

```bash
# Create network with custom subnet
podman network create \
  --subnet=172.20.0.0/16 \
  --gateway=172.20.0.1 \
  my-custom-net

# Run container with static IP
podman run -d \
  --name web \
  --network my-custom-net \
  --ip 172.20.0.10 \
  nginx:alpine

# Verify
podman inspect web --format='{{.NetworkSettings.Networks.my_custom_net.IPAddress}}'
```

> **Resources**
>
> * [Podman Network Documentation](https://docs.podman.io/en/latest/markdown/podman-network.1.html)
> * [Container Networking Guide](https://docs.docker.com/network/)
> * [Network Drivers](https://docs.docker.com/network/drivers/)
