# 📊 Langfuse Docker Setup

A complete Docker Compose setup for running **Langfuse** locally for LLM observability, prompt management, and session tracking. This is the companion observability platform for the [Podcast Summarizer](https://github.com/MFiech/podcast-analyzer) project.

## 🎯 What is Langfuse?

Langfuse is an open-source LLM engineering platform that provides:
- **📈 Observability**: Detailed traces, spans, and generations for LLM applications
- **📋 Prompt Management**: Centralized, versioned prompt templates with A/B testing
- **🔗 Sessions**: Group related LLM interactions together
- **💰 Cost Tracking**: Monitor token usage and API costs
- **🧪 Evaluations**: Assess and improve LLM performance

## ✨ Features

### 🔧 What's Included
- **🐳 Complete Docker Setup**: All services containerized and ready to run
- **🗄️ Full Database Stack**: PostgreSQL + ClickHouse + Redis + MinIO
- **⚡ Background Processing**: Worker container for async operations
- **🔒 Security**: Proper environment variable management
- **📊 Monitoring**: Built-in health checks and status monitoring

### 🚀 Components
- **Langfuse Web UI**: Main dashboard at `http://localhost:4000`
- **Langfuse Worker**: Background task processing
- **PostgreSQL**: Primary database
- **ClickHouse**: Analytics database  
- **Redis**: Cache and session store
- **MinIO**: S3-compatible object storage

## 🚀 Quick Start

### Prerequisites
- Docker Desktop
- Docker Compose v2+
- 8GB+ RAM (recommended)
- Ports 4000, 5432, 8123, 6379, 9000 available

### 1. Clone Repository

```bash
git clone git@github.com:MFiech/langfuse-docker.git
cd langfuse-docker
```

### 2. Configuration

The `.env` file is pre-configured for local development:

```bash
# NextAuth Configuration
NEXTAUTH_URL=http://localhost:4000
NEXTAUTH_SECRET=langfuse-dev-secret-key-2024-change-in-production

# Langfuse Encryption
LANGFUSE_ENCRYPTION_KEY=langfuse-dev-encryption-key-2024-change-in-production

# Database URLs
DATABASE_URL=postgresql://langfuse:langfuse@postgres:5432/langfuse
CLICKHOUSE_URL=http://clickhouse:8123
REDIS_URL=redis://redis:6379

# S3/MinIO Configuration
S3_ACCESS_KEY_ID=langfuse
S3_SECRET_ACCESS_KEY=langfuse123
S3_BUCKET_NAME=langfuse
S3_ENDPOINT=http://minio:9000
S3_REGION=us-east-1
S3_FORCE_PATH_STYLE=true
```

**⚠️ Important**: Change all default passwords and keys in production!

### 3. Start Services

```bash
# Start all services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f langfuse-server
```

### 4. Access Langfuse

Open your browser to **http://localhost:4000**

- Create your first account
- Generate API keys (Secret + Public key pair)
- Start tracing your LLM applications!

### 5. Stop Services

```bash
# Stop all services
docker-compose down

# Stop and remove data volumes (⚠️ deletes all data)
docker-compose down -v
```

## 📋 Service Details

### 🌐 Langfuse Server
- **URL**: http://localhost:4000
- **Function**: Main web interface and API
- **Container**: `langfuse-server`

### ⚙️ Langfuse Worker  
- **Function**: Background processing (traces, evaluations, exports)
- **Container**: `langfuse-worker`
- **Status**: Fixed worker implementation (no more crashes!)

### 🗄️ PostgreSQL
- **Port**: 5432
- **Database**: `langfuse`
- **User**: `langfuse` / `langfuse`
- **Container**: `postgres`

### 📊 ClickHouse
- **Port**: 8123 (HTTP), 9000 (TCP)
- **Function**: Analytics and time-series data
- **Container**: `clickhouse`

### ⚡ Redis
- **Port**: 6379
- **Function**: Caching and session storage
- **Container**: `redis`

### 💾 MinIO (S3)
- **Console**: http://localhost:9001
- **API**: http://localhost:9000
- **Credentials**: `minioadmin` / `minioadmin`
- **Container**: `minio`

## 🔧 Troubleshooting

### Common Issues

#### Worker Container Keeps Crashing
✅ **Fixed!** The worker now uses the official `langfuse-worker:3` image instead of trying to run custom commands.

#### Port Conflicts
```bash
# Check what's using your ports
lsof -i :4000  # Langfuse
lsof -i :5432  # PostgreSQL
lsof -i :8123  # ClickHouse

# Stop conflicting services
brew services stop postgresql  # If you have local PostgreSQL
```

#### Database Connection Issues
```bash
# Check if PostgreSQL is ready
docker-compose exec postgres pg_isready -U langfuse

# View database logs
docker-compose logs postgres
```

#### Worker Not Processing
```bash
# Check worker logs
docker-compose logs langfuse-worker

# Restart worker
docker-compose restart langfuse-worker
```

### Useful Commands

```bash
# View all logs
docker-compose logs -f

# Check service health
docker-compose ps

# Restart specific service
docker-compose restart langfuse-server

# Access PostgreSQL console
docker-compose exec postgres psql -U langfuse -d langfuse

# Access ClickHouse console
docker-compose exec clickhouse clickhouse-client

# View Redis data
docker-compose exec redis redis-cli
```

## 🔄 Updating Langfuse

```bash
# Pull latest images
docker-compose pull

# Restart with new images
docker-compose up -d

# Check new versions
docker-compose images
```

## 📊 Integration with Applications

### API Keys Setup

1. Go to http://localhost:4000
2. Navigate to Settings → API Keys
3. Generate a new key pair
4. Use in your applications:

```bash
LANGFUSE_SECRET_KEY=sk-lf-your-secret-key
LANGFUSE_PUBLIC_KEY=pk-lf-your-public-key
LANGFUSE_HOST=http://localhost:4000
```

### Python Integration Example

```python
from langfuse import Langfuse

langfuse = Langfuse(
    secret_key="sk-lf-your-secret-key",
    public_key="pk-lf-your-public-key", 
    host="http://localhost:4000"
)

# Your LLM application code here
```

## 🎯 Perfect for

- **Local Development**: Full-featured Langfuse instance
- **Testing**: Experiment with prompts and sessions
- **Learning**: Understand LLM observability concepts
- **Prototyping**: Build and test LLM applications with rich observability

## 📈 What You'll See

### Sessions Dashboard
- Grouped LLM interactions (e.g., podcast analysis workflows)
- User journey tracking and analytics
- Performance metrics across related traces

### Prompts Dashboard  
- Centralized prompt templates with variables
- Version management and A/B testing
- Collaboration and prompt optimization

### Traces Dashboard
- Detailed breakdown of LLM application flows
- Input/output inspection at every step
- Performance and cost analysis

### Generations Dashboard
- Individual LLM API call monitoring  
- Token usage and cost tracking
- Quality assessment and debugging

## 🔒 Security Notes

### Development vs Production

**Development (Current Setup)**:
- Default passwords for easy setup
- All services on localhost
- No SSL/TLS encryption
- Simplified authentication

**Production (Recommendations)**:
- Change all default passwords
- Use proper SSL certificates
- Configure external databases
- Set up proper authentication
- Use secrets management
- Enable audit logging

### Environment Variables to Change

```bash
# Change these in production!
NEXTAUTH_SECRET=your-strong-secret-here
LANGFUSE_ENCRYPTION_KEY=your-encryption-key-here
S3_SECRET_ACCESS_KEY=your-s3-secret-here

# Database passwords
POSTGRES_PASSWORD=your-db-password
CLICKHOUSE_PASSWORD=your-clickhouse-password
```

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📝 License

This Docker setup is provided under the MIT License. Langfuse itself is licensed under the MIT License.

## 🙏 Acknowledgments

- **Langfuse Team** for the excellent observability platform
- **Docker Community** for containerization tools
- **Open Source Contributors** for the supporting technologies

## 📞 Support

- 🐛 [Report Issues](https://github.com/MFiech/langfuse-docker/issues)
- 💬 [Discussions](https://github.com/MFiech/langfuse-docker/discussions)
- 📚 [Langfuse Documentation](https://langfuse.com/docs)
- 🔗 [Main Application](https://github.com/MFiech/podcast-analyzer)

---

**🎉 Ready to supercharge your LLM observability!**

Get detailed insights into your AI applications with sessions, prompts, traces, and cost tracking.
