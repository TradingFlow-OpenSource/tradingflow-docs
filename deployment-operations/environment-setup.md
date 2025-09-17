# Environment Setup

Complete guide for setting up TradingFlow development and production environments.

## Development Environment

### Prerequisites

**System Requirements:**
- macOS 12.0+, Ubuntu 20.04+, or Windows 10+
- 8GB RAM minimum, 16GB recommended
- 50GB free disk space
- Docker Desktop 4.0+
- Node.js 18+ and npm
- Python 3.9+

### Quick Setup

```bash
# Clone repository
git clone https://github.com/tradingflow/tradingflow-platform.git
cd tradingflow-platform

# Setup development environment
make setup-dev

# Start development services
make dev-up

# Run tests
make test
```

### Manual Setup

#### 1. Install Dependencies

```bash
# macOS with Homebrew
brew install node python docker docker-compose redis postgresql

# Ubuntu/Debian
sudo apt update
sudo apt install nodejs npm python3 python3-pip docker.io docker-compose redis-server postgresql

# Windows with Chocolatey
choco install nodejs python docker-desktop redis-64 postgresql
```

#### 2. Database Setup

```bash
# Start PostgreSQL
sudo systemctl start postgresql  # Linux
brew services start postgresql   # macOS

# Create database
createdb tradingflow_dev

# Create user
createuser tradingflow_user -P
# Enter password when prompted
```

#### 3. Environment Configuration

```bash
# Copy environment template
cp .env.example .env

# Edit configuration
nano .env
```

**.env configuration:**
```bash
# Database
DATABASE_URL=postgresql://tradingflow_user:password@localhost:5432/tradingflow_dev

# Redis
REDIS_URL=redis://localhost:6379

# Application
NODE_ENV=development
PORT=3000
JWT_SECRET=your-jwt-secret-key

# External APIs
INFURA_PROJECT_ID=your-infura-project-id
ALCHEMY_API_KEY=your-alchemy-api-key
```

#### 4. Install Application Dependencies

```bash
# Frontend dependencies
cd frontend
npm install

# Backend dependencies
cd ../backend
pip install -r requirements.txt

# Node dependencies
cd ../node
npm install
```

#### 5. Database Migration

```bash
# Run migrations
cd backend
alembic upgrade head

# Seed initial data (optional)
python scripts/seed_data.py
```

## Production Environment

### Infrastructure Setup

#### Docker Compose Production

```yaml
version: '3.8'

services:
  tradingflow-app:
    image: tradingflow/app:latest
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    ports:
      - "80:3000"
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

  postgres:
    image: postgres:14-alpine
    environment:
      - POSTGRES_DB=tradingflow
      - POSTGRES_USER=tradingflow
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

#### Environment Variables

```bash
# Production .env
NODE_ENV=production
DATABASE_URL=postgresql://tradingflow:secure_password@postgres:5432/tradingflow
REDIS_URL=redis://redis:6379
JWT_SECRET=very-secure-jwt-secret-key-here
ENCRYPTION_KEY=256-bit-encryption-key-here

# API Keys
INFURA_PROJECT_ID=production-infura-key
ALCHEMY_API_KEY=production-alchemy-key
ETHERSCAN_API_KEY=production-etherscan-key

# Email/SMS (for notifications)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=alerts@tradingflow.ai
SMTP_PASS=app-specific-password

# Monitoring
SENTRY_DSN=your-sentry-dsn
DATADOG_API_KEY=your-datadog-key
```

### Cloud Deployment

#### AWS Deployment

```bash
# Install AWS CLI
pip install awscli

# Configure AWS
aws configure

# Deploy with CloudFormation
aws cloudformation deploy \
  --template-file infrastructure/aws/cloudformation.yaml \
  --stack-name tradingflow-production \
  --parameter-overrides \
    Environment=production \
    DatabasePassword=${DB_PASSWORD} \
    DomainName=app.tradingflow.ai
```

#### Docker Swarm

```bash
# Initialize swarm
docker swarm init

# Deploy stack
docker stack deploy -c docker-compose.prod.yml tradingflow

# Check services
docker service ls
```

#### Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tradingflow-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tradingflow
  template:
    metadata:
      labels:
        app: tradingflow
    spec:
      containers:
      - name: tradingflow
        image: tradingflow/app:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: database_url
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

## Monitoring Setup

### Application Monitoring

```bash
# Install monitoring stack
docker-compose -f monitoring/docker-compose.yml up -d

# Access Grafana
open http://localhost:3001  # admin/admin
```

### Log Aggregation

```bash
# Configure centralized logging
docker-compose -f logging/docker-compose.yml up -d

# View logs
docker-compose logs -f app
```

## Backup Strategy

### Automated Backups

```bash
# Database backup script
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/opt/tradingflow/backups"

# Create backup directory
mkdir -p $BACKUP_DIR

# Database backup
docker exec tradingflow_postgres_1 pg_dump -U tradingflow tradingflow > $BACKUP_DIR/db_$DATE.sql

# Compress backup
gzip $BACKUP_DIR/db_$DATE.sql

# Upload to cloud storage (example with AWS S3)
aws s3 cp $BACKUP_DIR/db_$DATE.sql.gz s3://tradingflow-backups/database/

# Clean old backups (keep last 30 days)
find $BACKUP_DIR -name "db_*.sql.gz" -mtime +30 -delete
```

### Disaster Recovery

```bash
# Recovery script
#!/bin/bash
BACKUP_FILE=$1

# Stop application
docker-compose down

# Restore database
gunzip -c $BACKUP_FILE | docker exec -i tradingflow_postgres_1 psql -U tradingflow tradingflow

# Start application
docker-compose up -d

# Verify restoration
docker-compose logs app
```

## Security Hardening

### Network Security

```bash
# Configure firewall
sudo ufw enable
sudo ufw allow 22/tcp      # SSH
sudo ufw allow 80/tcp      # HTTP
sudo ufw allow 443/tcp     # HTTPS
sudo ufw allow 6379/tcp    # Redis (internal only)

# SSL/TLS configuration
certbot --nginx -d app.tradingflow.ai
```

### Application Security

```bash
# Security scanning
npm audit fix
pip audit
docker scan tradingflow/app:latest

# Secrets management
docker secret create db_password /path/to/db_password.txt
docker secret create jwt_secret /path/to/jwt_secret.txt
```

## Performance Tuning

### Database Optimization

```sql
-- Create indexes for better performance
CREATE INDEX CONCURRENTLY idx_trades_timestamp ON trades (timestamp DESC);
CREATE INDEX CONCURRENTLY idx_trades_strategy_id ON trades (strategy_id);
CREATE INDEX CONCURRENTLY idx_portfolio_timestamp ON portfolio_history (timestamp DESC, vault_id);

-- Partition large tables
CREATE TABLE trades_y2024m01 PARTITION OF trades
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
```

### Application Scaling

```yaml
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: tradingflow-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: tradingflow-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

Environment setup ensures your TradingFlow deployment is secure, performant, and maintainable. Follow these guides carefully for production deployments.

## Next Steps
- [Going Live →](going-live.md) - Production deployment checklist
- [Maintenance & Updates →](maintenance-updates.md) - Ongoing operations
- [Monitoring & Analytics →](../advanced-features/monitoring-analytics.md) - System monitoring
