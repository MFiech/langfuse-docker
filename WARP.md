# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This is a **Langfuse Docker Setup** repository that provides a complete Docker Compose environment for running Langfuse locally. Langfuse is an open-source LLM observability platform that offers detailed traces, spans, generations tracking, prompt management, and cost monitoring for AI applications.

This setup is designed as a companion observability platform for the [Podcast Summarizer](https://github.com/MFiech/podcast-analyzer) project.

## Architecture

### Docker Services Stack
The project runs a multi-container Docker environment with the following services:

- **langfuse-web** (port 4000): Main Langfuse web UI and API server
- **langfuse-worker**: Background processing for traces, evaluations, and exports  
- **postgres** (port 5432): Primary database for application data
- **clickhouse** (ports 8123, 9003): Analytics database for time-series data
- **redis** (port 6379): Cache and session storage with authentication
- **minio** (ports 9001, 9002): S3-compatible object storage

### Data Persistence
All service data is persisted in the `./data/` directory:
- `data/postgres/` - PostgreSQL data
- `data/clickhouse/` - ClickHouse analytics data
- `data/redis/` - Redis cache data
- `data/minio/` - MinIO object storage

### Key Configuration Files
- `docker-compose.yml`: Complete service orchestration with health checks and resource limits
- `.env.example`: Template for environment configuration
- `.env`: Local environment variables (not tracked in git)

## Common Commands

### Service Management
```bash
# Start all services
docker-compose up -d

# Check service status
docker-compose ps

# View all logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f langfuse-web
docker-compose logs -f langfuse-worker

# Stop all services
docker-compose down

# Stop and remove all data (⚠️ destructive)
docker-compose down -v

# Restart specific service
docker-compose restart langfuse-web
```

### Updates and Maintenance
```bash
# Pull latest images
docker-compose pull

# Restart with new images
docker-compose up -d

# Check image versions
docker-compose images
```

### Database Access
```bash
# PostgreSQL console
docker-compose exec postgres psql -U langfuse -d langfuse

# ClickHouse console
docker-compose exec clickhouse clickhouse-client

# Redis CLI
docker-compose exec redis redis-cli
```

### Health Monitoring
```bash
# Check PostgreSQL health
docker-compose exec postgres pg_isready -U langfuse

# Check port usage
lsof -i :4000  # Langfuse web
lsof -i :5432  # PostgreSQL
lsof -i :8123  # ClickHouse
```

## Environment Configuration

### Development Setup
The `.env` file contains all necessary configuration for local development. Key variables include:

- **NEXTAUTH_SECRET**: Authentication secret (change in production)
- **LANGFUSE_ENCRYPTION_KEY**: Data encryption key (must be 32 characters)
- **Database URLs**: Pre-configured for Docker Compose networking
- **S3/MinIO**: Object storage configuration with default credentials
- **ClickHouse**: Analytics database settings

### Production Considerations
For production deployment, ensure you change:
- All default passwords and secrets
- Enable SSL/TLS encryption
- Configure external databases if needed
- Use proper secrets management
- Enable audit logging

## Service Access Points

- **Langfuse Web UI**: http://localhost:4000
- **MinIO Console**: http://localhost:9001 (credentials: minioadmin/minioadmin)
- **PostgreSQL**: localhost:5432 (langfuse/langfuse)
- **ClickHouse HTTP**: localhost:8123
- **Redis**: localhost:6379 (password: langfuse123)

## Integration

### API Key Setup
1. Access http://localhost:4000
2. Navigate to Settings → API Keys
3. Generate Secret + Public key pair
4. Use in applications:
   ```bash
   LANGFUSE_SECRET_KEY=sk-lf-your-secret-key
   LANGFUSE_PUBLIC_KEY=pk-lf-your-public-key
   LANGFUSE_HOST=http://localhost:4000
   ```

### Resource Limits
Services are configured with CPU and memory limits:
- **langfuse-web**: 1.5 CPU, 2GB RAM
- **langfuse-worker**: 1.0 CPU, 1.5GB RAM
- **postgres**: 1.0 CPU, 2GB RAM
- **clickhouse**: 1.5 CPU, 3GB RAM
- **redis/minio**: 0.5 CPU, 1GB RAM each

## Troubleshooting

### Common Issues
- **Worker crashes**: Fixed with official `langfuse-worker:3` image
- **Port conflicts**: Stop local PostgreSQL with `brew services stop postgresql`
- **Memory issues**: Ensure 8GB+ RAM available for optimal performance
- **Database connection**: Check health with `docker-compose exec postgres pg_isready -U langfuse`

### Health Checks
All services include health checks with retry logic and start periods. Services won't start dependents until health checks pass.

## Project Context

This repository serves as the observability infrastructure for LLM applications, particularly the Podcast Summarizer project. It provides comprehensive monitoring, prompt management, and performance tracking for AI workflows involving podcast analysis and summarization.