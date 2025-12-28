fjk# FlowSpark AI - SaaS & Multi-Tenancy Architecture
## Enterprise-Grade Multi-Tenant Design

---

## TABLE OF CONTENTS
1. [Multi-Tenancy Strategy](#multi-tenancy-strategy)
2. [Tenant Isolation](#tenant-isolation)
3. [Data Security & Access Control](#data-security--access-control)
4. [Subscription & Billing](#subscription--billing)
5. [Feature Gating](#feature-gating)

---

## 1. MULTI-TENANCY STRATEGY

### 1.1 Approach Selection

We'll use a **Hybrid Multi-Tenant Architecture** combining:
- **Shared Database + Schema-Based Isolation** for core application data
- **Logical Partitioning** using tenant_id for row-level security
- **Separate Vector DB Collections** per tenant for knowledge base

### 1.2 Architecture Comparison

| Approach | Pros | Cons | Our Choice |
|----------|------|------|------------|
| **Database-per-Tenant** | Complete isolation, easy to backup/restore individual tenants | High resource overhead, difficult to manage at scale | ❌ Not chosen |
| **Schema-per-Tenant** | Good isolation, manageable at scale | Schema proliferation, migration complexity | ⚠️ Considered |
| **Shared Database + Row-Level Security** | Cost-efficient, easy to scale | Requires careful query filtering | ✅ **Primary approach** |
| **Hybrid (Shared + Isolated)** | Flexibility for different tenant tiers | Added complexity | ✅ **For premium tiers** |

### 1.3 Our Implementation Strategy

```
┌─────────────────────────────────────────────────────────────┐
│                    TENANT ARCHITECTURE                      │
└─────────────────────────────────────────────────────────────┘

FREE & STANDARD TIERS
────────────────────────────────────────────
Shared PostgreSQL Database
├── Row-level tenant_id filtering
├── Shared vector DB with tenant_id metadata
└── Shared object storage with tenant prefixes

ENTERPRISE TIER
────────────────────────────────────────────
Optional Dedicated Resources
├── Dedicated vector DB collection (performance)
├── Dedicated storage bucket (compliance)
└── Dedicated compute (SLA guarantees)

ON-PREMISE TIER
────────────────────────────────────────────
Fully Isolated Deployment
├── Customer's infrastructure
├── Single-tenant configuration
└── Full data sovereignty
```

---

## 2. TENANT ISOLATION

### 2.1 Database-Level Isolation

#### Tenant Model
```python
# core/models/tenant.py
from django.db import models
from django.utils.translation import gettext_lazy as _
import uuid

class Tenant(models.Model):
    """Organization/Tenant model"""
    
    class Status(models.TextChoices):
        ACTIVE = 'active', _('Active')
        SUSPENDED = 'suspended', _('Suspended')
        TRIAL = 'trial', _('Trial')
        CANCELLED = 'cancelled', _('Cancelled')
    
    class Tier(models.TextChoices):
        FREE = 'free', _('Free')
        STANDARD = 'standard', _('Standard')
        PROFESSIONAL = 'professional', _('Professional')
        ENTERPRISE = 'enterprise', _('Enterprise')
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    name = models.CharField(max_length=255)
    slug = models.SlugField(unique=True, max_length=100)
    
    # Subscription
    tier = models.CharField(max_length=20, choices=Tier.choices, default=Tier.FREE)
    status = models.CharField(max_length=20, choices=Status.choices, default=Status.TRIAL)
    
    # Settings
    max_users = models.IntegerField(default=5)
    max_storage_gb = models.IntegerField(default=10)
    custom_domain = models.CharField(max_length=255, blank=True, null=True)
    
    # Isolation options
    dedicated_vector_db = models.BooleanField(default=False)
    dedicated_storage = models.BooleanField(default=False)
    
    # Metadata
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    # Branding
    logo_url = models.URLField(blank=True, null=True)
    primary_color = models.CharField(max_length=7, default='#0066CC')
    
    def __str__(self):
        return self.name
```

#### Base Model with Tenant Isolation
```python
# core/models/base.py
from django.db import models
import uuid

class TenantAwareModel(models.Model):
    """Abstract base model that includes tenant foreign key"""
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    tenant = models.ForeignKey(
        'core.Tenant',
        on_delete=models.CASCADE,
        related_name='%(class)s_set'
    )
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True
        indexes = [
            models.Index(fields=['tenant', '-created_at']),
        ]
```

#### Tenant-Aware Manager
```python
# core/managers.py
from django.db import models

class TenantManager(models.Manager):
    """Manager that automatically filters by tenant"""
    
    def __init__(self, *args, **kwargs):
        self._tenant_id = None
        super().__init__(*args, **kwargs)
    
    def get_queryset(self):
        qs = super().get_queryset()
        if self._tenant_id:
            return qs.filter(tenant_id=self._tenant_id)
        return qs
    
    def for_tenant(self, tenant):
        """Return queryset filtered for specific tenant"""
        clone = self._clone()
        clone._tenant_id = tenant.id if hasattr(tenant, 'id') else tenant
        return clone
```

### 2.2 Middleware for Tenant Context

```python
# core/middleware/tenant.py
from django.utils.deprecation import MiddlewareMixin
from django.http import JsonResponse
from core.models import Tenant
import threading

# Thread-local storage for tenant context
_thread_locals = threading.local()

def get_current_tenant():
    """Get current tenant from thread-local storage"""
    return getattr(_thread_locals, 'tenant', None)

def set_current_tenant(tenant):
    """Set current tenant in thread-local storage"""
    _thread_locals.tenant = tenant

class TenantMiddleware(MiddlewareMixin):
    """
    Middleware to identify and set current tenant from:
    1. Custom subdomain (tenant.flowspark.ai)
    2. Custom domain (customdomain.com)
    3. JWT token tenant claim
    """
    
    def process_request(self, request):
        tenant = None
        
        # Method 1: Subdomain-based tenant identification
        host = request.get_host().split(':')[0]
        parts = host.split('.')
        
        if len(parts) >= 3:  # e.g., acme.flowspark.ai
            subdomain = parts[0]
            try:
                tenant = Tenant.objects.get(slug=subdomain, status=Tenant.Status.ACTIVE)
            except Tenant.DoesNotExist:
                pass
        
        # Method 2: Custom domain
        if not tenant:
            try:
                tenant = Tenant.objects.get(custom_domain=host, status=Tenant.Status.ACTIVE)
            except Tenant.DoesNotExist:
                pass
        
        # Method 3: From authenticated user's tenant
        if not tenant and request.user.is_authenticated:
            tenant = getattr(request.user, 'tenant', None)
        
        # Method 4: From X-Tenant-ID header (for API calls)
        if not tenant:
            tenant_id = request.headers.get('X-Tenant-ID')
            if tenant_id:
                try:
                    tenant = Tenant.objects.get(id=tenant_id, status=Tenant.Status.ACTIVE)
                except Tenant.DoesNotExist:
                    pass
        
        if tenant:
            set_current_tenant(tenant)
            request.tenant = tenant
        else:
            # For public endpoints, no tenant required
            request.tenant = None
        
        return None
    
    def process_response(self, request, response):
        # Clear tenant context
        set_current_tenant(None)
        return response
```

### 2.3 Tenant-Aware Views & Serializers

```python
# core/mixins.py
from rest_framework import exceptions
from .middleware.tenant import get_current_tenant

class TenantRequiredMixin:
    """Mixin to enforce tenant context in views"""
    
    def get_queryset(self):
        tenant = get_current_tenant()
        if not tenant:
            raise exceptions.PermissionDenied("Tenant context required")
        
        queryset = super().get_queryset()
        
        # Automatically filter by tenant if model has tenant field
        if hasattr(queryset.model, 'tenant'):
            return queryset.filter(tenant=tenant)
        
        return queryset
    
    def perform_create(self, serializer):
        tenant = get_current_tenant()
        if not tenant:
            raise exceptions.PermissionDenied("Tenant context required")
        
        serializer.save(tenant=tenant)
```

### 2.4 Vector Database Isolation

```python
# services/vector_db.py
from chromadb import Client
from chromadb.config import Settings
from .middleware.tenant import get_current_tenant

class TenantVectorDB:
    """Tenant-aware vector database service"""
    
    def __init__(self):
        self.client = Client(Settings(
            chroma_db_impl="duckdb+parquet",
            persist_directory="./chromadb"
        ))
    
    def get_collection(self, collection_name: str):
        """Get or create tenant-specific collection"""
        tenant = get_current_tenant()
        if not tenant:
            raise ValueError("Tenant context required")
        
        # Namespace collection by tenant
        tenant_collection_name = f"{tenant.id}_{collection_name}"
        
        return self.client.get_or_create_collection(
            name=tenant_collection_name,
            metadata={"tenant_id": str(tenant.id)}
        )
    
    def add_documents(self, collection_name: str, documents: list, embeddings: list, metadatas: list):
        """Add documents with automatic tenant tagging"""
        tenant = get_current_tenant()
        collection = self.get_collection(collection_name)
        
        # Add tenant_id to all metadata
        for metadata in metadatas:
            metadata['tenant_id'] = str(tenant.id)
        
        collection.add(
            documents=documents,
            embeddings=embeddings,
            metadatas=metadatas,
            ids=[f"{tenant.id}_{i}" for i in range(len(documents))]
        )
    
    def query(self, collection_name: str, query_embedding: list, n_results: int = 5):
        """Query with automatic tenant filtering"""
        collection = self.get_collection(collection_name)
        
        results = collection.query(
            query_embeddings=[query_embedding],
            n_results=n_results
        )
        
        return results
```

### 2.5 Object Storage Isolation

```python
# services/storage.py
import boto3
from django.conf import settings
from .middleware.tenant import get_current_tenant

class TenantStorage:
    """Tenant-aware S3-compatible storage"""
    
    def __init__(self):
        self.s3_client = boto3.client(
            's3',
            endpoint_url=settings.S3_ENDPOINT_URL,
            aws_access_key_id=settings.S3_ACCESS_KEY,
            aws_secret_access_key=settings.S3_SECRET_KEY
        )
        self.bucket_name = settings.S3_BUCKET_NAME
    
    def _get_tenant_prefix(self):
        """Get tenant-specific prefix for object keys"""
        tenant = get_current_tenant()
        if not tenant:
            raise ValueError("Tenant context required")
        return f"tenants/{tenant.id}/"
    
    def upload_file(self, file_obj, file_path: str):
        """Upload file with tenant prefix"""
        tenant_prefix = self._get_tenant_prefix()
        object_key = f"{tenant_prefix}{file_path}"
        
        self.s3_client.upload_fileobj(
            file_obj,
            self.bucket_name,
            object_key
        )
        
        return object_key
    
    def download_file(self, file_path: str):
        """Download file ensuring tenant isolation"""
        tenant_prefix = self._get_tenant_prefix()
        object_key = f"{tenant_prefix}{file_path}"
        
        response = self.s3_client.get_object(
            Bucket=self.bucket_name,
            Key=object_key
        )
        
        return response['Body'].read()
    
    def list_files(self, prefix: str = ""):
        """List files for current tenant"""
        tenant_prefix = self._get_tenant_prefix()
        full_prefix = f"{tenant_prefix}{prefix}"
        
        response = self.s3_client.list_objects_v2(
            Bucket=self.bucket_name,
            Prefix=full_prefix
        )
        
        return response.get('Contents', [])
```

---

## 3. DATA SECURITY & ACCESS CONTROL

### 3.1 Role-Based Access Control (RBAC)

```python
# core/models/rbac.py
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    """Extended user model with tenant relationship"""
    
    class Role(models.TextChoices):
        OWNER = 'owner', 'Owner'
        ADMIN = 'admin', 'Admin'
        MANAGER = 'manager', 'Manager'
        MEMBER = 'member', 'Member'
        VIEWER = 'viewer', 'Viewer'
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    tenant = models.ForeignKey('core.Tenant', on_delete=models.CASCADE, related_name='users')
    role = models.CharField(max_length=20, choices=Role.choices, default=Role.MEMBER)
    
    # Profile
    avatar_url = models.URLField(blank=True, null=True)
    phone = models.CharField(max_length=20, blank=True, null=True)
    department = models.CharField(max_length=100, blank=True, null=True)
    
    # Preferences
    timezone = models.CharField(max_length=50, default='UTC')
    language = models.CharField(max_length=10, default='en')
    
    # Status
    is_active = models.BooleanField(default=True)
    email_verified = models.BooleanField(default=False)
    
    def has_permission(self, permission: str) -> bool:
        """Check if user has specific permission"""
        return permission in ROLE_PERMISSIONS.get(self.role, [])

# Permission definitions
ROLE_PERMISSIONS = {
    'owner': [
        'tenant.manage',
        'user.manage',
        'document.create', 'document.read', 'document.update', 'document.delete',
        'template.create', 'template.read', 'template.update', 'template.delete',
        'kb.create', 'kb.read', 'kb.update', 'kb.delete',
        'billing.manage',
        'analytics.view',
    ],
    'admin': [
        'user.manage',
        'document.create', 'document.read', 'document.update', 'document.delete',
        'template.create', 'template.read', 'template.update', 'template.delete',
        'kb.create', 'kb.read', 'kb.update', 'kb.delete',
        'analytics.view',
    ],
    'manager': [
        'document.create', 'document.read', 'document.update',
        'template.read',
        'kb.create', 'kb.read',
        'analytics.view',
    ],
    'member': [
        'document.create', 'document.read',
        'template.read',
        'kb.read',
    ],
    'viewer': [
        'document.read',
        'template.read',
    ],
}
```

### 3.2 Permission Decorators

```python
# core/permissions.py
from rest_framework import permissions
from functools import wraps
from django.http import JsonResponse

class HasPermission(permissions.BasePermission):
    """DRF permission class for checking user permissions"""
    
    def __init__(self, required_permission):
        self.required_permission = required_permission
    
    def has_permission(self, request, view):
        if not request.user.is_authenticated:
            return False
        
        return request.user.has_permission(self.required_permission)

def require_permission(permission: str):
    """Decorator for view functions"""
    def decorator(view_func):
        @wraps(view_func)
        def wrapped(request, *args, **kwargs):
            if not request.user.is_authenticated:
                return JsonResponse({'error': 'Authentication required'}, status=401)
            
            if not request.user.has_permission(permission):
                return JsonResponse({'error': 'Permission denied'}, status=403)
            
            return view_func(request, *args, **kwargs)
        return wrapped
    return decorator
```

### 3.3 Resource-Level Permissions

```python
# core/models/document.py
from django.db import models
from .base import TenantAwareModel

class Document(TenantAwareModel):
    """Document model with resource-level permissions"""
    
    class Visibility(models.TextChoices):
        PRIVATE = 'private', 'Private'
        TEAM = 'team', 'Team'
        ORGANIZATION = 'organization', 'Organization'
    
    title = models.CharField(max_length=255)
    content = models.TextField()
    file_url = models.URLField()
    
    # Ownership
    created_by = models.ForeignKey('core.User', on_delete=models.CASCADE, related_name='created_documents')
    
    # Access control
    visibility = models.CharField(max_length=20, choices=Visibility.choices, default=Visibility.PRIVATE)
    shared_with = models.ManyToManyField('core.User', related_name='shared_documents', blank=True)
    
    def user_can_access(self, user):
        """Check if user can access this document"""
        # Owner can always access
        if self.created_by == user:
            return True
        
        # Check visibility
        if self.visibility == self.Visibility.ORGANIZATION:
            return user.tenant == self.tenant
        
        if self.visibility == self.Visibility.TEAM:
            return user.department == self.created_by.department
        
        # Check explicit sharing
        return self.shared_with.filter(id=user.id).exists()
```

### 3.4 Data Encryption

```python
# core/encryption.py
from cryptography.fernet import Fernet
from django.conf import settings
import base64

class FieldEncryption:
    """Encrypt sensitive fields in database"""
    
    def __init__(self):
        self.cipher_suite = Fernet(settings.ENCRYPTION_KEY.encode())
    
    def encrypt(self, plaintext: str) -> str:
        """Encrypt string"""
        if not plaintext:
            return plaintext
        
        encrypted = self.cipher_suite.encrypt(plaintext.encode())
        return base64.urlsafe_b64encode(encrypted).decode()
    
    def decrypt(self, ciphertext: str) -> str:
        """Decrypt string"""
        if not ciphertext:
            return ciphertext
        
        encrypted = base64.urlsafe_b64decode(ciphertext.encode())
        decrypted = self.cipher_suite.decrypt(encrypted)
        return decrypted.decode()

# Usage in models
class EncryptedTextField(models.TextField):
    """Custom field that encrypts data"""
    
    def __init__(self, *args, **kwargs):
        self.encryption = FieldEncryption()
        super().__init__(*args, **kwargs)
    
    def get_prep_value(self, value):
        if value is None:
            return value
        return self.encryption.encrypt(value)
    
    def from_db_value(self, value, expression, connection):
        if value is None:
            return value
        return self.encryption.decrypt(value)
```

---

## 4. SUBSCRIPTION & BILLING

### 4.1 Subscription Model

```python
# billing/models.py
from django.db import models
from core.models.base import TenantAwareModel
import uuid

class SubscriptionPlan(models.Model):
    """Available subscription plans"""
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    name = models.CharField(max_length=100)
    slug = models.SlugField(unique=True)
    tier = models.CharField(max_length=20, choices=Tenant.Tier.choices)
    
    # Pricing
    price_monthly = models.DecimalField(max_digits=10, decimal_places=2)
    price_yearly = models.DecimalField(max_digits=10, decimal_places=2)
    
    # Limits
    max_users = models.IntegerField()
    max_documents_per_month = models.IntegerField()
    max_storage_gb = models.IntegerField()
    included_tokens = models.IntegerField(help_text="Tokens included per month")
    
    # Features
    features = models.JSONField(default=dict)
    
    # Stripe integration
    stripe_price_id_monthly = models.CharField(max_length=100, blank=True)
    stripe_price_id_yearly = models.CharField(max_length=100, blank=True)
    
    is_active = models.BooleanField(default=True)
    
    def __str__(self):
        return self.name

class Subscription(TenantAwareModel):
    """Tenant subscription"""
    
    class Status(models.TextChoices):
        TRIAL = 'trialing', 'Trial'
        ACTIVE = 'active', 'Active'
        PAST_DUE = 'past_due', 'Past Due'
        CANCELLED = 'canceled', 'Cancelled'
        UNPAID = 'unpaid', 'Unpaid'
    
    class Interval(models.TextChoices):
        MONTHLY = 'monthly', 'Monthly'
        YEARLY = 'yearly', 'Yearly'
    
    plan = models.ForeignKey(SubscriptionPlan, on_delete=models.PROTECT)
    status = models.CharField(max_length=20, choices=Status.choices, default=Status.TRIAL)
    interval = models.CharField(max_length=20, choices=Interval.choices, default=Interval.MONTHLY)
    
    # Stripe
    stripe_subscription_id = models.CharField(max_length=100, blank=True)
    stripe_customer_id = models.CharField(max_length=100, blank=True)
    
    # Dates
    trial_end = models.DateTimeField(null=True, blank=True)
    current_period_start = models.DateTimeField()
    current_period_end = models.DateTimeField()
    cancelled_at = models.DateTimeField(null=True, blank=True)
    
    # Usage tracking
    documents_used_this_period = models.IntegerField(default=0)
    storage_used_gb = models.FloatField(default=0.0)
    
    def is_active(self):
        return self.status in [self.Status.ACTIVE, self.Status.TRIAL]
    
    def has_reached_document_limit(self):
        return self.documents_used_this_period >= self.plan.max_documents_per_month
```

### 4.2 Token System

```python
# billing/models.py (continued)

class TokenPackage(models.Model):
    """Token top-up packages"""
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    name = models.CharField(max_length=100)
    tokens = models.IntegerField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    
    stripe_price_id = models.CharField(max_length=100, blank=True)
    
    is_active = models.BooleanField(default=True)
    
    def __str__(self):
        return f"{self.name} - {self.tokens} tokens"

class TokenBalance(TenantAwareModel):
    """Tenant token balance"""
    
    balance = models.IntegerField(default=0)
    lifetime_purchased = models.IntegerField(default=0)
    lifetime_used = models.IntegerField(default=0)
    
    # Alert settings
    low_balance_threshold = models.IntegerField(default=1000)
    low_balance_alert_sent = models.BooleanField(default=False)
    
    def deduct(self, amount: int, operation: str):
        """Deduct tokens and create transaction record"""
        if self.balance < amount:
            raise ValueError("Insufficient token balance")
        
        self.balance -= amount
        self.lifetime_used += amount
        self.save()
        
        # Create transaction record
        TokenTransaction.objects.create(
            tenant=self.tenant,
            amount=-amount,
            operation=operation,
            balance_after=self.balance
        )
    
    def add(self, amount: int, source: str = 'purchase'):
        """Add tokens"""
        self.balance += amount
        if source == 'purchase':
            self.lifetime_purchased += amount
        self.save()
        
        TokenTransaction.objects.create(
            tenant=self.tenant,
            amount=amount,
            operation=f'token_{source}',
            balance_after=self.balance
        )

class TokenTransaction(TenantAwareModel):
    """Token usage history"""
    
    amount = models.IntegerField(help_text="Positive for additions, negative for usage")
    operation = models.CharField(max_length=100, help_text="e.g., 'document_generation', 'token_purchase'")
    balance_after = models.IntegerField()
    
    # Metadata
    metadata = models.JSONField(default=dict, help_text="Additional context like document_id, model used, etc.")
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['tenant', '-created_at']),
        ]
```

### 4.3 Token Cost Configuration

```python
# billing/costs.py

TOKEN_COSTS = {
    # Document operations
    'document_generation_word': 100,
    'document_generation_pdf': 100,
    'document_generation_excel': 80,
    
    # Presentation
    'presentation_generation': 200,
    
    # Image generation
    'image_generation_standard': 50,
    'image_generation_hd': 100,
    'logo_generation': 75,
    
    # Knowledge base
    'document_upload_per_page': 5,
    'kb_query': 10,
    'advanced_rag_query': 25,
    
    # Analysis
    'document_analysis': 50,
    'data_analysis': 100,
}

def calculate_operation_cost(operation_type: str, **kwargs) -> int:
    """Calculate token cost for an operation"""
    base_cost = TOKEN_COSTS.get(operation_type, 0)
    
    # Dynamic cost adjustments
    if operation_type == 'document_upload_per_page':
        pages = kwargs.get('pages', 1)
        return base_cost * pages
    
    if operation_type in ['document_generation_word', 'document_generation_pdf']:
        # Cost scales with length
        word_count = kwargs.get('word_count', 1000)
        if word_count > 5000:
            return base_cost * 2
        elif word_count > 10000:
            return base_cost * 3
    
    return base_cost
```

### 4.4 Stripe Integration

```python
# billing/services/stripe_service.py
import stripe
from django.conf import settings
from billing.models import Subscription, TokenBalance

stripe.api_key = settings.STRIPE_SECRET_KEY

class StripeService:
    """Handle Stripe operations"""
    
    def create_customer(self, tenant, email: str):
        """Create Stripe customer"""
        customer = stripe.Customer.create(
            email=email,
            metadata={'tenant_id': str(tenant.id), 'tenant_name': tenant.name}
        )
        return customer.id
    
    def create_subscription(self, tenant, plan, interval: str):
        """Create subscription"""
        subscription = Subscription.objects.get(tenant=tenant)
        
        # Get Stripe price ID based on interval
        price_id = (
            plan.stripe_price_id_monthly if interval == 'monthly'
            else plan.stripe_price_id_yearly
        )
        
        # Create Stripe subscription
        stripe_sub = stripe.Subscription.create(
            customer=subscription.stripe_customer_id,
            items=[{'price': price_id}],
            trial_period_days=14
        )
        
        # Update local subscription
        subscription.stripe_subscription_id = stripe_sub.id
        subscription.plan = plan
        subscription.interval = interval
        subscription.status = Subscription.Status.TRIAL
        subscription.current_period_start = stripe_sub.current_period_start
        subscription.current_period_end = stripe_sub.current_period_end
        subscription.trial_end = stripe_sub.trial_end
        subscription.save()
        
        return subscription
    
    def handle_webhook(self, event):
        """Process Stripe webhook events"""
        event_type = event['type']
        
        if event_type == 'customer.subscription.updated':
            self._handle_subscription_updated(event['data']['object'])
        
        elif event_type == 'customer.subscription.deleted':
            self._handle_subscription_deleted(event['data']['object'])
        
        elif event_type == 'invoice.payment_succeeded':
            self._handle_payment_succeeded(event['data']['object'])
        
        elif event_type == 'invoice.payment_failed':
            self._handle_payment_failed(event['data']['object'])
    
    def _handle_subscription_updated(self, stripe_subscription):
        """Update local subscription from Stripe"""
        subscription = Subscription.objects.get(
            stripe_subscription_id=stripe_subscription['id']
        )
        
        subscription.status = stripe_subscription['status']
        subscription.current_period_start = stripe_subscription['current_period_start']
        subscription.current_period_end = stripe_subscription['current_period_end']
        subscription.save()
    
    def purchase_tokens(self, tenant, package):
        """Purchase token package"""
        # Create Stripe payment intent
        intent = stripe.PaymentIntent.create(
            amount=int(package.price * 100),  # Convert to cents
            currency='usd',
            customer=Subscription.objects.get(tenant=tenant).stripe_customer_id,
            metadata={'tenant_id': str(tenant.id), 'package_id': str(package.id)}
        )
        
        return intent.client_secret
```

---

## 5. FEATURE GATING

### 5.1 Feature Flags by Plan

```python
# core/features.py

PLAN_FEATURES = {
    'free': {
        'max_users': 3,
        'max_documents_per_month': 10,
        'max_storage_gb': 1,
        'document_generation': True,
        'presentation_generation': False,
        'image_generation': False,
        'knowledge_base': True,
        'knowledge_base_max_docs': 10,
        'advanced_rag': False,
        'custom_templates': False,
        'api_access': False,
        'sso': False,
        'analytics': False,
        'audit_logs': False,
        'priority_support': False,
    },
    'standard': {
        'max_users': 10,
        'max_documents_per_month': 100,
        'max_storage_gb': 10,
        'document_generation': True,
        'presentation_generation': True,
        'image_generation': True,
        'knowledge_base': True,
        'knowledge_base_max_docs': 100,
        'advanced_rag': False,
        'custom_templates': True,
        'api_access': True,
        'sso': False,
        'analytics': True,
        'audit_logs': False,
        'priority_support': False,
    },
    'professional': {
        'max_users': 50,
        'max_documents_per_month': 500,
        'max_storage_gb': 50,
        'document_generation': True,
        'presentation_generation': True,
        'image_generation': True,
        'knowledge_base': True,
        'knowledge_base_max_docs': 1000,
        'advanced_rag': True,
        'custom_templates': True,
        'api_access': True,
        'sso': True,
        'analytics': True,
        'audit_logs': True,
        'priority_support': True,
    },
    'enterprise': {
        'max_users': -1,  # Unlimited
        'max_documents_per_month': -1,
        'max_storage_gb': 500,
        'document_generation': True,
        'presentation_generation': True,
        'image_generation': True,
        'knowledge_base': True,
        'knowledge_base_max_docs': -1,
        'advanced_rag': True,
        'custom_templates': True,
        'api_access': True,
        'sso': True,
        'analytics': True,
        'audit_logs': True,
        'priority_support': True,
        'dedicated_resources': True,
        'on_premise_option': True,
        'custom_sla': True,
    },
}

def tenant_has_feature(tenant, feature_name: str) -> bool:
    """Check if tenant's plan includes a feature"""
    tier = tenant.tier
    features = PLAN_FEATURES.get(tier, {})
    return features.get(feature_name, False)

def require_feature(feature_name: str):
    """Decorator to check feature access"""
    def decorator(view_func):
        @wraps(view_func)
        def wrapped(request, *args, **kwargs):
            tenant = get_current_tenant()
            if not tenant:
                return JsonResponse({'error': 'Tenant context required'}, status=400)
            
            if not tenant_has_feature(tenant, feature_name):
                return JsonResponse({
                    'error': 'Feature not available in your plan',
                    'feature': feature_name,
                    'upgrade_required': True
                }, status=403)
            
            return view_func(request, *args, **kwargs)
        return wrapped
    return decorator
```

### 5.2 Usage Limits Enforcement

```python
# core/limiters.py
from billing.models import Subscription, TokenBalance
from .middleware.tenant import get_current_tenant
from .features import PLAN_FEATURES

class UsageLimiter:
    """Enforce plan limits"""
    
    @staticmethod
    def check_document_limit():
        """Check if tenant can create more documents this period"""
        tenant = get_current_tenant()
        subscription = Subscription.objects.get(tenant=tenant)
        
        plan_features = PLAN_FEATURES[tenant.tier]
        max_docs = plan_features['max_documents_per_month']
        
        # Unlimited
        if max_docs == -1:
            return True
        
        # Check current usage
        if subscription.documents_used_this_period >= max_docs:
            return False
        
        return True
    
    @staticmethod
    def check_storage_limit(file_size_mb: float):
        """Check if file upload would exceed storage limit"""
        tenant = get_current_tenant()
        subscription = Subscription.objects.get(tenant=tenant)
        
        plan_features = PLAN_FEATURES[tenant.tier]
        max_storage_gb = plan_features['max_storage_gb']
        
        current_storage_gb = subscription.storage_used_gb
        new_storage_gb = current_storage_gb + (file_size_mb / 1024)
        
        return new_storage_gb <= max_storage_gb
    
    @staticmethod
    def check_token_balance(required_tokens: int):
        """Check if tenant has enough tokens"""
        tenant = get_current_tenant()
        token_balance = TokenBalance.objects.get(tenant=tenant)
        
        return token_balance.balance >= required_tokens
```

---

## 6. SECURITY BEST PRACTICES

### 6.1 Security Checklist

- [x] **Tenant Isolation**: Row-level filtering on all queries
- [x] **Input Validation**: All user inputs validated and sanitized
- [x] **Authentication**: JWT tokens with refresh mechanism
- [x] **Authorization**: RBAC with resource-level permissions
- [x] **Encryption**: Sensitive data encrypted at rest
- [x] **HTTPS Only**: All communication over TLS
- [x] **Rate Limiting**: API rate limits per tenant
- [x] **Audit Logging**: All critical actions logged
- [x] **SQL Injection Protection**: Using ORM, parameterized queries
- [x] **XSS Protection**: Content Security Policy, input sanitization
- [x] **CSRF Protection**: CSRF tokens on state-changing requests

### 6.2 Compliance Readiness

**GDPR Compliance**
- Right to access (data export)
- Right to erasure (account deletion)
- Data portability
- Consent management

**SOC 2 Type II**
- Access controls
- Encryption
- Incident response
- Change management

---

**Document Version**: 1.0  
**Created**: December 28, 2025  
**Status**: Draft for Review

