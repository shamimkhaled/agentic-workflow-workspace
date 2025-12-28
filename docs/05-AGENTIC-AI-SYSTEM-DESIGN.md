# FlowSpark AI - Agentic AI System Design
## Multi-Agent Architecture, Communication & Memory Management

---

## TABLE OF CONTENTS
1. [Agent Types & Responsibilities](#agent-types--responsibilities)
2. [Agent Communication & Orchestration](#agent-communication--orchestration)
3. [Memory Management System](#memory-management-system)
4. [LLM Integration Strategy](#llm-integration-strategy)
5. [RAG Pipeline Architecture](#rag-pipeline-architecture)
6. [Agent Error Handling & Recovery](#agent-error-handling--recovery)

---

## 1. AGENT TYPES & RESPONSIBILITIES

### 1.1 Agent Hierarchy

```
┌─────────────────────────────────────────────────────────┐
│           ORCHESTRATOR AGENT (Master)                   │
│  - Receives all user requests                           │
│  - Parses intent and decomposes tasks                   │
│  - Routes to specialized agents                         │
│  - Coordinates multi-agent workflows                    │
│  - Aggregates results and validates                     │
└────────────┬────────────────────────────────────────────┘
             │
    ┌────────┴─────────┬──────────────┬───────────────┐
    │                  │              │               │
┌───▼───────────┐ ┌───▼──────────┐ ┌─▼─────────────┐ ┌▼──────────────┐
│  DOCUMENT     │ │ PRESENTATION │ │ KNOWLEDGE     │ │ IMAGE         │
│  AGENT        │ │ AGENT        │ │ BASE AGENT    │ │ AGENT         │
│               │ │              │ │ (RAG)         │ │               │
│ - Generates   │ │ - Creates    │ │ - Ingests     │ │ - Generates   │
│   Word/PDF    │ │   PPTs       │ │   documents   │ │   images      │
│ - Templates   │ │ - Slide      │ │ - Retrieves   │ │ - Logos       │
│ - Formatting  │ │   layouts    │ │   context     │ │ - Diagrams    │
└───────────────┘ └──────────────┘ └───────────────┘ └───────────────┘
         │                │               │                   │
    ┌────▼────────────────▼───────────────▼───────────────────▼────┐
    │                    ANALYSIS AGENT                            │
    │  - Analyzes data and documents                               │
    │  - Generates insights and recommendations                    │
    │  - Creates charts and visualizations                         │
    └──────────────────────────────────────────────────────────────┘
         │
    ┌────▼────────────────────────────────────────────────────────┐
    │               TEMPLATE & BRAND AGENT                        │
    │  - Enforces brand guidelines                                 │
    │  - Manages templates                                         │
    │  - Validates outputs for compliance                          │
    └──────────────────────────────────────────────────────────────┘
```

---

### 1.2 Base Agent Interface

```python
# apps/agents/agents/base_agent.py
from abc import ABC, abstractmethod
from typing import Dict, Any, List
import logging
from datetime import datetime

logger = logging.getLogger(__name__)

class BaseAgent(ABC):
    """Abstract base class for all agents"""
    
    def __init__(self, name: str, description: str):
        self.name = name
        self.description = description
        self.memory = AgentMemory()
        self.execution_history = []
    
    @abstractmethod
    async def execute(self, state: Dict[str, Any]) -> Dict[str, Any]:
        """Execute agent logic"""
        pass
    
    @abstractmethod
    def can_handle(self, request: str) -> bool:
        """Determine if agent can handle the request"""
        pass
    
    def log_execution(self, input_data: Dict, output_data: Dict, duration: float):
        """Log execution for monitoring"""
        self.execution_history.append({
            'timestamp': datetime.utcnow(),
            'input': input_data,
            'output': output_data,
            'duration_seconds': duration
        })
        
        logger.info(f"{self.name} executed in {duration:.2f}s")
    
    def get_capabilities(self) -> List[str]:
        """Return list of agent capabilities"""
        return []

class AgentMemory:
    """Memory storage for agents"""
    
    def __init__(self):
        self.short_term = {}  # Conversation context
        self.long_term = []   # Historical interactions
    
    def store_short_term(self, key: str, value: Any):
        """Store in short-term memory"""
        self.short_term[key] = value
    
    def retrieve_short_term(self, key: str) -> Any:
        """Retrieve from short-term memory"""
        return self.short_term.get(key)
    
    def store_long_term(self, interaction: Dict):
        """Store in long-term memory"""
        self.long_term.append({
            'timestamp': datetime.utcnow(),
            'data': interaction
        })
    
    def clear_short_term(self):
        """Clear short-term memory"""
        self.short_term = {}
```

---

### 1.3 Specialized Agents

#### Document Agent

```python
# apps/agents/agents/document_agent.py
from .base_agent import BaseAgent
from apps.documents.services.document_generator import DocumentGeneratorService
from apps.knowledge_base.services.retrieval_service import RetrievalService
from services.llm.openai_service import OpenAIService
import asyncio

class DocumentAgent(BaseAgent):
    """Agent responsible for document generation"""
    
    def __init__(self):
        super().__init__(
            name="DocumentAgent",
            description="Generates Word and PDF documents from templates and AI content"
        )
        self.doc_service = DocumentGeneratorService()
        self.retrieval_service = RetrievalService()
        self.llm = OpenAIService()
    
    def can_handle(self, request: str) -> bool:
        """Check if request is for document generation"""
        keywords = ['document', 'report', 'word', 'pdf', 'generate document']
        return any(keyword in request.lower() for keyword in keywords)
    
    async def execute(self, state: Dict[str, Any]) -> Dict[str, Any]:
        """Execute document generation"""
        start_time = datetime.utcnow()
        
        # Extract requirements
        user_request = state.get('user_request', '')
        tenant_id = state.get('tenant_id')
        user_id = state.get('user_id')
        
        # Step 1: Check if knowledge base retrieval is needed
        needs_kb_context = self._needs_kb_context(user_request)
        context = ""
        
        if needs_kb_context:
            # Retrieve relevant context from KB
            kb_results = await self.retrieval_service.retrieve(
                query=user_request,
                tenant_id=tenant_id,
                top_k=3
            )
            context = self._format_kb_context(kb_results)
        
        # Step 2: Generate content with LLM
        content = await self._generate_content(user_request, context)
        
        # Step 3: Select or create template
        template = self._select_template(state)
        
        # Step 4: Generate document
        document = self.doc_service.create_document(
            title=self._extract_title(user_request),
            content=content,
            template=template,
            format='docx',
            tenant_id=tenant_id,
            user_id=user_id
        )
        
        # Log execution
        duration = (datetime.utcnow() - start_time).total_seconds()
        self.log_execution(state, {'document_id': str(document.id)}, duration)
        
        return {
            'document_id': str(document.id),
            'document_url': document.file_url,
            'status': 'completed',
            'tokens_used': content.get('tokens_used', 0)
        }
    
    def _needs_kb_context(self, request: str) -> bool:
        """Determine if KB context is needed"""
        kb_keywords = ['based on', 'using our', 'from documents', 'previous']
        return any(keyword in request.lower() for keyword in kb_keywords)
    
    async def _generate_content(self, request: str, context: str = "") -> Dict:
        """Generate content using LLM"""
        prompt = self._build_prompt(request, context)
        
        response = await self.llm.generate(
            prompt=prompt,
            max_tokens=2000,
            temperature=0.7
        )
        
        return response
    
    def _build_prompt(self, request: str, context: str) -> str:
        """Build prompt for LLM"""
        if context:
            return f"""You are a professional document writer. Generate high-quality content for the following request.

Context from company documents:
{context}

User Request:
{request}

Generate well-structured, professional content suitable for a business document.
"""
        else:
            return f"""You are a professional document writer. Generate high-quality content for the following request.

User Request:
{request}

Generate well-structured, professional content suitable for a business document.
"""
    
    def _format_kb_context(self, kb_results: List[Dict]) -> str:
        """Format KB results for context"""
        context_parts = []
        for result in kb_results:
            context_parts.append(f"From {result['source']}:\n{result['content']}\n")
        
        return "\n".join(context_parts)
    
    def _select_template(self, state: Dict) -> str:
        """Select appropriate template"""
        # Logic to select template based on document type
        # For now, return default
        return "default_template"
    
    def _extract_title(self, request: str) -> str:
        """Extract title from request"""
        # Simplified - could use LLM to extract
        return request[:100]
    
    def get_capabilities(self) -> List[str]:
        return [
            'generate_word_document',
            'generate_pdf_document',
            'apply_templates',
            'format_content',
            'integrate_kb_context'
        ]
```

#### Knowledge Base (RAG) Agent

```python
# apps/agents/agents/kb_agent.py
from .base_agent import BaseAgent
from apps.knowledge_base.services.ingestion_service import IngestionService
from apps.knowledge_base.services.retrieval_service import RetrievalService
from services.llm.openai_service import OpenAIService

class KBAgent(BaseAgent):
    """Agent responsible for knowledge base operations"""
    
    def __init__(self):
        super().__init__(
            name="KBAgent",
            description="Manages document ingestion and contextual retrieval"
        )
        self.ingestion_service = IngestionService()
        self.retrieval_service = RetrievalService()
        self.llm = OpenAIService()
    
    def can_handle(self, request: str) -> bool:
        """Check if request is for KB operations"""
        keywords = ['search', 'find', 'query', 'what does', 'tell me about']
        return any(keyword in request.lower() for keyword in keywords)
    
    async def execute(self, state: Dict[str, Any]) -> Dict[str, Any]:
        """Execute KB query"""
        start_time = datetime.utcnow()
        
        query = state.get('user_request', '')
        tenant_id = state.get('tenant_id')
        
        # Step 1: Retrieve relevant chunks
        retrieval_results = await self.retrieval_service.retrieve(
            query=query,
            tenant_id=tenant_id,
            top_k=5
        )
        
        # Step 2: Re-rank results
        ranked_results = self._rerank_results(query, retrieval_results)
        
        # Step 3: Generate response with LLM
        response = await self._generate_response(query, ranked_results)
        
        # Log execution
        duration = (datetime.utcnow() - start_time).total_seconds()
        self.log_execution(state, response, duration)
        
        return {
            'answer': response['text'],
            'sources': [r['document_id'] for r in ranked_results],
            'retrieved_chunks': ranked_results,
            'tokens_used': response.get('tokens_used', 0)
        }
    
    def _rerank_results(self, query: str, results: List[Dict]) -> List[Dict]:
        """Re-rank results by relevance"""
        # Implement cross-encoder re-ranking or other re-ranking logic
        # For now, return as-is
        return results
    
    async def _generate_response(self, query: str, chunks: List[Dict]) -> Dict:
        """Generate response from retrieved chunks"""
        context = "\n\n".join([
            f"[Source {i+1}]: {chunk['content']}"
            for i, chunk in enumerate(chunks)
        ])
        
        prompt = f"""Answer the following question based on the provided context.
If the answer is not in the context, say so.

Context:
{context}

Question: {query}

Answer:"""
        
        response = await self.llm.generate(
            prompt=prompt,
            max_tokens=500,
            temperature=0.3
        )
        
        return response
    
    async def ingest_document(self, file_path: str, tenant_id: str, user_id: str) -> Dict:
        """Ingest document into knowledge base"""
        result = await self.ingestion_service.ingest(
            file_path=file_path,
            tenant_id=tenant_id,
            user_id=user_id
        )
        
        return result
    
    def get_capabilities(self) -> List[str]:
        return [
            'ingest_documents',
            'semantic_search',
            'multi_document_reasoning',
            'contextual_retrieval',
            'answer_generation'
        ]
```

#### Image Generation Agent

```python
# apps/agents/agents/image_agent.py
from .base_agent import BaseAgent
from apps.images.services.image_generator import ImageGeneratorService

class ImageAgent(BaseAgent):
    """Agent responsible for image generation"""
    
    def __init__(self):
        super().__init__(
            name="ImageAgent",
            description="Generates images, logos, and diagrams"
        )
        self.image_service = ImageGeneratorService()
    
    def can_handle(self, request: str) -> bool:
        """Check if request is for image generation"""
        keywords = ['image', 'logo', 'diagram', 'visual', 'illustration']
        return any(keyword in request.lower() for keyword in keywords)
    
    async def execute(self, state: Dict[str, Any]) -> Dict[str, Any]:
        """Execute image generation"""
        start_time = datetime.utcnow()
        
        description = state.get('user_request', '')
        tenant_id = state.get('tenant_id')
        user_id = state.get('user_id')
        
        # Extract image parameters
        style = self._extract_style(description)
        size = state.get('image_size', '1024x1024')
        
        # Generate image
        image = await self.image_service.generate(
            prompt=description,
            style=style,
            size=size,
            tenant_id=tenant_id,
            user_id=user_id
        )
        
        # Log execution
        duration = (datetime.utcnow() - start_time).total_seconds()
        self.log_execution(state, {'image_id': str(image.id)}, duration)
        
        return {
            'image_id': str(image.id),
            'image_url': image.file_url,
            'status': 'completed'
        }
    
    def _extract_style(self, description: str) -> str:
        """Extract style from description"""
        style_keywords = {
            'corporate': ['corporate', 'professional', 'business'],
            'minimal': ['minimal', 'simple', 'clean'],
            'flat': ['flat', 'modern'],
            '3d': ['3d', 'realistic']
        }
        
        for style, keywords in style_keywords.items():
            if any(kw in description.lower() for kw in keywords):
                return style
        
        return 'corporate'  # default
    
    def get_capabilities(self) -> List[str]:
        return [
            'generate_images',
            'generate_logos',
            'generate_diagrams',
            'style_customization',
            'brand_compliance'
        ]
```

---

## 2. AGENT COMMUNICATION & ORCHESTRATION

### 2.1 LangGraph Orchestration Flow

```python
# apps/agents/orchestrator.py (Extended)
from langgraph.graph import StateGraph, END
from typing import TypedDict, List, Annotated, Sequence
import operator
from datetime import datetime

class AgentState(TypedDict):
    """Shared state across all agents"""
    # Input
    user_request: str
    tenant_id: str
    user_id: str
    
    # Routing
    intent: str
    required_agents: List[str]
    
    # Execution tracking
    current_agent: str
    completed_agents: List[str]
    
    # Results
    document_id: str
    presentation_id: str
    image_ids: List[str]
    kb_results: List[dict]
    
    # Agent outputs
    messages: Annotated[Sequence[dict], operator.add]
    
    # Error handling
    errors: List[str]
    retry_count: int
    
    # Metadata
    start_time: datetime
    total_tokens_used: int

class FlowSparkOrchestrator:
    """Master orchestrator for all agents"""
    
    def __init__(self):
        # Initialize all agents
        from .agents.document_agent import DocumentAgent
        from .agents.presentation_agent import PresentationAgent
        from .agents.kb_agent import KBAgent
        from .agents.image_agent import ImageAgent
        
        self.agents = {
            'document': DocumentAgent(),
            'presentation': PresentationAgent(),
            'kb': KBAgent(),
            'image': ImageAgent()
        }
        
        self.graph = self._build_workflow()
    
    def _build_workflow(self) -> StateGraph:
        """Build the agent workflow graph"""
        workflow = StateGraph(AgentState)
        
        # Add nodes
        workflow.add_node("parse_intent", self._parse_intent)
        workflow.add_node("document_agent", self._execute_document_agent)
        workflow.add_node("presentation_agent", self._execute_presentation_agent)
        workflow.add_node("kb_agent", self._execute_kb_agent)
        workflow.add_node("image_agent", self._execute_image_agent)
        workflow.add_node("aggregate_results", self._aggregate_results)
        workflow.add_node("validate_output", self._validate_output)
        
        # Set entry point
        workflow.set_entry_point("parse_intent")
        
        # Define conditional routing from parse_intent
        workflow.add_conditional_edges(
            "parse_intent",
            self._route_to_agents,
            {
                "document": "document_agent",
                "presentation": "presentation_agent",
                "kb_query": "kb_agent",
                "image": "image_agent",
                "multi_agent": "document_agent"  # Start multi-agent flow
            }
        )
        
        # Single agent flows go directly to validation
        workflow.add_edge("document_agent", "validate_output")
        workflow.add_edge("presentation_agent", "validate_output")
        workflow.add_edge("kb_agent", "validate_output")
        workflow.add_edge("image_agent", "validate_output")
        
        # Multi-agent flows
        # (Would add more complex routing here for multi-step workflows)
        
        # Validation always ends the workflow
        workflow.add_edge("validate_output", END)
        
        return workflow.compile()
    
    async def _parse_intent(self, state: AgentState) -> AgentState:
        """Parse user intent and determine required agents"""
        from services.llm.openai_service import OpenAIService
        
        llm = OpenAIService()
        
        prompt = f"""Analyze the following user request and determine the intent.

User Request: {state['user_request']}

Classify the intent as one of:
- document_generation: User wants to create a document
- presentation_generation: User wants to create a presentation
- kb_query: User wants to search/query knowledge base
- image_generation: User wants to generate an image
- multi_step: Requires multiple agents (describe which ones)

Return only the intent classification."""
        
        response = await llm.generate(prompt, max_tokens=50, temperature=0.1)
        intent = response['text'].strip().lower()
        
        state['intent'] = intent
        state['start_time'] = datetime.utcnow()
        
        return state
    
    def _route_to_agents(self, state: AgentState) -> str:
        """Route to appropriate agent(s) based on intent"""
        intent = state['intent']
        
        if 'document' in intent:
            return "document"
        elif 'presentation' in intent:
            return "presentation"
        elif 'query' in intent or 'search' in intent:
            return "kb_query"
        elif 'image' in intent:
            return "image"
        elif 'multi' in intent:
            return "multi_agent"
        
        return "document"  # default
    
    async def _execute_document_agent(self, state: AgentState) -> AgentState:
        """Execute document agent"""
        agent = self.agents['document']
        result = await agent.execute(state)
        
        state['document_id'] = result.get('document_id', '')
        state['completed_agents'].append('document')
        state['total_tokens_used'] += result.get('tokens_used', 0)
        state['messages'].append({
            'agent': 'document',
            'result': result
        })
        
        return state
    
    async def _execute_presentation_agent(self, state: AgentState) -> AgentState:
        """Execute presentation agent"""
        agent = self.agents['presentation']
        result = await agent.execute(state)
        
        state['presentation_id'] = result.get('presentation_id', '')
        state['completed_agents'].append('presentation')
        state['total_tokens_used'] += result.get('tokens_used', 0)
        state['messages'].append({
            'agent': 'presentation',
            'result': result
        })
        
        return state
    
    async def _execute_kb_agent(self, state: AgentState) -> AgentState:
        """Execute KB agent"""
        agent = self.agents['kb']
        result = await agent.execute(state)
        
        state['kb_results'] = result.get('retrieved_chunks', [])
        state['completed_agents'].append('kb')
        state['total_tokens_used'] += result.get('tokens_used', 0)
        state['messages'].append({
            'agent': 'kb',
            'result': result
        })
        
        return state
    
    async def _execute_image_agent(self, state: AgentState) -> AgentState:
        """Execute image agent"""
        agent = self.agents['image']
        result = await agent.execute(state)
        
        state['image_ids'].append(result.get('image_id', ''))
        state['completed_agents'].append('image')
        state['messages'].append({
            'agent': 'image',
            'result': result
        })
        
        return state
    
    async def _validate_output(self, state: AgentState) -> AgentState:
        """Validate all outputs"""
        # Implement validation logic
        # Check if outputs meet quality standards
        
        duration = (datetime.utcnow() - state['start_time']).total_seconds()
        
        state['messages'].append({
            'agent': 'orchestrator',
            'validation': 'passed',
            'total_duration': duration,
            'total_tokens': state['total_tokens_used']
        })
        
        return state
    
    async def run(self, user_request: str, tenant_id: str, user_id: str) -> Dict:
        """Execute the workflow"""
        initial_state = AgentState(
            user_request=user_request,
            tenant_id=tenant_id,
            user_id=user_id,
            intent='',
            required_agents=[],
            current_agent='',
            completed_agents=[],
            document_id='',
            presentation_id='',
            image_ids=[],
            kb_results=[],
            messages=[],
            errors=[],
            retry_count=0,
            start_time=datetime.utcnow(),
            total_tokens_used=0
        )
        
        final_state = await self.graph.ainvoke(initial_state)
        
        return final_state
```

---

## 3. MEMORY MANAGEMENT SYSTEM

### 3.1 Memory Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  MEMORY HIERARCHY                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌───────────────────────────────────────────────┐    │
│  │      SHORT-TERM MEMORY (Redis)                 │    │
│  │  - Current conversation context                │    │
│  │  - Active agent states                         │    │
│  │  - Session data                                │    │
│  │  TTL: 1 hour                                   │    │
│  └───────────────────────────────────────────────┘    │
│                        │                                │
│  ┌───────────────────────────────────────────────┐    │
│  │      WORKING MEMORY (Database)                 │    │
│  │  - Recent interactions (7 days)                │    │
│  │  - User preferences                            │    │
│  │  - Ongoing tasks                               │    │
│  └───────────────────────────────────────────────┘    │
│                        │                                │
│  ┌───────────────────────────────────────────────┐    │
│  │      LONG-TERM MEMORY (Vector DB)              │    │
│  │  - All historical documents                    │    │
│  │  - User patterns and behaviors                 │    │
│  │  - Learned preferences                         │    │
│  │  Persistent storage                            │    │
│  └───────────────────────────────────────────────┘    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 3.2 Memory Implementation

```python
# apps/agents/memory/memory_manager.py
from typing import Dict, Any, List, Optional
import redis
from django.core.cache import cache
from apps.knowledge_base.services.vector_db import TenantVectorDB
import json

class MemoryManager:
    """Manages agent memory across short-term, working, and long-term"""
    
    def __init__(self, tenant_id: str, user_id: str):
        self.tenant_id = tenant_id
        self.user_id = user_id
        self.redis_client = redis.from_url('redis://localhost:6379/0')
        self.vector_db = TenantVectorDB()
    
    # SHORT-TERM MEMORY (Redis)
    
    def store_conversation_context(self, conversation_id: str, messages: List[Dict]):
        """Store conversation in short-term memory"""
        key = f"conversation:{self.tenant_id}:{self.user_id}:{conversation_id}"
        self.redis_client.setex(
            key,
            3600,  # 1 hour TTL
            json.dumps(messages)
        )
    
    def get_conversation_context(self, conversation_id: str) -> List[Dict]:
        """Retrieve conversation context"""
        key = f"conversation:{self.tenant_id}:{self.user_id}:{conversation_id}"
        data = self.redis_client.get(key)
        
        if data:
            return json.loads(data)
        return []
    
    def store_agent_state(self, agent_name: str, state: Dict):
        """Store current agent state"""
        key = f"agent_state:{self.tenant_id}:{agent_name}"
        self.redis_client.setex(
            key,
            3600,
            json.dumps(state)
        )
    
    def get_agent_state(self, agent_name: str) -> Optional[Dict]:
        """Retrieve agent state"""
        key = f"agent_state:{self.tenant_id}:{agent_name}"
        data = self.redis_client.get(key)
        
        if data:
            return json.loads(data)
        return None
    
    # WORKING MEMORY (Database)
    
    def store_interaction(self, interaction_type: str, data: Dict):
        """Store interaction in working memory"""
        from apps.agents.models import AgentInteraction
        
        AgentInteraction.objects.create(
            tenant_id=self.tenant_id,
            user_id=self.user_id,
            interaction_type=interaction_type,
            data=data
        )
    
    def get_recent_interactions(self, limit: int = 10) -> List[Dict]:
        """Get recent interactions"""
        from apps.agents.models import AgentInteraction
        from datetime import timedelta, datetime
        
        seven_days_ago = datetime.utcnow() - timedelta(days=7)
        
        interactions = AgentInteraction.objects.filter(
            tenant_id=self.tenant_id,
            user_id=self.user_id,
            created_at__gte=seven_days_ago
        ).order_by('-created_at')[:limit]
        
        return [i.data for i in interactions]
    
    def store_user_preference(self, key: str, value: Any):
        """Store user preference"""
        from apps.agents.models import UserPreference
        
        UserPreference.objects.update_or_create(
            tenant_id=self.tenant_id,
            user_id=self.user_id,
            key=key,
            defaults={'value': value}
        )
    
    def get_user_preference(self, key: str) -> Optional[Any]:
        """Get user preference"""
        from apps.agents.models import UserPreference
        
        try:
            pref = UserPreference.objects.get(
                tenant_id=self.tenant_id,
                user_id=self.user_id,
                key=key
            )
            return pref.value
        except UserPreference.DoesNotExist:
            return None
    
    # LONG-TERM MEMORY (Vector DB)
    
    async def store_in_long_term(self, content: str, metadata: Dict):
        """Store in long-term memory (vector DB)"""
        await self.vector_db.add_memory(
            tenant_id=self.tenant_id,
            user_id=self.user_id,
            content=content,
            metadata=metadata
        )
    
    async def retrieve_relevant_memories(self, query: str, limit: int = 5) -> List[Dict]:
        """Retrieve relevant memories from long-term storage"""
        results = await self.vector_db.search_memories(
            tenant_id=self.tenant_id,
            user_id=self.user_id,
            query=query,
            limit=limit
        )
        
        return results
    
    # MEMORY CONSOLIDATION
    
    async def consolidate_memories(self):
        """Consolidate working memory into long-term memory"""
        # Get interactions from last 24 hours
        from datetime import datetime, timedelta
        
        recent_interactions = self.get_recent_interactions(limit=50)
        
        # Summarize and store important patterns
        # (This could use LLM to extract key insights)
        
        for interaction in recent_interactions:
            if self._is_important(interaction):
                await self.store_in_long_term(
                    content=json.dumps(interaction),
                    metadata={
                        'type': 'interaction_summary',
                        'timestamp': interaction.get('timestamp')
                    }
                )
    
    def _is_important(self, interaction: Dict) -> bool:
        """Determine if interaction is important enough for long-term memory"""
        # Simple heuristic - could be more sophisticated
        return interaction.get('successful', False)
```

---

## 4. LLM INTEGRATION STRATEGY

### 4.1 Multi-Model Strategy

```python
# services/llm/llm_router.py
from typing import Dict, Any
from .openai_service import OpenAIService
from .ollama_service import OllamaService
from .claude_service import ClaudeService

class LLMRouter:
    """Route requests to appropriate LLM based on task complexity and cost"""
    
    def __init__(self):
        self.openai = OpenAIService()
        self.ollama = OllamaService()
        self.claude = ClaudeService()
    
    async def generate(self, prompt: str, task_type: str, **kwargs) -> Dict:
        """Route to appropriate LLM"""
        
        # Simple tasks -> Local LLM (cost-effective)
        if task_type in ['simple_query', 'classification', 'extraction']:
            return await self.ollama.generate(prompt, **kwargs)
        
        # Complex reasoning -> GPT-4 or Claude
        elif task_type in ['complex_reasoning', 'creative_writing', 'code_generation']:
            return await self.openai.generate(prompt, model='gpt-4', **kwargs)
        
        # Document generation -> GPT-3.5 (balance of cost/quality)
        elif task_type in ['document_generation', 'summarization']:
            return await self.openai.generate(prompt, model='gpt-3.5-turbo', **kwargs)
        
        # Default to GPT-3.5
        else:
            return await self.openai.generate(prompt, **kwargs)
```

---

## 5. RAG PIPELINE ARCHITECTURE

### 5.1 Advanced RAG Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    RAG PIPELINE                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. INGESTION                                               │
│     ├── Document Upload                                     │
│     ├── Parse & Extract Text                                │
│     ├── Smart Chunking (semantic boundaries)                │
│     ├── Generate Embeddings                                 │
│     └── Store in Vector DB                                  │
│                                                             │
│  2. RETRIEVAL                                               │
│     ├── Query Embedding                                     │
│     ├── Vector Similarity Search                            │
│     ├── Hybrid Search (vector + keyword)                    │
│     ├── Re-ranking with Cross-Encoder                       │
│     └── Top-K Selection                                     │
│                                                             │
│  3. AUGMENTATION                                            │
│     ├── Context Assembly                                    │
│     ├── Prompt Construction                                 │
│     └── Metadata Injection                                  │
│                                                             │
│  4. GENERATION                                              │
│     ├── LLM Generation with Context                         │
│     ├── Citation Tracking                                   │
│     └── Response Formatting                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

*(Detailed RAG implementation provided in Knowledge Base app)*

---

## 6. AGENT ERROR HANDLING & RECOVERY

```python
# apps/agents/error_handling.py
from typing import Dict, Any
import logging

logger = logging.getLogger(__name__)

class AgentErrorHandler:
    """Handle agent errors and implement recovery strategies"""
    
    @staticmethod
    async def handle_error(agent_name: str, error: Exception, state: Dict[str, Any]) -> Dict:
        """Handle agent error with appropriate recovery strategy"""
        
        logger.error(f"Error in {agent_name}: {str(error)}", exc_info=True)
        
        # Determine error type and recovery strategy
        if isinstance(error, TimeoutError):
            return await AgentErrorHandler._handle_timeout(agent_name, state)
        
        elif isinstance(error, ConnectionError):
            return await AgentErrorHandler._handle_connection_error(agent_name, state)
        
        elif isinstance(error, ValueError):
            return await AgentErrorHandler._handle_validation_error(agent_name, state, error)
        
        else:
            return await AgentErrorHandler._handle_unknown_error(agent_name, state, error)
    
    @staticmethod
    async def _handle_timeout(agent_name: str, state: Dict) -> Dict:
        """Handle timeout errors"""
        retry_count = state.get('retry_count', 0)
        
        if retry_count < 3:
            logger.info(f"Retrying {agent_name} (attempt {retry_count + 1})")
            state['retry_count'] = retry_count + 1
            return {'status': 'retry', 'state': state}
        
        return {'status': 'failed', 'error': 'Timeout after 3 retries'}
    
    @staticmethod
    async def _handle_connection_error(agent_name: str, state: Dict) -> Dict:
        """Handle connection errors"""
        # Fallback to alternative service
        return {'status': 'fallback', 'message': 'Using fallback service'}
    
    @staticmethod
    async def _handle_validation_error(agent_name: str, state: Dict, error: Exception) -> Dict:
        """Handle validation errors"""
        return {
            'status': 'failed',
            'error': 'Validation failed',
            'details': str(error)
        }
    
    @staticmethod
    async def _handle_unknown_error(agent_name: str, state: Dict, error: Exception) -> Dict:
        """Handle unknown errors"""
        return {
            'status': 'failed',
            'error': 'Unknown error occurred',
            'details': str(error)
        }
```

---

**Document Version**: 1.0  
**Created**: December 28, 2025  
**Status**: Draft for Review

