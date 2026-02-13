# Flask API Sample Application

Simple Flask REST API demonstrating containerization best practices.

## Features

- Multi-stage Docker build
- Non-root user
- Health checks
- Production-ready with Gunicorn

## Build and Run

```bash
# Build image
podman build -t flask-api:latest .

# Run container
podman run -d \
  --name my-api \
  -p 8000:8000 \
  -e ENVIRONMENT=production \
  flask-api:latest

# Test endpoints
curl http://localhost:8000/
curl http://localhost:8000/health
curl http://localhost:8000/api/data

# View logs
podman logs -f my-api

# Stop and remove
podman rm -f my-api
```

## Production Deployment

```bash
podman run -d \
  --name flask-api \
  --memory=256m \
  --cpus=0.5 \
  --read-only \
  --tmpfs /tmp \
  --cap-drop=ALL \
  --security-opt=no-new-privileges \
  -p 127.0.0.1:8000:8000 \
  --restart=unless-stopped \
  flask-api:latest
```
