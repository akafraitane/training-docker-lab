# E-commerce Multi-Container Stack

Complete e-commerce backend stack demonstrating:
- Network isolation
- Service discovery
- Health checks
- Persistent storage
- Multi-tier architecture

## Architecture

```
┌─────────────────────────────────────────┐
│         Frontend Network                │
│  ┌──────────┐      ┌──────────┐        │
│  │   Web    │─────▶│   API    │        │
│  │  (nginx) │      │ (Flask)  │        │
│  └──────────┘      └──────────┘        │
│      :8080              │               │
└─────────────────────────┼───────────────┘
                          │
┌─────────────────────────┼───────────────┐
│         Backend Network │ (Isolated)    │
│                    ┌────┴────┐          │
│                    │   API   │          │
│                    └─┬─────┬─┘          │
│                      │     │            │
│              ┌───────┘     └──────┐     │
│              ▼                    ▼     │
│         ┌────────┐          ┌────────┐  │
│         │Postgres│          │ Redis  │  │
│         │   DB   │          │ Cache  │  │
│         └────────┘          └────────┘  │
└─────────────────────────────────────────┘
```

## Services

- **Web (nginx)**: Frontend web server, reverse proxy
- **API (Flask)**: Backend REST API
- **Postgres**: Relational database
- **Redis**: Caching layer

## Quick Start

```bash
# Start all services
cd sample-app/ecommerce-stack
podman-compose up -d

# Or with Docker Compose
docker-compose up -d

# Check status
podman-compose ps

# View logs
podman-compose logs -f

# Stop all services
podman-compose down

# Stop and remove volumes
podman-compose down -v
```

## Testing

```bash
# Test web frontend
curl http://localhost:8080

# Check API health
curl http://localhost:8080/api/health

# Test database connection
podman exec ecommerce-db psql -U ecommerce -c "\l"

# Test Redis
podman exec ecommerce-cache redis-cli ping
```

## Network Isolation

- **Frontend network**: Web and API
- **Backend network** (internal): API, Postgres, Redis
- Web cannot access database directly ✅ Security!

## Monitoring

```bash
# View resource usage
podman stats

# Check health status
podman inspect ecommerce-api --format='{{.State.Health.Status}}'
podman inspect ecommerce-db --format='{{.State.Health.Status}}'
podman inspect ecommerce-cache --format='{{.State.Health.Status}}'
```

## Development

```bash
# Rebuild specific service
podman-compose build api

# Restart service
podman-compose restart api

# View service logs
podman-compose logs api
```
