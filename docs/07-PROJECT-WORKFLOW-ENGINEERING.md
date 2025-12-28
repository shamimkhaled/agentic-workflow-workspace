# FlowSpark AI - Project Workflow & Engineering Process
## DevOps, CI/CD Pipeline & Environment Management

---

## TABLE OF CONTENTS
1. [Development Workflow](#development-workflow)
2. [Environment Strategy](#environment-strategy)
3. [CI/CD Pipeline](#cicd-pipeline)
4. [Secrets Management](#secrets-management)
5. [Deployment Process](#deployment-process)
6. [Monitoring & Logging](#monitoring--logging)

---

## 1. DEVELOPMENT WORKFLOW

### 1.1 Git Branching Strategy

```
main (production)
  │
  ├── staging (pre-production)
  │     │
  │     ├── develop (integration)
  │     │     │
  │     │     ├── feature/doc-generation-enhancement
  │     │     ├── feature/rag-improvements
  │     │     ├── bugfix/issue-123-token-calculation
  │     │     └── hotfix/critical-auth-bug
```

#### Branch Types

| Branch Type | Naming Convention | Purpose | Merges To |
|-------------|-------------------|---------|-----------|
| **main** | `main` | Production-ready code | - |
| **staging** | `staging` | Pre-production testing | `main` |
| **develop** | `develop` | Integration branch | `staging` |
| **feature** | `feature/description` | New features | `develop` |
| **bugfix** | `bugfix/issue-id-description` | Bug fixes | `develop` |
| **hotfix** | `hotfix/description` | Critical production fixes | `main` + `develop` |
| **release** | `release/v1.2.0` | Release preparation | `main` + `develop` |

### 1.2 Commit Message Convention

Follow **Conventional Commits**:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting)
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

**Examples**:
```
feat(documents): add PDF export functionality

fix(auth): resolve JWT token expiration issue

docs(api): update authentication endpoint documentation

refactor(kb): improve chunk retrieval performance
```

### 1.3 Pull Request Process

#### PR Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Checklist
- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No new warnings
- [ ] Tested locally

## Related Issues
Closes #123

## Screenshots (if applicable)
```

#### Code Review Guidelines

**Reviewers should check**:
1. ✅ Code follows Django/React best practices
2. ✅ Tests cover new functionality
3. ✅ No security vulnerabilities
4. ✅ Performance considerations
5. ✅ Documentation updated
6. ✅ No hardcoded credentials
7. ✅ Tenant isolation maintained

**Response Time SLA**:
- Initial review: Within 24 hours
- Follow-up reviews: Within 4 hours

---

## 2. ENVIRONMENT STRATEGY

### 2.1 Environment Overview

```
┌─────────────────────────────────────────────────────────┐
│                    ENVIRONMENTS                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  LOCAL DEVELOPMENT                                      │
│  - Docker Compose                                       │
│  - Local databases (PostgreSQL, Redis, ChromaDB)       │
│  - Mock external services                              │
│  - Debug mode enabled                                  │
│                                                         │
│           ↓ (Push to develop)                           │
│                                                         │
│  DEVELOPMENT/CI                                         │
│  - Automated tests                                      │
│  - Code quality checks                                 │
│  - Ephemeral test databases                            │
│                                                         │
│           ↓ (Merge to staging)                          │
│                                                         │
│  STAGING                                                │
│  - Production-like environment                          │
│  - Real services (but isolated)                        │
│  - QA testing                                          │
│  - Load testing                                        │
│                                                         │
│           ↓ (Merge to main)                             │
│                                                         │
│  PRODUCTION                                             │
│  - Live customer data                                  │
│  - High availability                                   │
│  - Auto-scaling                                        │
│  - Full monitoring                                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 2.2 Environment Configuration

#### Local Development

```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: flowspark_dev
      POSTGRES_USER: flowspark
      POSTGRES_PASSWORD: dev_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  chromadb:
    image: chromadb/chroma:latest
    ports:
      - "8000:8000"
    volumes:
      - chroma_data:/chroma/chroma

  backend:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
      - chromadb
    environment:
      - DEBUG=True
      - DATABASE_URL=postgresql://flowspark:dev_password@db:5432/flowspark_dev
      - REDIS_URL=redis://redis:6379/0
      - CHROMA_URL=http://chromadb:8000

  celery_worker:
    build: .
    command: celery -A config worker -l info
    volumes:
      - .:/app
    depends_on:
      - db
      - redis
    environment:
      - DATABASE_URL=postgresql://flowspark:dev_password@db:5432/flowspark_dev
      - REDIS_URL=redis://redis:6379/0

  frontend:
    build: ./frontend
    command: npm run dev
    volumes:
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:8000/api/v1

volumes:
  postgres_data:
  chroma_data:
```

#### Environment Variables

**.env.example**:
```bash
# Django
DEBUG=False
SECRET_KEY=your-secret-key-here
ALLOWED_HOSTS=localhost,127.0.0.1
DJANGO_SETTINGS_MODULE=config.settings.production

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/flowspark_prod
DATABASE_POOL_SIZE=20

# Redis
REDIS_URL=redis://localhost:6379/0

# Vector Database
CHROMA_URL=http://chromadb:8000
CHROMA_TENANT_ISOLATION=true

# Storage
S3_BUCKET_NAME=flowspark-storage
S3_REGION=us-east-1
S3_ACCESS_KEY=your-access-key
S3_SECRET_KEY=your-secret-key

# AI Services
OPENAI_API_KEY=sk-...
OPENAI_ORG_ID=org-...
ANTHROPIC_API_KEY=sk-ant-...
LEONARDO_API_KEY=...

# Ollama (Local LLM)
OLLAMA_BASE_URL=http://localhost:11434

# Stripe
STRIPE_SECRET_KEY=sk_test_...
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Email
EMAIL_BACKEND=django.core.mail.backends.smtp.EmailBackend
EMAIL_HOST=smtp.sendgrid.net
EMAIL_PORT=587
EMAIL_HOST_USER=apikey
EMAIL_HOST_PASSWORD=your-sendgrid-api-key
DEFAULT_FROM_EMAIL=noreply@flowspark.ai

# Monitoring
SENTRY_DSN=https://...@sentry.io/...

# Feature Flags
ENABLE_SSO=true
ENABLE_ANALYTICS=true
ENABLE_ADVANCED_RAG=false
```

---

## 3. CI/CD PIPELINE

### 3.1 GitHub Actions Workflow

**.github/workflows/ci.yml**:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ develop, staging, main ]
  pull_request:
    branches: [ develop, staging, main ]

env:
  PYTHON_VERSION: '3.11'
  NODE_VERSION: '18'

jobs:
  # ==========================================
  # BACKEND TESTS
  # ==========================================
  backend-tests:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: flowspark_test
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Cache Python dependencies
        uses: actions/cache@v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install -r requirements-dev.txt
      
      - name: Run migrations
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/flowspark_test
        run: |
          python manage.py migrate
      
      - name: Run tests with coverage
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/flowspark_test
          REDIS_URL: redis://localhost:6379/0
        run: |
          pytest --cov=apps --cov-report=xml --cov-report=html
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
          fail_ci_if_error: true

  # ==========================================
  # CODE QUALITY
  # ==========================================
  code-quality:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Install linting tools
        run: |
          pip install black flake8 pylint isort
      
      - name: Run Black
        run: black --check .
      
      - name: Run Flake8
        run: flake8 apps/ services/
      
      - name: Run Pylint
        run: pylint apps/ services/ --fail-under=8.0
      
      - name: Run isort
        run: isort --check-only .

  # ==========================================
  # SECURITY SCAN
  # ==========================================
  security-scan:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Bandit security scan
        run: |
          pip install bandit
          bandit -r apps/ services/ -f json -o bandit-report.json
      
      - name: Run Safety check
        run: |
          pip install safety
          safety check --json
      
      - name: Upload security reports
        uses: actions/upload-artifact@v3
        with:
          name: security-reports
          path: bandit-report.json

  # ==========================================
  # FRONTEND TESTS
  # ==========================================
  frontend-tests:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      
      - name: Install dependencies
        working-directory: ./frontend
        run: npm ci
      
      - name: Run ESLint
        working-directory: ./frontend
        run: npm run lint
      
      - name: Run tests
        working-directory: ./frontend
        run: npm run test:ci
      
      - name: Build
        working-directory: ./frontend
        run: npm run build

  # ==========================================
  # BUILD & PUSH DOCKER IMAGES
  # ==========================================
  build-and-push:
    runs-on: ubuntu-latest
    needs: [backend-tests, code-quality, frontend-tests]
    if: github.ref == 'refs/heads/staging' || github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: flowspark/backend
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
      
      - name: Build and push Backend
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Build and push Frontend
        uses: docker/build-push-action@v4
        with:
          context: ./frontend
          push: true
          tags: flowspark/frontend:${{ github.sha }}

  # ==========================================
  # DEPLOY TO STAGING
  # ==========================================
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build-and-push]
    if: github.ref == 'refs/heads/staging'
    environment: staging
    
    steps:
      - name: Deploy to staging
        run: |
          # Deploy via Kubernetes or cloud provider CLI
          echo "Deploying to staging environment"
          # kubectl set image deployment/backend backend=flowspark/backend:${{ github.sha }}
  
  # ==========================================
  # DEPLOY TO PRODUCTION
  # ==========================================
  deploy-production:
    runs-on: ubuntu-latest
    needs: [build-and-push]
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
      - name: Deploy to production
        run: |
          # Deploy via Kubernetes or cloud provider CLI
          echo "Deploying to production environment"
          # kubectl set image deployment/backend backend=flowspark/backend:${{ github.sha }}
```

---

## 4. SECRETS MANAGEMENT

### 4.1 Local Development

Use **`.env` files** (never commit these):

```bash
# .env (git-ignored)
SECRET_KEY=local-dev-secret-key
DATABASE_URL=postgresql://localhost/flowspark_dev
OPENAI_API_KEY=sk-...
```

### 4.2 Staging/Production

Use **environment-specific secrets**:

#### Option 1: GitHub Secrets (for CI/CD)
- Store in GitHub Repository Settings → Secrets
- Access via `${{ secrets.SECRET_NAME }}`

#### Option 2: AWS Secrets Manager / HashiCorp Vault
```python
# settings/production.py
import boto3
import json

def get_secret(secret_name):
    client = boto3.client('secretsmanager', region_name='us-east-1')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])

secrets = get_secret('flowspark-prod-secrets')
SECRET_KEY = secrets['SECRET_KEY']
DATABASE_URL = secrets['DATABASE_URL']
```

#### Option 3: Kubernetes Secrets
```yaml
# kubernetes/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: flowspark-secrets
type: Opaque
data:
  SECRET_KEY: <base64-encoded>
  DATABASE_URL: <base64-encoded>
```

---

## 5. DEPLOYMENT PROCESS

### 5.1 Kubernetes Deployment

#### Backend Deployment

```yaml
# kubernetes/backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flowspark-backend
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: flowspark/backend:latest
        ports:
        - containerPort: 8000
        env:
        - name: DJANGO_SETTINGS_MODULE
          value: "config.settings.production"
        - name: SECRET_KEY
          valueFrom:
            secretKeyRef:
              name: flowspark-secrets
              key: SECRET_KEY
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: flowspark-secrets
              key: DATABASE_URL
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health/
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health/
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: production
spec:
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer
```

#### Horizontal Pod Autoscaler

```yaml
# kubernetes/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: flowspark-backend
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### 5.2 Database Migrations

#### Zero-Downtime Migration Strategy

```bash
# 1. Deploy new code (with backward-compatible migrations)
kubectl apply -f kubernetes/backend-deployment.yaml

# 2. Run migrations (in a Job)
kubectl run migration-job --image=flowspark/backend:latest \
  --command -- python manage.py migrate

# 3. Verify migration success
kubectl logs migration-job

# 4. Clean up
kubectl delete pod migration-job
```

#### Migration Job (Kubernetes)

```yaml
# kubernetes/migration-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: migration-job
  namespace: production
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: flowspark/backend:latest
        command: ["python", "manage.py", "migrate"]
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: flowspark-secrets
              key: DATABASE_URL
      restartPolicy: OnFailure
  backoffLimit: 3
```

---

## 6. MONITORING & LOGGING

### 6.1 Application Monitoring

#### Health Check Endpoint

```python
# apps/core/views.py
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import AllowAny
from rest_framework.response import Response
from django.db import connection
import redis

@api_view(['GET'])
@permission_classes([AllowAny])
def health_check(request):
    """Health check endpoint for load balancers"""
    
    health = {
        'status': 'healthy',
        'database': False,
        'redis': False,
        'version': '1.0.0'
    }
    
    # Check database
    try:
        with connection.cursor() as cursor:
            cursor.execute("SELECT 1")
        health['database'] = True
    except Exception as e:
        health['status'] = 'unhealthy'
        health['database_error'] = str(e)
    
    # Check Redis
    try:
        r = redis.from_url(settings.REDIS_URL)
        r.ping()
        health['redis'] = True
    except Exception as e:
        health['status'] = 'unhealthy'
        health['redis_error'] = str(e)
    
    status_code = 200 if health['status'] == 'healthy' else 503
    return Response(health, status=status_code)
```

### 6.2 Logging Configuration

```python
# config/settings/base.py

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
        'json': {
            '()': 'pythonjsonlogger.jsonlogger.JsonFormatter',
            'format': '%(asctime)s %(levelname)s %(name)s %(message)s'
        },
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'formatter': 'json',
        },
        'file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': '/var/log/flowspark/app.log',
            'maxBytes': 10485760,  # 10MB
            'backupCount': 5,
            'formatter': 'json',
        },
    },
    'root': {
        'handlers': ['console', 'file'],
        'level': 'INFO',
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'file'],
            'level': 'INFO',
            'propagate': False,
        },
        'apps': {
            'handlers': ['console', 'file'],
            'level': 'DEBUG',
            'propagate': False,
        },
    },
}
```

### 6.3 Sentry Integration

```python
# config/settings/production.py
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration
from sentry_sdk.integrations.celery import CeleryIntegration

sentry_sdk.init(
    dsn=os.getenv('SENTRY_DSN'),
    integrations=[
        DjangoIntegration(),
        CeleryIntegration(),
    ],
    traces_sample_rate=0.1,
    send_default_pii=False,
    environment='production'
)
```

### 6.4 Metrics & Monitoring Stack

```yaml
# docker-compose.monitoring.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana_data:/var/lib/grafana
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - prometheus

  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    volumes:
      - loki_data:/loki

  promtail:
    image: grafana/promtail:latest
    volumes:
      - /var/log:/var/log
      - ./promtail-config.yml:/etc/promtail/config.yml
    depends_on:
      - loki

volumes:
  prometheus_data:
  grafana_data:
  loki_data:
```

---

## 7. ROLLBACK STRATEGY

### 7.1 Kubernetes Rollback

```bash
# Check rollout history
kubectl rollout history deployment/flowspark-backend -n production

# Rollback to previous version
kubectl rollout undo deployment/flowspark-backend -n production

# Rollback to specific revision
kubectl rollout undo deployment/flowspark-backend --to-revision=3 -n production

# Monitor rollback status
kubectl rollout status deployment/flowspark-backend -n production
```

### 7.2 Database Rollback

```bash
# List migrations
python manage.py showmigrations

# Rollback to specific migration
python manage.py migrate app_name 0005_previous_migration

# Create rollback migration if needed
python manage.py makemigrations --empty app_name
```

---

## 8. DISASTER RECOVERY

### 8.1 Backup Strategy

**Database Backups**:
- **Frequency**: Daily automated backups
- **Retention**: 30 days
- **Storage**: S3 with versioning enabled

```bash
# Automated backup script
#!/bin/bash
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="flowspark_backup_${TIMESTAMP}.sql"

pg_dump $DATABASE_URL > /tmp/$BACKUP_FILE
aws s3 cp /tmp/$BACKUP_FILE s3://flowspark-backups/database/
rm /tmp/$BACKUP_FILE
```

**Object Storage Backups**:
- Cross-region replication enabled
- Versioning enabled
- Lifecycle policies for old versions

### 8.2 Recovery Procedures

1. **Database Restore**:
```bash
# Download backup
aws s3 cp s3://flowspark-backups/database/backup.sql /tmp/

# Restore
psql $DATABASE_URL < /tmp/backup.sql
```

2. **Application Recovery**:
```bash
# Redeploy last known good version
kubectl set image deployment/backend backend=flowspark/backend:v1.2.3
```

---

**Document Version**: 1.0  
**Created**: December 28, 2025  
**Status**: Draft for Review

