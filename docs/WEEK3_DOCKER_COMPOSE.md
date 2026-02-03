# Week 3: Docker Compose - Multi-Container Orchestration

**Goal:** Run a 3-service architecture with Docker Compose

---

## Quick Start

```bash
# 1. Copy environment file
cp .env.example .env

# 2. Start all services
docker compose up --build

# 3. Open http://localhost
```

---

## Architecture

```
Browser → Nginx (port 80) → Next.js App (port 3000) → MongoDB (port 27017)
            │                      │                        │
         frontend-net           both nets              backend-net
```

### Services

| Service | Image | Port | Network | Purpose |
|---------|-------|------|---------|---------|
| nginx | Custom (Alpine) | 80 (exposed) | frontend-net | Reverse proxy, security headers |
| app | Custom (Node 20) | 3000 (internal) | frontend + backend | Next.js application |
| db | mongo:7 | 27017 (exposed*) | backend-net | MongoDB database |

*Exposed for development only. Remove in production.

### Network Isolation

- **frontend-net**: nginx ↔ app communication
- **backend-net**: app ↔ db communication
- nginx cannot reach db directly (security!)

---

## Common Commands

```bash
# Start all services
docker compose up --build

# Start in background
docker compose up -d

# View logs
docker compose logs -f

# View specific service logs
docker compose logs -f app

# Check status
docker compose ps

# Stop all services
docker compose down

# Stop and remove volumes (WARNING: deletes data!)
docker compose down -v

# Restart a service
docker compose restart app

# Shell into a container
docker compose exec app sh
```

---

## Health Checks

```bash
# Nginx health
curl http://localhost/health

# App health (via nginx)
curl http://localhost/api/health

# MongoDB health
docker compose exec db mongosh --eval "db.adminCommand('ping')"
```

---

## Seed Demo Data

```bash
# Copy seed script and run
docker compose cp scripts/seed-db.js db:/tmp/seed-db.js
docker compose exec db mongosh team-flags-edu /tmp/seed-db.js
```

---

## Key Files

| File | Purpose |
|------|---------|
| `docker-compose.yml` | Orchestrates all 3 services |
| `Dockerfile` | Multi-stage build for Next.js app |
| `nginx/nginx.conf` | Reverse proxy configuration |
| `nginx/Dockerfile` | Nginx container build |
| `scripts/mongo-init.js` | Database initialization |
| `scripts/seed-db.js` | Demo data seeder |
| `.env.example` | Environment template |

---

## Learning Objectives

After completing this lab, you should understand:

- [ ] What Docker Compose does and why it's useful
- [ ] How to define multiple services in `docker-compose.yml`
- [ ] How Docker networks provide isolation
- [ ] How `depends_on` with health checks ensures startup order
- [ ] How volumes provide persistent storage
- [ ] How a reverse proxy (nginx) works

---

## Next Steps

- **Week 4:** CI/CD Pipeline with GitHub Actions
- **Week 5-6:** Security scanning with Trivy, Firebase authentication

---

## Resources

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Nginx Reverse Proxy Guide](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
- [MongoDB Docker Hub](https://hub.docker.com/_/mongo)
