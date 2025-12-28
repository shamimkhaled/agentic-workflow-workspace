# FlowSpark AI - Database Design & Schema
## Complete Relational Schema, Relationships & Indexing

---

## TABLE OF CONTENTS
1. [Database Overview](#database-overview)
2. [Core Schema](#core-schema)
3. [Entity Relationship Diagram](#entity-relationship-diagram)
4. [Indexing Strategy](#indexing-strategy)
5. [Scalability Considerations](#scalability-considerations)

---

## 1. DATABASE OVERVIEW

### 1.1 Database Technology Stack

**Primary Database**: PostgreSQL 15+
- ACID compliance for transactional integrity
- Advanced indexing (B-tree, GiST, GIN)
- JSON/JSONB support for flexible metadata
- Full-text search capabilities
- Excellent Django ORM support

**Vector Database**: ChromaDB / Weaviate
- Specialized for embedding storage and similarity search
- Separate from relational data for performance

**Cache Layer**: Redis
- Session storage
- Rate limiting
- Job queue (Celery broker)
- Short-term memory

---

## 2. CORE SCHEMA

### 2.1 Core Models (Tenancy & Users)

```sql
-- ==========================================
-- CORE: TENANTS & USERS
-- ==========================================

CREATE TYPE tenant_tier AS ENUM ('free', 'standard', 'professional', 'enterprise');
CREATE TYPE tenant_status AS ENUM ('active', 'suspended', 'trial', 'cancelled');
CREATE TYPE user_role AS ENUM ('owner', 'admin', 'manager', 'member', 'viewer');

-- Tenants (Organizations)
CREATE TABLE core_tenant (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    
    -- Subscription
    tier tenant_tier NOT NULL DEFAULT 'free',
    status tenant_status NOT NULL DEFAULT 'trial',
    
    -- Limits
    max_users INTEGER DEFAULT 5,
    max_storage_gb INTEGER DEFAULT 10,
    
    -- Branding
    logo_url TEXT,
    primary_color VARCHAR(7) DEFAULT '#0066CC',
    custom_domain VARCHAR(255),
    
    -- Isolation options (enterprise)
    dedicated_vector_db BOOLEAN DEFAULT FALSE,
    dedicated_storage BOOLEAN DEFAULT FALSE,
    
    -- Timestamps
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
    
    -- Indexes
    CONSTRAINT tenant_slug_unique UNIQUE(slug)
);

CREATE INDEX idx_tenant_status ON core_tenant(status);
CREATE INDEX idx_tenant_tier ON core_tenant(tier);

-- Users
CREATE TABLE core_user (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES core_tenant(id) ON DELETE CASCADE,
    
    -- Authentication
    email VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    email_verified BOOLEAN DEFAULT FALSE,
    
    -- Profile
    first_name VARCHAR(150),
    last_name VARCHAR(150),
    avatar_url TEXT,
    phone VARCHAR(20),
    department VARCHAR(100),
    
    -- Role
    role user_role NOT NULL DEFAULT 'member',
    
    -- Preferences
    timezone VARCHAR(50) DEFAULT 'UTC',
    language VARCHAR(10) DEFAULT 'en',
    
    -- Timestamps
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
    last_login TIMESTAMP,
    
    -- Constraints
    CONSTRAINT user_email_tenant_unique UNIQUE(email, tenant_id)
);

CREATE INDEX idx_user_tenant ON core_user(tenant_id);
CREATE INDEX idx_user_email ON core_user(email);
CREATE INDEX idx_user_role ON core_user(tenant_id, role);
```

---

### 2.2 Document Models

```sql
-- ==========================================
-- DOCUMENTS
-- ==========================================

CREATE TYPE document_status AS ENUM ('pending', 'processing', 'completed', 'failed');
CREATE TYPE document_format AS ENUM ('docx', 'pdf', 'xlsx');
CREATE TYPE template_type AS ENUM ('word', 'pdf', 'excel');
CREATE TYPE document_visibility AS ENUM ('private', 'team', 'organization');

-- Document Templates
CREATE TABLE documents_template (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES core_tenant(id) ON DELETE CASCADE,
    created_by_id UUID REFERENCES core_user(id) ON DELETE SET NULL,
    
    -- Template info
    name VARCHAR(255) NOT NULL,
    description TEXT,
    type template_type NOT NULL,
    
    -- Template file
    file_url TEXT NOT NULL,
    
    -- Structure (JSON)
    variables JSONB DEFAULT '[]',
    
    -- Branding
    is_branded BOOLEAN DEFAULT FALSE,
    
    -- Visibility
    is_public BOOLEAN DEFAULT FALSE,
    
    -- Usage tracking
    usage_count INTEGER DEFAULT 0,
    
    -- Timestamps
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_template_tenant ON documents_template(tenant_id);
CREATE INDEX idx_template_type ON documents_template(tenant_id, type);
CREATE INDEX idx_template_public ON documents_template(is_public) WHERE is_public = TRUE;

-- Generated Documents
CREATE TABLE documents_document (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES core_tenant(id) ON DELETE CASCADE,
    created_by_id UUID NOT NULL REFERENCES core_user(id) ON DELETE CASCADE,
    template_id UUID REFERENCES documents_template(id) ON DELETE SET NULL,
    
    -- Document info
    title VARCHAR(255) NOT NULL,
    description TEXT,
    format document_format NOT NULL,
    status document_status DEFAULT 'pending',
    
    -- Content
    input_data JSONB,
    generated_content TEXT,
    
    -- Files
    file_url TEXT,
    file_size_bytes BIGINT DEFAULT 0,
    
    -- Generation metadata
    generation_time_seconds FLOAT,
    tokens_used INTEGER DEFAULT 0,
    model_used VARCHAR(100),
    
    -- Error tracking
    error_message TEXT,
    
    -- Access control
    visibility document_visibility DEFAULT 'private',
    
    -- Timestamps
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_document_tenant_user ON documents_document(tenant_id, created_by_id, created_at DESC);
CREATE INDEX idx_document_status ON documents_document(tenant_id, status);
CREATE INDEX idx_document_created_at ON documents_document(created_at DESC);

-- Document Sharing (many-to-many)
CREATE TABLE documents_document_shared_with (
    id BIGSERIAL PRIMARY KEY,
    document_id UUID NOT NULL REFERENCES documents_document(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES core_user(id) ON DELETE CASCADE,
    
    shared_at TIMESTAMP DEFAULT NOW(),
    
    CONSTRAINT document_user_unique UNIQUE(document_id, user_id)
);

CREATE INDEX idx_shared_docs_user ON documents_document_shared_with(user_id);

-- Document Versions (for tracking changes)
CREATE TABLE documents_version (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id UUID NOT NULL REFERENCES documents_document(id) ON DELETE CASCADE,
    
    version_number INTEGER NOT NULL,
    file_url TEXT NOT NULL,
    content_snapshot TEXT,
    
    created_by_id UUID REFERENCES core_user(id) ON DELETE SET NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    
    CONSTRAINT doc_version_number_unique UNIQUE(document_id, version_number)
);

CREATE INDEX idx_version_document ON documents_version(document_id, version_number DESC);
```

---

### 2.3 Knowledge Base Models

```sql
-- ==========================================
-- KNOWLEDGE BASE (RAG)
-- ==========================================

CREATE TYPE kb_status AS ENUM ('uploading', 'processing', 'indexed', 'failed');

-- Knowledge Base Documents
CREATE TABLE kb_document (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES core_tenant(id) ON DELETE CASCADE,
    uploaded_by_id UUID NOT NULL REFERENCES core_user(id) ON DELETE CASCADE,
    
    -- File info
    title VARCHAR(255) NOT NULL,
    file_name VARCHAR(255) NOT NULL,
    file_url TEXT NOT NULL,
    file_type VARCHAR(50),
    file_size_bytes BIGINT,
    
    -- Processing status
    status kb_status DEFAULT 'uploading',
    
    -- Processing metadata
    total_pages INTEGER,
    total_chunks INTEGER DEFAULT 0,
    processing_time_seconds FLOAT,
    
    -- Embedding metadata
    embedding_model VARCHAR(100),
    vector_db_collection VARCHAR(255),
    
    -- Content
    extracted_text TEXT,
    metadata JSONB DEFAULT '{}',
    
    -- Categorization
    tags JSONB DEFAULT '[]',
    category VARCHAR(100),
    
    -- Error tracking
    error_message TEXT,
    
    -- Timestamps
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_kb_doc_tenant ON kb_document(tenant_id);
CREATE INDEX idx_kb_doc_status ON kb_document(tenant_id, status);
CREATE INDEX idx_kb_doc_uploader ON kb_document(tenant_id, uploaded_by_id, created_at DESC);
CREATE INDEX idx_kb_doc_tags ON kb_document USING GIN(tags);

-- Document Chunks (references to vector DB)
CREATE TABLE kb_chunk (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id UUID NOT NULL REFERENCES kb_document(id) ON DELETE CASCADE,
    
    -- Content
    content TEXT NOT NULL,
    chunk_index INTEGER NOT NULL,
    
    -- Vector DB reference
    vector_db_id VARCHAR(255) NOT NULL,
    
    -- Metadata
    metadata JSONB DEFAULT '{}',
    page_number INTEGER,
    
    CONSTRAINT chunk_doc_index_unique UNIQUE(document_id, chunk_index)
);

CREATE INDEX idx_chunk_document ON kb_chunk(document_id, chunk_index);
CREATE INDEX idx_chunk_vector_id ON kb_chunk(vector_db_id);

-- KB Query History
CREATE TABLE kb_query (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES core_tenant(id) ON DELETE CASCADE,
    queried_by_id UUID NOT NULL REFERENCES core_user(id) ON DELETE CASCADE,
    
    -- Query
    query_text TEXT NOT NULL,
    response_text TEXT,
    
    -- Retrieved chunks (JSON array of chunk IDs)
    retrieved_chunks JSONB DEFAULT '[]',
    
    -- Performance metrics
    retrieval_time_seconds FLOAT,
    generation_time_seconds FLOAT,
    
    -- Model info
    embedding_model VARCHAR(100),
    llm_model VARCHAR(100),
    
    -- Tokens
    tokens_used INTEGER DEFAULT 0,
    
    -- Timestamps
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_query_tenant ON kb_query(tenant_id, created_at DESC);
CREATE INDEX idx_query_user ON kb_query(queried_by_id, created_at DESC);
```

---

### 2.4 Billing Models

```sql
-- ==========================================
-- BILLING & SUBSCRIPTIONS
-- ==========================================

CREATE TYPE subscription_status AS ENUM ('trialing', 'active', 'past_due', 'canceled', 'unpaid');
CREATE TYPE subscription_interval AS ENUM ('monthly', 'yearly');

-- Subscription Plans
CREATE TABLE billing_plan (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(50) UNIQUE NOT NULL,
    tier tenant_tier NOT NULL,
    
    -- Pricing
    price_monthly DECIMAL(10, 2) NOT NULL,
    price_yearly DECIMAL(10, 2) NOT NULL,
    
    -- Limits
    max_users INTEGER NOT NULL,
    max_documents_per_month INTEGER NOT NULL,
    max_storage_gb INTEGER NOT NULL,
    included_tokens INTEGER DEFAULT 0,
    
    -- Features (JSON)
    features JSONB DEFAULT '{}',
    
    -- Stripe
    stripe_price_id_monthly VARCHAR(100),
    stripe_price_id_yearly VARCHAR(100),
    
    is_active BOOLEAN DEFAULT TRUE,
    
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Tenant Subscriptions
CREATE TABLE billing_subscription (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES core_tenant(id) ON DELETE CASCADE,
    plan_id UUID NOT NULL REFERENCES billing_plan(id) ON DELETE PROTECT,
    
    status subscription_status DEFAULT 'trialing',
    interval subscription_interval DEFAULT 'monthly',
    
    -- Stripe
    stripe_subscription_id VARCHAR(100),
    stripe_customer_id VARCHAR(100),
    
    -- Dates
    trial_end TIMESTAMP,
    current_period_start TIMESTAMP NOT NULL,
    current_period_end TIMESTAMP NOT NULL,
    cancelled_at TIMESTAMP,
    
    -- Usage tracking (resets each period)
    documents_used_this_period INTEGER DEFAULT 0,
    storage_used_gb FLOAT DEFAULT 0.0,
    
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    CONSTRAINT subscription_tenant_unique UNIQUE(tenant_id)
);

CREATE INDEX idx_subscription_tenant ON billing_subscription(tenant_id);
CREATE INDEX idx_subscription_status ON billing_subscription(status);
CREATE INDEX idx_subscription_stripe ON billing_subscription(stripe_subscription_id);

-- Token Packages
CREATE TABLE billing_token_package (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    name VARCHAR(100) NOT NULL,
    tokens INTEGER NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    
    stripe_price_id VARCHAR(100),
    
    is_active BOOLEAN DEFAULT TRUE,
    
    created_at TIMESTAMP DEFAULT NOW()
);

-- Token Balance
CREATE TABLE billing_token_balance (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES core_tenant(id) ON DELETE CASCADE,
    
    balance INTEGER DEFAULT 0,
    lifetime_purchased INTEGER DEFAULT 0,
    lifetime_used INTEGER DEFAULT 0,
    
    -- Alert settings
    low_balance_threshold INTEGER DEFAULT 1000,
    low_balance_alert_sent BOOLEAN DEFAULT FALSE,
    
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    CONSTRAINT token_balance_tenant_unique UNIQUE(tenant_id)
);

CREATE INDEX idx_token_balance_tenant ON billing_token_balance(tenant_id);

-- Token Transactions
CREATE TABLE billing_token_transaction (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES core_tenant(id) ON DELETE CASCADE,
    
    amount INTEGER NOT NULL, -- Positive for additions, negative for usage
    operation VARCHAR(100) NOT NULL, -- 'document_generation', 'token_purchase', etc.
    balance_after INTEGER NOT NULL,
    
    -- Metadata
    metadata JSONB DEFAULT '{}',
    
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_token_tx_tenant ON billing_token_transaction(tenant_id, created_at DESC);
CREATE INDEX idx_token_tx_operation ON billing_token_transaction(tenant_id, operation);
```

---

### 2.5 Analytics & Audit Models

```sql
-- ==========================================
-- ANALYTICS & AUDIT
-- ==========================================

-- Usage Analytics
CREATE TABLE analytics_usage (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES core_tenant(id) ON DELETE CASCADE,
    user_id UUID REFERENCES core_user(id) ON DELETE SET NULL,
    
    -- Activity
    activity_type VARCHAR(100) NOT NULL, -- 'document_generated', 'kb_query', 'image_generated'
    
    -- Metrics
    tokens_used INTEGER DEFAULT 0,
    execution_time_seconds FLOAT,
    
    -- Metadata
    metadata JSONB DEFAULT '{}',
    
    -- Timestamp
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_usage_tenant_date ON analytics_usage(tenant_id, created_at DESC);
CREATE INDEX idx_usage_activity ON analytics_usage(tenant_id, activity_type, created_at DESC);
CREATE INDEX idx_usage_user ON analytics_usage(user_id, created_at DESC);

-- Audit Logs
CREATE TABLE audit_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES core_tenant(id) ON DELETE CASCADE,
    user_id UUID REFERENCES core_user(id) ON DELETE SET NULL,
    
    -- Action
    action VARCHAR(100) NOT NULL, -- 'user.created', 'document.deleted', etc.
    resource_type VARCHAR(50), -- 'user', 'document', etc.
    resource_id UUID,
    
    -- Details
    details JSONB DEFAULT '{}',
    
    -- Metadata
    ip_address INET,
    user_agent TEXT,
    
    -- Timestamp
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_audit_tenant ON audit_log(tenant_id, created_at DESC);
CREATE INDEX idx_audit_user ON audit_log(user_id, created_at DESC);
CREATE INDEX idx_audit_action ON audit_log(tenant_id, action);
CREATE INDEX idx_audit_resource ON audit_log(resource_type, resource_id);
```

---

### 2.6 Agent & Memory Models

```sql
-- ==========================================
-- AGENTS & MEMORY
-- ==========================================

-- Agent Interactions
CREATE TABLE agents_interaction (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES core_tenant(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES core_user(id) ON DELETE CASCADE,
    
    interaction_type VARCHAR(100) NOT NULL,
    data JSONB NOT NULL,
    
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_interaction_tenant ON agents_interaction(tenant_id, created_at DESC);
CREATE INDEX idx_interaction_user ON agents_interaction(user_id, created_at DESC);

-- User Preferences (for memory)
CREATE TABLE agents_preference (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES core_tenant(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES core_user(id) ON DELETE CASCADE,
    
    key VARCHAR(100) NOT NULL,
    value JSONB NOT NULL,
    
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    CONSTRAINT preference_user_key_unique UNIQUE(user_id, key)
);

CREATE INDEX idx_preference_user ON agents_preference(user_id);
```

---

## 3. ENTITY RELATIONSHIP DIAGRAM

```
┌─────────────────┐
│  core_tenant    │
│                 │
│  id (PK)        │
│  name           │
│  slug           │
│  tier           │
│  status         │
└────────┬────────┘
         │
         │ 1:N
         │
┌────────▼────────┐
│  core_user      │
│                 │
│  id (PK)        │
│  tenant_id (FK) │
│  email          │
│  role           │
└────────┬────────┘
         │
         │ 1:N
         │
    ┌────▼─────────────────┬──────────────────┬──────────────────┐
    │                      │                  │                  │
┌───▼────────────┐  ┌─────▼────────┐  ┌─────▼─────────┐  ┌─────▼──────────┐
│ documents_doc  │  │ kb_document  │  │ kb_query      │  │ analytics_usage│
│                │  │              │  │               │  │                │
│ id (PK)        │  │ id (PK)      │  │ id (PK)       │  │ id (PK)        │
│ tenant_id (FK) │  │ tenant_id    │  │ tenant_id     │  │ tenant_id (FK) │
│ created_by(FK) │  │ uploaded_by  │  │ queried_by    │  │ user_id (FK)   │
│ template_id    │  │              │  │               │  │                │
└────────────────┘  └──────┬───────┘  └───────────────┘  └────────────────┘
                           │
                           │ 1:N
                           │
                    ┌──────▼───────┐
                    │  kb_chunk    │
                    │              │
                    │  id (PK)     │
                    │  document_id │
                    │  content     │
                    └──────────────┘

┌─────────────────┐
│ billing_plan    │
│                 │
│ id (PK)         │
│ name            │
│ tier            │
└────────┬────────┘
         │
         │ N:1
         │
┌────────▼──────────────┐
│ billing_subscription  │
│                       │
│ id (PK)               │
│ tenant_id (FK)        │
│ plan_id (FK)          │
│ stripe_subscription_id│
└───────────────────────┘

┌──────────────────────┐
│ billing_token_balance│
│                      │
│ tenant_id (FK)       │
│ balance              │
└───────┬──────────────┘
        │
        │ 1:N
        │
┌───────▼────────────────────┐
│ billing_token_transaction  │
│                            │
│ tenant_id (FK)             │
│ amount                     │
│ operation                  │
└────────────────────────────┘
```

---

## 4. INDEXING STRATEGY

### 4.1 Index Types & Usage

| Index Type | Use Case | Example |
|------------|----------|---------|
| **B-Tree (Default)** | Equality, range queries | `created_at`, `email` |
| **GIN** | JSONB, array fields | `tags`, `metadata` |
| **GiST** | Full-text search | `document_content` (if using PostgreSQL FTS) |
| **Partial Index** | Filtered queries | `WHERE is_public = TRUE` |
| **Composite Index** | Multi-column queries | `(tenant_id, created_at DESC)` |

### 4.2 Critical Indexes

```sql
-- High-traffic query patterns

-- Tenant-scoped queries (most common)
CREATE INDEX idx_doc_tenant_date ON documents_document(tenant_id, created_at DESC);
CREATE INDEX idx_kb_tenant_status ON kb_document(tenant_id, status);

-- User activity queries
CREATE INDEX idx_doc_user_date ON documents_document(created_by_id, created_at DESC);
CREATE INDEX idx_query_user_date ON kb_query(queried_by_id, created_at DESC);

-- Analytics time-series queries
CREATE INDEX idx_usage_tenant_time ON analytics_usage(tenant_id, created_at DESC);
CREATE INDEX idx_audit_tenant_time ON audit_log(tenant_id, created_at DESC);

-- Search and filtering
CREATE INDEX idx_doc_status ON documents_document(tenant_id, status) WHERE status != 'completed';
CREATE INDEX idx_kb_tags ON kb_document USING GIN(tags);
```

### 4.3 Index Maintenance

```sql
-- Periodic index maintenance (run monthly)
REINDEX DATABASE flowspark_ai;

-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM documents_document WHERE tenant_id = 'uuid' ORDER BY created_at DESC LIMIT 20;

-- Identify missing indexes
SELECT schemaname, tablename, attname, n_distinct, correlation
FROM pg_stats
WHERE schemaname = 'public' AND tablename = 'documents_document';
```

---

## 5. SCALABILITY CONSIDERATIONS

### 5.1 Partitioning Strategy

**Time-Based Partitioning** for high-volume tables:

```sql
-- Partition analytics_usage by month
CREATE TABLE analytics_usage_2025_01 PARTITION OF analytics_usage
    FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');

CREATE TABLE analytics_usage_2025_02 PARTITION OF analytics_usage
    FOR VALUES FROM ('2025-02-01') TO ('2025-03-01');

-- Automate partition creation with cron job or pg_cron
```

### 5.2 Read Replicas

- **Primary**: Write operations
- **Read Replica 1**: Analytics queries
- **Read Replica 2**: API read queries

### 5.3 Connection Pooling

```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'flowspark_ai',
        'CONN_MAX_AGE': 600,  # Connection pooling
        'OPTIONS': {
            'connect_timeout': 10,
            'options': '-c statement_timeout=30000'  # 30s query timeout
        }
    }
}
```

### 5.4 Query Optimization Tips

1. **Always filter by tenant_id first** (leverages partition key)
2. **Use `select_related()` and `prefetch_related()`** to avoid N+1 queries
3. **Limit result sets** with pagination
4. **Use `.only()` and `.defer()`** to fetch only needed fields
5. **Cache frequently accessed data** (plans, tenant settings)

---

## 6. DATABASE MIGRATIONS

### 6.1 Migration Best Practices

```python
# Example migration for adding index
from django.db import migrations, models

class Migration(migrations.Migration):
    dependencies = [
        ('documents', '0002_previous_migration'),
    ]

    operations = [
        migrations.AddIndex(
            model_name='document',
            index=models.Index(
                fields=['tenant', 'status', '-created_at'],
                name='doc_tenant_status_date_idx'
            ),
        ),
    ]
```

### 6.2 Zero-Downtime Migrations

For large tables:
1. Add new column as nullable
2. Backfill data in batches (background task)
3. Make column non-nullable
4. Remove old column

---

**Document Version**: 1.0  
**Created**: December 28, 2025  
**Status**: Draft for Review

