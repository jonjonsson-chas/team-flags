# Team Flags EDU - DevSecOps Learning Platform

> **Educational version** of the Team Flags platform, designed for learning Docker, CI/CD, and DevSecOps practices.

A Next.js application showcasing modern DevSecOps workflows. This project is used by students at Chas Academy to learn containerization, security scanning, and deployment automation.

## What You'll Learn

- **Docker**: Multi-stage builds, security best practices, image optimization
- **Docker Compose**: Multi-container orchestration, networks, volumes
- **CI/CD**: Automated builds, testing, and deployments
- **Security**: Container scanning, vulnerability management, SBOM generation
- **Kubernetes**: Container orchestration, policies, monitoring
- **Observability**: Logging, metrics, alerting

---

## Quick Start with Docker Compose (Week 3+)

The fastest way to get the full application running locally.

### Prerequisites

- Docker Desktop installed and running
- Git

### Steps

```bash
# 1. Fork this repository on GitHub, then clone your fork
git clone https://github.com/YOUR-USERNAME/team-flags-edu.git
cd team-flags-edu

# 2. Copy environment file
cp .env.example .env

# 3. Start all services (nginx + app + mongodb)
docker compose up --build

# 4. Open http://localhost in your browser
```

That's it! You now have:
- **Nginx** reverse proxy on port 80
- **Next.js** app on port 3000 (internal)
- **MongoDB** database on port 27017

### Seed Demo Data (Optional)

```bash
# Run the seed script to populate with demo teams/students
docker compose exec db mongosh team-flags-edu /docker-entrypoint-initdb.d/../tmp/seed.js

# Or copy and run:
docker compose cp scripts/seed-db.js db:/tmp/seed-db.js
docker compose exec db mongosh team-flags-edu /tmp/seed-db.js
```

---

## Architecture (Week 3)

```
                         ┌─────────────────┐
                         │    Browser      │
                         │  localhost:80   │
                         └────────┬────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │         Nginx             │
                    │    (Reverse Proxy)        │
                    │  - Security headers       │
                    │  - Static caching         │
                    │  - Health checks          │
                    └─────────────┬─────────────┘
                                  │ frontend-net
                    ┌─────────────▼─────────────┐
                    │       Next.js App         │
                    │      (Port 3000)          │
                    │  - React frontend         │
                    │  - API routes             │
                    │  - Health endpoint        │
                    └─────────────┬─────────────┘
                                  │ backend-net
                    ┌─────────────▼─────────────┐
                    │        MongoDB            │
                    │      (Port 27017)         │
                    │  - Persistent data        │
                    │  - Volume mounted         │
                    └───────────────────────────┘
```

### Service Communication

| Service | Port | Network | Description |
|---------|------|---------|-------------|
| nginx | 80 (exposed) | frontend-net | Entry point, reverse proxy |
| app | 3000 (internal) | frontend-net, backend-net | Next.js application |
| db | 27017 (exposed*) | backend-net | MongoDB database |

*MongoDB port exposed for development/debugging. Remove in production.

---

## Common Commands

### Docker Compose

```bash
# Start all services
docker compose up

# Start with rebuild
docker compose up --build

# Start in background
docker compose up -d

# View logs
docker compose logs -f

# View specific service logs
docker compose logs -f app

# Stop all services
docker compose down

# Stop and remove volumes (WARNING: deletes data!)
docker compose down -v

# Check service status
docker compose ps

# Restart a specific service
docker compose restart app
```

### Health Checks

```bash
# Check nginx health
curl http://localhost/health

# Check app health directly
docker compose exec app wget -qO- http://localhost:3000/api/health

# Check MongoDB
docker compose exec db mongosh --eval "db.adminCommand('ping')"
```

### Database

```bash
# Connect to MongoDB shell
docker compose exec db mongosh team-flags-edu

# Run seed script
docker compose cp scripts/seed-db.js db:/tmp/seed-db.js
docker compose exec db mongosh team-flags-edu /tmp/seed-db.js

# Backup database
docker compose exec db mongodump --db team-flags-edu --out /tmp/backup

# View collections
docker compose exec db mongosh team-flags-edu --eval "db.getCollectionNames()"
```

---

## Project Structure

