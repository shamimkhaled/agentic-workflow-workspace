# FlowSpark AI - Quick Reference Guide
## Essential Information at a Glance

---

## 🎯 CORE CONCEPT

**FlowSpark AI** = Multi-Agent AI Platform for Document Automation

```
User Request → Orchestrator Agent → Specialized Agents → Final Output
                      ↓
    ├─ Document Agent (Word/PDF)
    ├─ Presentation Agent (PPT)
    ├─ KB Agent (RAG Search)
    ├─ Image Agent (Visuals)
    └─ Analysis Agent (Insights)
```

---

## 📋 ESSENTIAL COMMANDS

### Development Setup
```bash
# Clone and setup
git clone <repo-url>
cd flowspark-ai
cp .env.example .env

# Start with Docker
docker-compose up -d

# Run migrations
docker-compose exec backend python manage.py migrate

# Create superuser
docker-compose exec backend python manage.py createsuperuser

# Access
# Backend: http://localhost:8000/api/v1/
# Frontend: http://localhost:3000/
```

### Testing
```bash
# Run all tests
pytest

# With coverage
pytest --cov=apps --cov-report=html

# Specific app
pytest apps/documents/tests/

# Backend linting
black .
flake8 apps/
pylint apps/

# Frontend
cd frontend
npm run test
npm run lint
```

### Database
```bash
# Make migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Create migration (empty)
python manage.py makemigrations --empty app_name

# Show migrations
python manage.py showmigrations

# Rollback
python manage.py migrate app_name 0005_previous
```

### Celery
```bash
# Start worker
celery -A config worker -l info

# Start beat (scheduler)
celery -A config beat -l info

# Monitor
celery -A config inspect active
celery -A config inspect stats
```

---

## 🏗️ KEY FILE LOCATIONS

### Backend (Django/DRF)
```
backend/
├── apps/
│   ├── core/models/tenant.py          # Tenant model
│   ├── core/middleware/tenant.py      # Tenant context
│   ├── authentication/                # Auth endpoints
│   ├── documents/                     # Document generation
│   ├── knowledge_base/                # RAG system
│   ├── agents/orchestrator.py         # Agent orchestration
│   └── billing/                       # Subscription & tokens
├── config/settings/                   # Settings by environment
├── services/llm/                      # LLM services
└── requirements.txt                   # Dependencies
```

### Frontend (React)
```
frontend/
├── src/
│   ├── components/                    # Reusable components
│   ├── pages/                         # Page components
│   ├── services/api.ts                # API client
│   ├── contexts/AuthContext.tsx       # Auth state
│   └── utils/                         # Utilities
└── package.json
```

---

## 🔑 ENVIRONMENT VARIABLES

### Essential Variables
```bash
# Django
SECRET_KEY=your-secret-key
DEBUG=False
ALLOWED_HOSTS=localhost,yourdomain.com

# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/flowspark

# Redis
REDIS_URL=redis://localhost:6379/0

# AI Services
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
OLLAMA_BASE_URL=http://localhost:11434

# Storage
S3_BUCKET_NAME=flowspark-storage
S3_ACCESS_KEY=your-key
S3_SECRET_KEY=your-secret

# Stripe
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

---

## 📊 DATABASE QUICK REFERENCE

### Core Tables
```sql
-- Tenancy
core_tenant (id, name, slug, tier, status)
core_user (id, tenant_id, email, role)

-- Documents
documents_template (id, tenant_id, name, type, file_url)
documents_document (id, tenant_id, created_by_id, title, status, file_url)

-- Knowledge Base
kb_document (id, tenant_id, uploaded_by_id, status, total_chunks)
kb_chunk (id, document_id, content, vector_db_id)

-- Billing
billing_subscription (id, tenant_id, plan_id, status)
billing_token_balance (id, tenant_id, balance)
billing_token_transaction (id, tenant_id, amount, operation)
```

### Common Queries
```sql
-- Get tenant documents
SELECT * FROM documents_document 
WHERE tenant_id = 'uuid' 
ORDER BY created_at DESC LIMIT 20;

-- Check token balance
SELECT balance FROM billing_token_balance WHERE tenant_id = 'uuid';

-- User activity
SELECT COUNT(*) FROM analytics_usage 
WHERE tenant_id = 'uuid' 
  AND created_at > NOW() - INTERVAL '7 days';
```

---

## 🔐 API AUTHENTICATION

### JWT Token Flow
```bash
# 1. Login
POST /api/v1/auth/login/
{
  "email": "user@example.com",
  "password": "password123"
}

# Response:
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "user": {...}
}

# 2. Use access token
Authorization: Bearer eyJ...

