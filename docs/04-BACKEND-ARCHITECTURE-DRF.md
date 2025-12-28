# FlowSpark AI - Backend Architecture (Django REST Framework)
## Production-Grade DRF Implementation

---

## TABLE OF CONTENTS
1. [Django Project Structure](#django-project-structure)
2. [App Module Breakdown](#app-module-breakdown)
3. [API Design Principles](#api-design-principles)
4. [Authentication & Authorization](#authentication--authorization)
5. [Database Models](#database-models)
6. [Services Layer](#services-layer)
7. [Background Tasks](#background-tasks)
8. [Error Handling & Logging](#error-handling--logging)

---

## 1. DJANGO PROJECT STRUCTURE

### 1.1 Recommended Project Structure

```
flowspark_ai/
├── manage.py
├── requirements.txt
├── .env.example
├── .gitignore
├── docker-compose.yml
├── Dockerfile
│
├── config/                          # Project configuration
│   ├── __init__.py
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py                 # Base settings
│   │   ├── development.py          # Dev settings
│   │   ├── production.py           # Production settings
│   │   └── test.py                 # Test settings
│   ├── urls.py                     # Root URL configuration
│   ├── wsgi.py
│   └── asgi.py
│
├── apps/                            # All Django apps
│   ├── __init__.py
│   │
│   ├── core/                        # Core models and utilities
│   │   ├── __init__.py
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   ├── tenant.py
│   │   │   ├── user.py
│   │   │   └── base.py
│   │   ├── middleware/
│   │   │   ├── __init__.py
│   │   │   ├── tenant.py
│   │   │   └── exception.py
│   │   ├── permissions.py
│   │   ├── mixins.py
│   │   └── admin.py
│   │
│   ├── authentication/              # Auth and user management
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── jwt_service.py
│   │   │   └── sso_service.py
│   │   └── tests/
│   │
│   ├── documents/                   # Document generation
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── document_generator.py
│   │   │   ├── pdf_generator.py
│   │   │   └── template_service.py
│   │   ├── tasks.py                # Celery tasks
│   │   └── tests/
│   │
│   ├── presentations/               # Presentation generation
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   └── ppt_generator.py
│   │   ├── tasks.py
│   │   └── tests/
│   │
│   ├── knowledge_base/              # RAG and knowledge management
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── ingestion_service.py
│   │   │   ├── embedding_service.py
│   │   │   ├── retrieval_service.py
│   │   │   └── vector_db.py
│   │   ├── tasks.py
│   │   └── tests/
│   │
│   ├── images/                      # Image generation
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── image_generator.py
│   │   │   └── image_optimizer.py
│   │   ├── tasks.py
│   │   └── tests/
│   │
│   ├── agents/                      # Agentic orchestration
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── orchestrator.py
│   │   ├── agents/
│   │   │   ├── __init__.py
│   │   │   ├── base_agent.py
│   │   │   ├── document_agent.py
│   │   │   ├── presentation_agent.py
│   │   │   ├── kb_agent.py
│   │   │   └── image_agent.py
│   │   └── tests/
│   │
│   ├── billing/                     # Subscription and tokens
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── stripe_service.py
│   │   │   └── token_service.py
│   │   └── tests/
│   │
│   ├── analytics/                   # Usage analytics
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   └── analytics_service.py
│   │   └── tests/
│   │
│   └── templates/                   # Template management
│       ├── __init__.py
│       ├── models.py
│       ├── serializers.py
│       ├── views.py
│       ├── urls.py
│       └── tests/
│
├── services/                        # Shared services
│   ├── __init__.py
│   ├── llm/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── ollama_service.py
│   │   ├── openai_service.py
│   │   └── prompt_manager.py
│   ├── storage/
│   │   ├── __init__.py
│   │   └── s3_storage.py
│   └── cache/
│       ├── __init__.py
│       └── redis_cache.py
│
├── utils/                           # Utility functions
│   ├── __init__.py
│   ├── validators.py
│   ├── helpers.py
│   └── constants.py
│
├── tests/                           # Integration tests
│   ├── __init__.py
│   ├── fixtures/
│   └── integration/
│
└── scripts/                         # Management scripts
    ├── seed_data.py
    ├── setup_vector_db.py
    └── migrate_tenants.py
```

---

## 2. APP MODULE BREAKDOWN

### 2.1 Core App

**Purpose**: Foundation models and shared utilities

**Responsibilities**:
- Tenant model
- User model (extends AbstractUser)
- Base models (TenantAwareModel)
- Middleware (tenant context, exception handling)
- Permissions and RBAC
- Common mixins

**Key Files**:
- `models/tenant.py` - Tenant/Organization model
- `models/user.py` - Extended user model
- `middleware/tenant.py` - Tenant identification and context
- `permissions.py` - Permission classes and decorators
- `mixins.py` - Reusable view/model mixins

---

### 2.2 Authentication App

**Purpose**: User authentication and authorization

**Responsibilities**:
- User registration and login
- JWT token management
- Password reset
- SSO/OAuth integration
- API key management

**API Endpoints**:
```
POST   /api/v1/auth/register/
POST   /api/v1/auth/login/
POST   /api/v1/auth/logout/
POST   /api/v1/auth/refresh/
POST   /api/v1/auth/password/reset/
POST   /api/v1/auth/password/change/
GET    /api/v1/auth/me/
PUT    /api/v1/auth/me/
POST   /api/v1/auth/sso/{provider}/
GET    /api/v1/auth/sso/{provider}/callback/
```

---

### 2.3 Documents App

**Purpose**: Document generation and management

**Responsibilities**:
- Create documents (Word, PDF, Excel)
- Template management
- Document versioning
- Document storage and retrieval

**API Endpoints**:
```
POST   /api/v1/documents/generate/
GET    /api/v1/documents/
GET    /api/v1/documents/{id}/
PUT    /api/v1/documents/{id}/
DELETE /api/v1/documents/{id}/
GET    /api/v1/documents/{id}/download/
GET    /api/v1/documents/{id}/versions/
```

**Models**:
```python
# apps/documents/models.py
from django.db import models
from apps.core.models.base import TenantAwareModel

class DocumentTemplate(TenantAwareModel):
    """Document template"""
    
    class Type(models.TextChoices):
        WORD = 'word', 'Word Document'
        PDF = 'pdf', 'PDF'
        EXCEL = 'excel', 'Excel'
    
    name = models.CharField(max_length=255)
    description = models.TextField(blank=True)
    type = models.CharField(max_length=20, choices=Type.choices)
    
    # Template file
    file_url = models.URLField()
    
    # Template structure
    variables = models.JSONField(default=list, help_text="List of template variables")
    
    # Branding
    is_branded = models.BooleanField(default=False)
    
    # Usage
    is_public = models.BooleanField(default=False)
    usage_count = models.IntegerField(default=0)
    
    # Ownership
    created_by = models.ForeignKey('core.User', on_delete=models.SET_NULL, null=True)

class Document(TenantAwareModel):
    """Generated document"""
    
    class Status(models.TextChoices):
        PENDING = 'pending', 'Pending'
        PROCESSING = 'processing', 'Processing'
        COMPLETED = 'completed', 'Completed'
        FAILED = 'failed', 'Failed'
    
    class Format(models.TextChoices):
        DOCX = 'docx', 'Word Document'
        PDF = 'pdf', 'PDF'
        XLSX = 'xlsx', 'Excel'
    
    title = models.CharField(max_length=255)
    description = models.TextField(blank=True)
    
    # Generation
    template = models.ForeignKey(DocumentTemplate, on_delete=models.SET_NULL, null=True)
    format = models.CharField(max_length=10, choices=Format.choices)
    status = models.CharField(max_length=20, choices=Status.choices, default=Status.PENDING)
    
    # Content
    input_data = models.JSONField(help_text="User input for generation")
    generated_content = models.TextField(blank=True)
    
    # Files
    file_url = models.URLField(blank=True)
    file_size_bytes = models.BigIntegerField(default=0)
    
    # Generation metadata
    generation_time_seconds = models.FloatField(null=True)
    tokens_used = models.IntegerField(default=0)
    model_used = models.CharField(max_length=100, blank=True)
    
    # Error tracking
    error_message = models.TextField(blank=True)
    
    # Ownership
    created_by = models.ForeignKey('core.User', on_delete=models.CASCADE)
    
    # Sharing
    visibility = models.CharField(max_length=20, default='private')
    shared_with = models.ManyToManyField('core.User', related_name='shared_documents', blank=True)
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['tenant', 'created_by', '-created_at']),
            models.Index(fields=['tenant', 'status']),
        ]
```

---

### 2.4 Presentations App

**Purpose**: PowerPoint presentation generation

**API Endpoints**:
```
POST   /api/v1/presentations/generate/
GET    /api/v1/presentations/
GET    /api/v1/presentations/{id}/
DELETE /api/v1/presentations/{id}/
GET    /api/v1/presentations/{id}/download/
```

---

### 2.5 Knowledge Base App

**Purpose**: RAG system for document ingestion and retrieval

**Responsibilities**:
- Document upload and parsing
- Text chunking and embedding
- Vector storage and search
- Contextual retrieval

**API Endpoints**:
```
POST   /api/v1/kb/documents/upload/
GET    /api/v1/kb/documents/
GET    /api/v1/kb/documents/{id}/
DELETE /api/v1/kb/documents/{id}/
POST   /api/v1/kb/query/
GET    /api/v1/kb/search/
```

**Models**:
```python
# apps/knowledge_base/models.py
from django.db import models
from apps.core.models.base import TenantAwareModel

class KBDocument(TenantAwareModel):
    """Knowledge base document"""
    
    class Status(models.TextChoices):
        UPLOADING = 'uploading', 'Uploading'
        PROCESSING = 'processing', 'Processing'
        INDEXED = 'indexed', 'Indexed'
        FAILED = 'failed', 'Failed'
    
    title = models.CharField(max_length=255)
    file_name = models.CharField(max_length=255)
    file_url = models.URLField()
    file_type = models.CharField(max_length=50)
    file_size_bytes = models.BigIntegerField()
    
    status = models.CharField(max_length=20, choices=Status.choices, default=Status.UPLOADING)
    
    # Processing metadata
    total_pages = models.IntegerField(null=True)
    total_chunks = models.IntegerField(default=0)
    processing_time_seconds = models.FloatField(null=True)
    
    # Embedding metadata
    embedding_model = models.CharField(max_length=100, blank=True)
    vector_db_collection = models.CharField(max_length=255, blank=True)
    
    # Content
    extracted_text = models.TextField(blank=True)
    metadata = models.JSONField(default=dict)
    
    # Tags and categorization
    tags = models.JSONField(default=list)
    category = models.CharField(max_length=100, blank=True)
    
    # Ownership
    uploaded_by = models.ForeignKey('core.User', on_delete=models.CASCADE)
    
    # Error tracking
    error_message = models.TextField(blank=True)
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['tenant', 'status']),
            models.Index(fields=['tenant', 'uploaded_by', '-created_at']),
        ]

class KBChunk(models.Model):
    """Document chunk for vector search"""
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    document = models.ForeignKey(KBDocument, on_delete=models.CASCADE, related_name='chunks')
    
    # Content
    content = models.TextField()
    chunk_index = models.IntegerField()
    
    # Vector DB reference
    vector_db_id = models.CharField(max_length=255)
    
    # Metadata
    metadata = models.JSONField(default=dict)
    
    # Page reference
    page_number = models.IntegerField(null=True)
    
    class Meta:
        ordering = ['chunk_index']
        indexes = [
            models.Index(fields=['document', 'chunk_index']),
        ]

class KBQuery(TenantAwareModel):
    """Knowledge base query history"""
    
    query_text = models.TextField()
    response_text = models.TextField()
    
    # Retrieved chunks
    retrieved_chunks = models.JSONField(default=list)
    
    # Performance
    retrieval_time_seconds = models.FloatField()
    generation_time_seconds = models.FloatField()
    
    # Model info
    embedding_model = models.CharField(max_length=100)
    llm_model = models.CharField(max_length=100)
    
    # Tokens
    tokens_used = models.IntegerField(default=0)
    
    # User
    queried_by = models.ForeignKey('core.User', on_delete=models.CASCADE)
    
    class Meta:
        ordering = ['-created_at']
```

---

### 2.6 Agents App

**Purpose**: Agentic orchestration using LangGraph

**Key Components**:
```python
# apps/agents/orchestrator.py
from langgraph.graph import StateGraph, END
from typing import TypedDict, List
import logging

logger = logging.getLogger(__name__)

class AgentState(TypedDict):
    """Shared state across agents"""
    user_request: str
    tenant_id: str
    user_id: str
    
    # Execution flow
    current_step: str
    completed_steps: List[str]
    
    # Results
    document_id: str
    image_ids: List[str]
    kb_results: List[dict]
    
    # Error handling
    errors: List[str]
    retry_count: int

class AgentOrchestrator:
    """Orchestrates multiple AI agents"""
    
    def __init__(self):
        self.graph = self._build_graph()
    
    def _build_graph(self):
        """Build LangGraph workflow"""
        workflow = StateGraph(AgentState)
        
        # Add nodes (agents)
        workflow.add_node("parse_intent", self.parse_intent_node)
        workflow.add_node("document_agent", self.document_agent_node)
        workflow.add_node("kb_agent", self.kb_agent_node)
        workflow.add_node("image_agent", self.image_agent_node)
        workflow.add_node("validate", self.validate_node)
        
        # Define edges (flow)
        workflow.set_entry_point("parse_intent")
        
        workflow.add_conditional_edges(
            "parse_intent",
            self.route_request,
            {
                "document": "document_agent",
                "kb_query": "kb_agent",
                "image": "image_agent",
            }
        )
        
        workflow.add_edge("document_agent", "validate")
        workflow.add_edge("kb_agent", "validate")
        workflow.add_edge("image_agent", "validate")
        workflow.add_edge("validate", END)
        
        return workflow.compile()
    
    def parse_intent_node(self, state: AgentState):
        """Parse user intent"""
        # Use LLM to understand request
        logger.info(f"Parsing intent: {state['user_request']}")
        
        # Simplified - in reality, use LLM
        state['current_step'] = 'parsed'
        return state
    
    def route_request(self, state: AgentState):
        """Route to appropriate agent"""
        request = state['user_request'].lower()
        
        if 'document' in request or 'report' in request:
            return "document"
        elif 'search' in request or 'find' in request:
            return "kb_query"
        elif 'image' in request or 'logo' in request:
            return "image"
        
        return "document"  # default
    
    def document_agent_node(self, state: AgentState):
        """Document generation agent"""
        from apps.agents.agents.document_agent import DocumentAgent
        
        agent = DocumentAgent()
        result = agent.execute(state)
        
        state['document_id'] = result['document_id']
        state['completed_steps'].append('document_generation')
        
        return state
    
    def kb_agent_node(self, state: AgentState):
        """Knowledge base agent"""
        from apps.agents.agents.kb_agent import KBAgent
        
        agent = KBAgent()
        result = agent.execute(state)
        
        state['kb_results'] = result['results']
        state['completed_steps'].append('kb_retrieval')
        
        return state
    
    def image_agent_node(self, state: AgentState):
        """Image generation agent"""
        from apps.agents.agents.image_agent import ImageAgent
        
        agent = ImageAgent()
        result = agent.execute(state)
        
        state['image_ids'] = result['image_ids']
        state['completed_steps'].append('image_generation')
        
        return state
    
    def validate_node(self, state: AgentState):
        """Validate results"""
        logger.info(f"Validation complete: {state['completed_steps']}")
        return state
    
    def run(self, user_request: str, tenant_id: str, user_id: str):
        """Execute agent workflow"""
        initial_state = AgentState(
            user_request=user_request,
            tenant_id=tenant_id,
            user_id=user_id,
            current_step='init',
            completed_steps=[],
            document_id='',
            image_ids=[],
            kb_results=[],
            errors=[],
            retry_count=0
        )
        
        result = self.graph.invoke(initial_state)
        return result
```

---

## 3. API DESIGN PRINCIPLES

### 3.1 API Versioning

**Strategy**: URL-based versioning

```python
# config/urls.py
from django.urls import path, include

urlpatterns = [
    path('api/v1/', include('apps.api_v1.urls')),
    # Future: path('api/v2/', include('apps.api_v2.urls')),
]

# apps/api_v1/urls.py
urlpatterns = [
    path('auth/', include('apps.authentication.urls')),
    path('documents/', include('apps.documents.urls')),
    path('presentations/', include('apps.presentations.urls')),
    path('kb/', include('apps.knowledge_base.urls')),
    path('images/', include('apps.images.urls')),
    path('billing/', include('apps.billing.urls')),
    path('analytics/', include('apps.analytics.urls')),
    path('templates/', include('apps.templates.urls')),
]
```

### 3.2 Response Format

**Standard Success Response**:
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Document Title",
    ...
  },
  "meta": {
    "timestamp": "2025-12-28T10:00:00Z",
    "request_id": "req_abc123"
  }
}
```

**Standard Error Response**:
```json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_TOKENS",
    "message": "Insufficient token balance",
    "details": {
      "required": 100,
      "available": 50
    }
  },
  "meta": {
    "timestamp": "2025-12-28T10:00:00Z",
    "request_id": "req_abc123"
  }
}
```

**Pagination Response**:
```json
{
  "success": true,
  "data": [...],
  "pagination": {
    "total": 150,
    "page": 1,
    "page_size": 20,
    "total_pages": 8,
    "has_next": true,
    "has_previous": false
  },
  "meta": {
    "timestamp": "2025-12-28T10:00:00Z",
    "request_id": "req_abc123"
  }
}
```

### 3.3 HTTP Status Codes

| Code | Usage |
|------|-------|
| 200 | Successful GET, PUT, PATCH |
| 201 | Successful POST (resource created) |
| 204 | Successful DELETE (no content) |
| 400 | Bad request (validation error) |
| 401 | Unauthorized (authentication required) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Not found |
| 409 | Conflict (e.g., duplicate resource) |
| 422 | Unprocessable entity (semantic errors) |
| 429 | Too many requests (rate limited) |
| 500 | Internal server error |
| 503 | Service unavailable |

### 3.4 Standard Serializers

```python
# utils/serializers.py
from rest_framework import serializers

class BaseSerializer(serializers.ModelSerializer):
    """Base serializer with common fields"""
    
    created_at = serializers.DateTimeField(read_only=True)
    updated_at = serializers.DateTimeField(read_only=True)

class ResponseSerializer(serializers.Serializer):
    """Wrapper for consistent API responses"""
    
    success = serializers.BooleanField()
    data = serializers.DictField()
    meta = serializers.DictField()

class ErrorSerializer(serializers.Serializer):
    """Error response serializer"""
    
    code = serializers.CharField()
    message = serializers.CharField()
    details = serializers.DictField(required=False)
```

---

## 4. AUTHENTICATION & AUTHORIZATION

### 4.1 JWT Authentication

```python
# apps/authentication/services/jwt_service.py
from datetime import datetime, timedelta
from django.conf import settings
import jwt

class JWTService:
    """Handle JWT token operations"""
    
    @staticmethod
    def generate_access_token(user):
        """Generate access token (15 min expiry)"""
        payload = {
            'user_id': str(user.id),
            'tenant_id': str(user.tenant.id),
            'email': user.email,
            'role': user.role,
            'exp': datetime.utcnow() + timedelta(minutes=15),
            'iat': datetime.utcnow(),
            'type': 'access'
        }
        
        return jwt.encode(payload, settings.SECRET_KEY, algorithm='HS256')
    
    @staticmethod
    def generate_refresh_token(user):
        """Generate refresh token (7 days expiry)"""
        payload = {
            'user_id': str(user.id),
            'exp': datetime.utcnow() + timedelta(days=7),
            'iat': datetime.utcnow(),
            'type': 'refresh'
        }
        
        return jwt.encode(payload, settings.SECRET_KEY, algorithm='HS256')
    
    @staticmethod
    def decode_token(token):
        """Decode and validate token"""
        try:
            return jwt.decode(token, settings.SECRET_KEY, algorithms=['HS256'])
        except jwt.ExpiredSignatureError:
            raise ValueError("Token has expired")
        except jwt.InvalidTokenError:
            raise ValueError("Invalid token")
```

### 4.2 Custom Authentication Backend

```python
# apps/authentication/backends.py
from rest_framework import authentication, exceptions
from apps.core.models import User
from .services.jwt_service import JWTService

class JWTAuthentication(authentication.BaseAuthentication):
    """Custom JWT authentication"""
    
    def authenticate(self, request):
        auth_header = request.headers.get('Authorization')
        
        if not auth_header:
            return None
        
        try:
            # Expected format: "Bearer <token>"
            parts = auth_header.split()
            
            if len(parts) != 2 or parts[0].lower() != 'bearer':
                return None
            
            token = parts[1]
            payload = JWTService.decode_token(token)
            
            # Only accept access tokens
            if payload.get('type') != 'access':
                raise exceptions.AuthenticationFailed('Invalid token type')
            
            # Get user
            user = User.objects.get(id=payload['user_id'])
            
            return (user, payload)
            
        except ValueError as e:
            raise exceptions.AuthenticationFailed(str(e))
        except User.DoesNotExist:
            raise exceptions.AuthenticationFailed('User not found')
```

### 4.3 Settings Configuration

```python
# config/settings/base.py

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'apps.authentication.backends.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
    'EXCEPTION_HANDLER': 'apps.core.exceptions.custom_exception_handler',
}
```

---

## 5. DATABASE MODELS

*See previous documents for detailed models. Key principles:*

1. **All tenant-aware models** inherit from `TenantAwareModel`
2. **Use UUIDs** for primary keys
3. **Add indexes** on frequently queried fields
4. **Use choices** for enum-like fields
5. **Add metadata fields**: created_at, updated_at, created_by

---

## 6. SERVICES LAYER

### 6.1 Service Pattern

**Principle**: Keep views thin, business logic in services

```python
# apps/documents/services/document_generator.py
from docx import Document
from apps.documents.models import Document as DocModel
from services.llm.openai_service import OpenAIService
from services.storage.s3_storage import S3Storage

class DocumentGeneratorService:
    """Service for generating documents"""
    
    def __init__(self):
        self.llm_service = OpenAIService()
        self.storage = S3Storage()
    
    def generate_document(self, title: str, template_id: str, input_data: dict, user):
        """Generate document from template"""
        
        # 1. Create document record
        doc = DocModel.objects.create(
            tenant=user.tenant,
            title=title,
            template_id=template_id,
            input_data=input_data,
            format='docx',
            status='processing',
            created_by=user
        )
        
        try:
            # 2. Generate content with LLM
            content = self._generate_content(input_data)
            
            # 3. Create Word document
            docx_file = self._create_docx(title, content)
            
            # 4. Upload to storage
            file_url = self.storage.upload(docx_file, f"documents/{doc.id}.docx")
            
            # 5. Update document record
            doc.generated_content = content
            doc.file_url = file_url
            doc.status = 'completed'
            doc.save()
            
            return doc
            
        except Exception as e:
            doc.status = 'failed'
            doc.error_message = str(e)
            doc.save()
            raise
    
    def _generate_content(self, input_data: dict) -> str:
        """Generate content using LLM"""
        prompt = f"Generate a professional document based on: {input_data}"
        return self.llm_service.generate(prompt)
    
    def _create_docx(self, title: str, content: str):
        """Create Word document"""
        document = Document()
        document.add_heading(title, 0)
        document.add_paragraph(content)
        return document
```

---

## 7. BACKGROUND TASKS

### 7.1 Celery Configuration

```python
# config/celery.py
from celery import Celery
import os

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings.development')

app = Celery('flowspark_ai')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()

# config/settings/base.py
CELERY_BROKER_URL = 'redis://localhost:6379/0'
CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = 'UTC'
```

### 7.2 Example Tasks

```python
# apps/documents/tasks.py
from celery import shared_task
from apps.documents.services.document_generator import DocumentGeneratorService

@shared_task(bind=True, max_retries=3)
def generate_document_task(self, document_id):
    """Async document generation"""
    try:
        service = DocumentGeneratorService()
        service.generate_by_id(document_id)
    except Exception as exc:
        self.retry(exc=exc, countdown=60)
```

---

## 8. ERROR HANDLING & LOGGING

```python
# apps/core/exceptions.py
from rest_framework.views import exception_handler
from rest_framework.response import Response
import logging

logger = logging.getLogger(__name__)

def custom_exception_handler(exc, context):
    """Custom exception handler for consistent error responses"""
    
    response = exception_handler(exc, context)
    
    if response is not None:
        error_response = {
            'success': False,
            'error': {
                'code': exc.__class__.__name__,
                'message': str(exc),
                'details': response.data
            }
        }
        
        response.data = error_response
    
    # Log error
    logger.error(f"API Error: {exc}", exc_info=True)
    
    return response
```

---

**Document Version**: 1.0  
**Created**: December 28, 2025  
**Status**: Draft for Review