```
team-flags-edu/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   └── health/        # Health check endpoint
│   ├── components/        # React components
│   └── page.tsx           # Main page
├── lib/                   # Utility libraries
│   ├── mongodb.ts         # Database connection
│   ├── firebase/          # Firebase configuration (optional)
│   └── types.ts           # TypeScript types
├── nginx/                 # Nginx configuration (Week 3)
│   ├── Dockerfile         # Nginx container
│   └── nginx.conf         # Reverse proxy config
├── scripts/               # Utility scripts
│   ├── mongo-init.js      # MongoDB initialization
│   └── seed-db.js         # Demo data seeder
├── public/                # Static assets
├── docker-compose.yml     # Multi-container orchestration
├── Dockerfile             # Next.js container
├── .env.example           # Environment template
└── README.md             # This file
```

---

## Environment Variables

Copy `.env.example` to `.env` and configure:

| Variable | Description | Default |
|----------|-------------|---------|
| `MONGODB_URI` | MongoDB connection string | `mongodb://admin:password@db:27017/...` |
| `MONGODB_DB` | Database name | `team-flags-edu` |
| `NGINX_PORT` | External nginx port | `80` |
| `MONGO_PORT` | External MongoDB port | `27017` |

### Using MongoDB Atlas (Cloud)

To use MongoDB Atlas instead of local container:

1. Create free cluster at [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas/register)
2. Get connection string
3. Update `.env`:
   ```
   MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/team-flags-edu
   ```
4. Comment out `db` service in `docker-compose.yml`

---

## Learning Path

### Week 2: Container Basics
- [x] Fork this repository
- [x] Write Dockerfile (use ours as reference)
- [x] Build and run a single container
- [x] Understand multi-stage builds

### Week 3: Docker Compose
- [ ] Run with `docker compose up`
- [ ] Understand 3-service architecture
- [ ] Explore networks and volumes
- [ ] Configure health checks
- [ ] Customize for your team

### Week 4: CI/CD Pipeline
- [ ] Set up GitHub Actions
- [ ] Automate Docker builds
- [ ] Add linting and testing
- [ ] Push to container registry

### Week 5-6: Security Integration
- [ ] Add Trivy container scanning
- [ ] Implement dependency checks
- [ ] Generate SBOM
- [ ] Configure security gates

### Week 7-8: Kubernetes Deployment
- [ ] Create Kubernetes manifests
- [ ] Deploy to local cluster
- [ ] Implement OPA Gatekeeper policies
- [ ] Set up runtime security (Falco)

### Week 9-10: SRE & Operations
- [ ] Define SLIs/SLOs
- [ ] Set up monitoring and alerting
- [ ] Write runbooks
- [ ] Practice incident response

---

## Troubleshooting

### Port already in use

```bash
# Change ports in .env
NGINX_PORT=8080
MONGO_PORT=27018
```

### Container won't start

```bash
# Check logs
docker compose logs app

# Rebuild from scratch
docker compose down -v
docker compose up --build
```

### MongoDB connection refused

The app waits for MongoDB to be healthy. If it keeps failing:

```bash
# Check MongoDB logs
docker compose logs db

# Verify MongoDB is running
docker compose exec db mongosh --eval "db.adminCommand('ping')"
```

### Changes not reflected

```bash
# Force rebuild
docker compose up --build --force-recreate
```

---

## Docker Best Practices Demonstrated

1. **Multi-stage builds** - Separate build and runtime stages
2. **Minimal base images** - Using Alpine Linux for smaller size
3. **Non-root user** - Running as `nextjs` user for security
4. **Health checks** - Both in Dockerfile and docker-compose
5. **Network isolation** - Separate frontend and backend networks
6. **Named volumes** - Persistent MongoDB data
7. **Environment variables** - Configuration via .env file

---

## Security Features

- Non-root container users
- Minimal attack surface (Alpine base)
- No secrets in images (environment variables only)
- Network isolation between services
- Security headers in nginx
- Health checks for reliability

---

## License

MIT License - see [LICENSE](LICENSE) file

Copyright (c) 2026 Retro 87

## Credits

Created for [Chas Academy](https://chasacademy.se/) - IT- och Cybersäkerhetstekniker program

---

## Resources

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Next.js Documentation](https://nextjs.org/docs)
- [MongoDB Documentation](https://docs.mongodb.com/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [OWASP Docker Security](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

---

Happy learning!