# 3. Refresh when expired
POST /api/v1/auth/refresh/
{
  "refresh_token": "eyJ..."
}
```

---

## 🎨 COMMON API ENDPOINTS

### Authentication
```
POST   /api/v1/auth/register/
POST   /api/v1/auth/login/
POST   /api/v1/auth/refresh/
GET    /api/v1/auth/me/
```

### Documents
```
POST   /api/v1/documents/generate/
GET    /api/v1/documents/
GET    /api/v1/documents/{id}/
DELETE /api/v1/documents/{id}/
GET    /api/v1/documents/{id}/download/
```

### Knowledge Base
```
POST   /api/v1/kb/documents/upload/
GET    /api/v1/kb/documents/
POST   /api/v1/kb/query/
DELETE /api/v1/kb/documents/{id}/
```

### Billing
```
GET    /api/v1/billing/subscription/
POST   /api/v1/billing/tokens/purchase/
GET    /api/v1/billing/tokens/balance/
GET    /api/v1/billing/tokens/transactions/
```

---

## 🐛 TROUBLESHOOTING

### Common Issues

**Database Connection Error**
```bash
# Check if PostgreSQL is running
docker-compose ps
# Check connection string
echo $DATABASE_URL
# Test connection
psql $DATABASE_URL -c "SELECT 1;"
```

**Redis Connection Error**
```bash
# Check Redis
docker-compose ps redis
redis-cli ping  # Should return PONG
```

**Migration Conflicts**
```bash
# Reset migrations (dev only!)
python manage.py migrate app_name zero
python manage.py showmigrations
python manage.py migrate
```

**Celery Tasks Not Running**
```bash
# Check worker is running
celery -A config inspect active
# Check Redis connection
redis-cli ping
# Restart worker
docker-compose restart celery_worker
```

**Out of Tokens**
```sql
-- Check balance
SELECT balance FROM billing_token_balance WHERE tenant_id = 'uuid';
-- Add tokens (dev only)
UPDATE billing_token_balance SET balance = balance + 10000 WHERE tenant_id = 'uuid';
```

---

## 📈 MONITORING QUERIES

### System Health
```sql
-- Active subscriptions
SELECT COUNT(*) FROM billing_subscription WHERE status = 'active';

-- Documents generated today
SELECT COUNT(*) FROM documents_document 
WHERE created_at > CURRENT_DATE;

-- Average generation time
SELECT AVG(generation_time_seconds) 
FROM documents_document 
WHERE status = 'completed' 
  AND created_at > NOW() - INTERVAL '7 days';

-- Token usage by tenant (top 10)
SELECT tenant_id, SUM(tokens_used) as total_tokens
FROM analytics_usage
WHERE created_at > NOW() - INTERVAL '30 days'
GROUP BY tenant_id
ORDER BY total_tokens DESC
LIMIT 10;
```

### Performance
```sql
-- Slow queries
SELECT query, calls, mean_exec_time, max_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Database size
SELECT pg_size_pretty(pg_database_size('flowspark_prod'));

-- Table sizes
SELECT schemaname, tablename,
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename))
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

---

## 🚀 DEPLOYMENT CHECKLIST

### Pre-Deployment
- [ ] All tests passing
- [ ] Code review approved
- [ ] Migrations tested
- [ ] Environment variables set
- [ ] Secrets configured
- [ ] Database backup created

### Deployment Steps
```bash
# 1. Build and push Docker image
docker build -t flowspark/backend:v1.0.0 .
docker push flowspark/backend:v1.0.0

# 2. Update Kubernetes deployment
kubectl set image deployment/backend backend=flowspark/backend:v1.0.0 -n production

# 3. Run migrations (if needed)
kubectl run migration-job --image=flowspark/backend:v1.0.0 \
  --command -- python manage.py migrate

# 4. Monitor rollout
kubectl rollout status deployment/backend -n production

# 5. Verify health
curl https://api.flowspark.ai/health/
```

### Post-Deployment
- [ ] Health check passing
- [ ] Logs reviewed (no errors)
- [ ] Monitoring dashboards checked
- [ ] Smoke tests passed
- [ ] Notify team

---

## 🔍 USEFUL DJANGO COMMANDS

```bash
# Shell
python manage.py shell
python manage.py shell_plus  # if django-extensions installed

# Database
python manage.py dbshell
python manage.py inspectdb  # Generate models from existing DB

# Users
python manage.py createsuperuser
python manage.py changepassword username

# Static files
python manage.py collectstatic

# Development server
python manage.py runserver 0.0.0.0:8000

# Show URLs
python manage.py show_urls  # if django-extensions installed
```

---

## 📚 QUICK LINKS

### Documentation
- [Executive Summary](./00-EXECUTIVE-SUMMARY-AND-INDEX.md)
- [Product Vision](./01-PRODUCT-VISION-AND-ARCHITECTURE.md)
- [Backend Architecture](./04-BACKEND-ARCHITECTURE-DRF.md)
- [Database Schema](./06-DATABASE-DESIGN-SCHEMA.md)

### External Resources
- [Django Docs](https://docs.djangoproject.com/)
- [DRF Docs](https://www.django-rest-framework.org/)
- [LangGraph Docs](https://langchain-ai.github.io/langgraph/)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)

### Tools
- **API Testing**: Postman, Insomnia, curl
- **Database**: pgAdmin, DBeaver
- **Monitoring**: Grafana, Sentry
- **Logs**: Kibana, Papertrail

---

## 💡 PRO TIPS

1. **Always filter by tenant_id first** in queries for performance
2. **Use select_related() and prefetch_related()** to avoid N+1 queries
3. **Cache frequently accessed data** (plans, tenant settings)
4. **Monitor token usage** to prevent unexpected costs
5. **Test multi-tenant isolation** thoroughly
6. **Log all API errors** for debugging
7. **Use feature flags** for gradual rollouts
8. **Back up before migrations** in production

---

## 🎯 DEVELOPMENT PRIORITIES

### Sprint 1-2 (Weeks 1-4)
- [ ] User authentication
- [ ] Tenant middleware
- [ ] Basic API endpoints
- [ ] Frontend auth flow

### Sprint 3-4 (Weeks 5-8)
- [ ] Document generation
- [ ] LLM integration
- [ ] Document creator UI
- [ ] Template management

### Sprint 5-6 (Weeks 9-12)
- [ ] Knowledge Base (RAG)
- [ ] Document ingestion
- [ ] Vector search
- [ ] KB query UI

---

**Created**: December 28, 2025  
**Version**: 1.0  
**Purpose**: Quick reference for developers

**Keep this document bookmarked for daily development!** 🚀

