# CyberInsightHub Data Flow Architecture

This document outlines the complete data flow architecture for the CyberInsightHub system, showing how data moves between components, is transformed throughout the application lifecycle, and how different services interact with each other.

## System Overview Data Flow

The following diagram illustrates the high-level data flow through the entire CyberInsightHub system, from document ingestion to user interactions:

```mermaid
graph TD
    subgraph "Document Processing"
        A[Report Upload] --> B[Text Extraction]
        B --> C[Document Chunking]
        C --> D[Metadata Extraction]
        D --> E[Entity Extraction]
    end
    
    subgraph "Vector Processing"
        E --> F[Vector Embedding Generation]
        F --> G[Vector Storage]
    end
    
    subgraph "Knowledge Graph"
        E --> H[Entity Resolution]
        H --> I[Relationship Extraction]
        I --> J[Neo4j Graph Population]
        J --> K[Entity Enrichment]
    end
    
    subgraph "Search & Retrieval"
        G --> L[Vector Search]
        J --> M[Graph Search]
        D --> N[Metadata Search]
        L --> O[Hybrid Search Service]
        M --> O
        N --> O
    end
    
    subgraph "LLM Services"
        O --> P[Context Assembly]
        P --> Q[Prompt Construction]
        Q --> R[LLM Query]
        R --> S[Response Processing]
        S --> T[Citation Generation]
    end
    
    subgraph "User Interface"
        U[User Authentication] --> V[Dashboard]
        V --> W1[Report Management]
        V --> W2[Search Interface]
        V --> W3[Chat Interface]
        V --> W4[Report Builder]
        V --> W5[Data Visualization]
        V --> W6[Admin Panel]
        W2 --> O
        W3 --> P
        W4 --> X[Custom Report Generation]
        W5 --> Y[Chart Generation]
    end
    
    A --> AA[Raw Storage]
    D --> BB[PostgreSQL Database]
    G --> BB
    K --> BB
    BB --> V
    X --> BB
    Y --> BB
    J --> Y
```

## Detailed Component Data Flows

### 1. Document Processing Flow

```mermaid
sequenceDiagram
    participant User
    participant UI as Frontend UI
    participant API as API Gateway
    participant Queue as Processing Queue
    participant Extractor as Text Extractor
    participant Chunker as Document Chunker
    participant NER as Entity Extractor
    participant DB as PostgreSQL
    participant Storage as File Storage

    User->>UI: Upload report file
    UI->>API: POST /api/reports
    API->>Storage: Store raw file
    API->>Queue: Enqueue processing job
    API->>UI: Return job ID
    Queue->>Extractor: Process document
    Extractor->>Storage: Read raw file
    Extractor->>Chunker: Pass extracted text
    Chunker->>DB: Store document metadata
    Chunker->>NER: Pass document chunks
    NER->>DB: Store extracted entities
    Queue->>API: Processing complete
    API->>UI: Update status
    UI->>User: Show completion
```

**Data Transformations**:
1. **Raw File → Text**: PDF/DOCX processing to extract plain text
2. **Text → Chunks**: Text segmentation into semantic sections
3. **Chunks → Entities**: Named entity recognition produces structured entities
4. **Metadata Extraction**: Document attributes like title, date, author, source

**Storage Operations**:
- Raw files stored in Supabase Storage `raw_reports` bucket
- Extracted text stored in `reports` table
- Chunks stored in `report_sections` table
- Entities stored in `report_entities` table
- Metadata stored in `reports` table

### 2. Vector Embedding Flow

```mermaid
sequenceDiagram
    participant Queue as Processing Queue
    participant Chunker as Document Chunker
    participant Embedder as Vector Embedder
    participant OpenAI as OpenAI API
    participant DB as PostgreSQL with pgvector
    
    Queue->>Chunker: Document processed
    Chunker->>Embedder: Pass document chunks
    Embedder->>OpenAI: Request embeddings
    OpenAI->>Embedder: Return embeddings
    Embedder->>DB: Store vectors with chunks
```

**Data Transformations**:
1. **Text Chunks → Embedding Requests**: Prepare text for embedding API
2. **API Response → Vector Storage**: Format vectors for pgvector storage

**Storage Operations**:
- Vectors stored in `report_sections` table with pgvector extension
- Vector metadata (model, dimensions) stored with vectors

### 3. Knowledge Graph Construction Flow

```mermaid
sequenceDiagram
    participant NER as Entity Extractor
    participant Resolver as Entity Resolver
    participant Relationship as Relationship Extractor
    participant Neo4j as Neo4j Database
    participant Enricher as Entity Enricher
    participant Diffbot as Diffbot API
    
    NER->>Resolver: Pass extracted entities
    Resolver->>Resolver: Deduplicate entities
    Resolver->>Relationship: Pass resolved entities
    Relationship->>Neo4j: Create entity nodes
    Relationship->>Neo4j: Create relationships
    Neo4j->>Enricher: Entity needs enrichment
    Enricher->>Diffbot: Request entity data
    Diffbot->>Enricher: Return enriched data
    Enricher->>Neo4j: Update entity attributes
```

**Data Transformations**:
1. **Raw Entities → Resolved Entities**: Deduplication and normalization
2. **Entities → Neo4j Nodes**: Formatting for graph database
3. **Text Context → Relationships**: Extracting connections between entities
4. **Diffbot Data → Entity Attributes**: Mapping external data to our schema

**Storage Operations**:
- Entities stored as nodes in Neo4j
- Relationships stored as edges in Neo4j
- Entity attributes stored as node properties
- Confidence scores stored with relationships

### 4. Search and Retrieval Flow

```mermaid
sequenceDiagram
    participant User
    participant UI as Search Interface
    participant API as Search API
    participant Vector as Vector Search
    participant Meta as Metadata Search
    participant Graph as Graph Search
    participant Ranker as Result Ranker
    participant DB as PostgreSQL
    participant Neo4j as Neo4j Database
    
    User->>UI: Submit search query
    UI->>API: POST /api/search
    API->>Vector: Semantic search
    API->>Meta: Keyword/filter search
    API->>Graph: Knowledge graph search
    Vector->>DB: Query pgvector
    Meta->>DB: Query metadata
    Graph->>Neo4j: Cypher query
    DB->>Vector: Return vector matches
    DB->>Meta: Return metadata matches
    Neo4j->>Graph: Return graph matches
    Vector->>Ranker: Return results
    Meta->>Ranker: Return results
    Graph->>Ranker: Return results
    Ranker->>API: Merged, ranked results
    API->>UI: Return search results
    UI->>User: Display results
```

**Data Transformations**:
1. **User Query → Search Requests**: Query parsing for different search types
2. **Query Text → Vector**: Embedding generation for vector search
3. **Query → Filters**: Extracting metadata filters
4. **Multi-Source Results → Unified Results**: Results merging and ranking

**Storage Operations**:
- Vector similarity queries against pgvector indices
- Full-text search queries against PostgreSQL
- Property and pattern matching in Neo4j
- Search history stored in `user_searches` table

### 5. LLM Integration Flow

```mermaid
sequenceDiagram
    participant User
    participant UI as Chat Interface
    participant API as Chat API
    participant Context as Context Builder
    participant Search as Search Service
    participant Prompt as Prompt Constructor
    participant LLM as OpenAI API
    participant Citation as Citation Generator
    participant History as Conversation Store
    participant DB as PostgreSQL
    
    User->>UI: Submit chat message
    UI->>API: POST /api/chat
    API->>History: Get conversation history
    API->>Search: Get relevant context
    Search->>DB: Retrieve relevant passages
    Search->>API: Return context passages
    History->>API: Return conversation
    API->>Context: Build context window
    Context->>Prompt: Format prompt
    Prompt->>LLM: Send prompt to OpenAI
    LLM->>API: Return completion
    API->>Citation: Process for citations
    Citation->>API: Add citation metadata
    API->>History: Store interaction
    API->>UI: Stream response with citations
    UI->>User: Display response
```

**Data Transformations**:
1. **User Message → Search Queries**: Extracting search terms from message
2. **Search Results → Context**: Selecting and formatting relevant passages
3. **Context + History → Prompt**: Structured prompt construction
4. **LLM Response → Cited Response**: Adding citation metadata

**Storage Operations**:
- Conversation history stored in `chat_conversations` table
- Message contents stored in `chat_messages` table
- Citations stored with responses
- Token usage logged for monitoring

### 6. Dashboard Data Flow

```mermaid
sequenceDiagram
    participant User
    participant UI as Dashboard Interface
    participant API as Dashboard API
    participant Vis as Visualization Service
    participant Stats as Analytics Service
    participant DB as PostgreSQL
    participant Graph as Neo4j
    
    User->>UI: Load dashboard
    UI->>API: GET /api/dashboard
    API->>Stats: Request analytics
    Stats->>DB: Query report metrics
    Stats->>Graph: Query entity metrics
    DB->>Stats: Return report data
    Graph->>Stats: Return entity data
    Stats->>Vis: Generate visualizations
    Vis->>API: Return visualization data
    API->>UI: Return dashboard data
    UI->>User: Display dashboard
```

**Data Transformations**:
1. **Raw Metrics → Visualization Data**: Processing numbers into chart formats
2. **Entity Frequencies → Network Graphs**: Transforming entity data to visualizations
3. **Time Series Data → Trends**: Processing temporal data for trend visualization

**Storage Operations**:
- Pre-computed metrics stored in `report_stats` table
- User dashboard preferences stored in `user_preferences` table
- Generated visualizations cached in `generated_assets` storage

### 7. User Management Flow

```mermaid
sequenceDiagram
    participant User
    participant UI as Auth Interface
    participant Auth as Supabase Auth
    participant API as API Gateway
    participant DB as PostgreSQL
    
    User->>UI: Login attempt
    UI->>Auth: Submit credentials
    Auth->>Auth: Validate credentials
    Auth->>UI: Return JWT token
    UI->>API: Request with JWT
    API->>Auth: Validate token
    Auth->>API: Token validation result
    API->>DB: Get user permissions
    DB->>API: Return permissions
    API->>UI: Return authorized data
    UI->>User: Access granted
```

**Data Transformations**:
1. **Credentials → JWT**: Authentication token generation
2. **JWT → User Context**: Extracting user identity and permissions
3. **Permissions → UI State**: Adapting UI based on permissions

**Storage Operations**:
- User accounts stored in Supabase Auth tables
- User preferences stored in `user_preferences` table
- Role assignments stored in `user_roles` table
- Access logs stored for auditing

### 8. Report Builder Flow

```mermaid
sequenceDiagram
    participant User
    participant UI as Report Builder
    participant API as Builder API
    participant Search as Search Service
    participant Templates as Template Service
    participant Generator as Report Generator
    participant Storage as File Storage
    participant DB as PostgreSQL
    
    User->>UI: Create custom report
    UI->>API: Save report configuration
    User->>UI: Add content blocks
    UI->>Search: Find relevant content
    Search->>UI: Return search results
    User->>UI: Select content sources
    UI->>API: Update report structure
    User->>UI: Generate report
    UI->>Templates: Get report template
    Templates->>UI: Return template
    UI->>Generator: Generate report
    Generator->>Storage: Store generated report
    Generator->>DB: Update report metadata
    Generator->>UI: Return report URL
    UI->>User: Show completed report
```

**Data Transformations**:
1. **User Selections → Report Structure**: Building report configuration
2. **Content Sources → Report Content**: Assembling content from various sources
3. **Template + Content → Generated Report**: Rendering final report

**Storage Operations**:
- Report configurations stored in `custom_reports` table
- Generated reports stored in `generated_assets` storage bucket
- Templates stored in `report_templates` table

## API Contract Specifications

### Authentication API

- **POST /api/auth/login**
  - Request: `{ email, password }`
  - Response: `{ user, session, token }`

- **POST /api/auth/register**
  - Request: `{ email, password, name }`
  - Response: `{ user, session }`

- **POST /api/auth/logout**
  - Request: `{ token }`
  - Response: `{ success }`

- **GET /api/auth/user**
  - Response: `{ user, permissions }`

### Reports API

- **POST /api/reports**
  - Request: `{ file, metadata }`
  - Response: `{ report_id, job_id, status }`

- **GET /api/reports**
  - Parameters: `{ page, limit, filters }`
  - Response: `{ reports[], total, page_count }`

- **GET /api/reports/:id**
  - Response: `{ report, sections[], metadata }`

- **GET /api/reports/:id/entities**
  - Parameters: `{ entity_types[], confidence }`
  - Response: `{ entities[] }`

### Search API

- **POST /api/search**
  - Request: `{ query, filters, page, limit }`
  - Response: `{ results[], total, page_count }`

- **POST /api/search/semantic**
  - Request: `{ text, filters, limit }`
  - Response: `{ results[] }`

- **POST /api/search/hybrid**
  - Request: `{ query, filters, weights, limit }`
  - Response: `{ results[] }`

### Chat API

- **POST /api/chat**
  - Request: `{ message, conversation_id, context_ids[] }`
  - Response: Stream of `{ content, citations[], finished }`

- **GET /api/chat/conversations**
  - Response: `{ conversations[] }`

- **GET /api/chat/conversations/:id**
  - Response: `{ messages[], metadata }`

### Knowledge Graph API

- **GET /api/graph/entities**
  - Parameters: `{ types[], limit }`
  - Response: `{ entities[] }`

- **GET /api/graph/entities/:id**
  - Response: `{ entity, attributes, relationships[] }`

- **GET /api/graph/search**
  - Parameters: `{ query, types[], limit }`
  - Response: `{ entities[], relationships[] }`

### Dashboard API

- **GET /api/dashboard**
  - Response: `{ stats, charts[], recent_reports[] }`

- **GET /api/dashboard/chart/:type**
  - Parameters: `{ period, filters }`
  - Response: `{ data, metadata }`

### Admin API

- **GET /api/admin/users**
  - Parameters: `{ page, limit, filters }`
  - Response: `{ users[], total, page_count }`

- **PUT /api/admin/users/:id**
  - Request: `{ role, permissions, status }`
  - Response: `{ user }`

- **GET /api/admin/jobs**
  - Parameters: `{ status, type, page, limit }`
  - Response: `{ jobs[], total, page_count }`

## Data Storage Schema

### PostgreSQL Tables

```mermaid
erDiagram
    users {
        uuid id PK
        string email
        string name
        string role
        jsonb preferences
        timestamp created_at
        timestamp last_login
    }
    
    reports {
        uuid id PK
        uuid uploader_id FK
        string title
        string source
        date publication_date
        jsonb metadata
        text summary
        timestamp created_at
        boolean processing_complete
    }
    
    report_sections {
        uuid id PK
        uuid report_id FK
        string title
        text content
        int position
        string section_type
        vector embedding
        jsonb metadata
    }
    
    report_entities {
        uuid id PK
        uuid report_id FK
        uuid section_id FK
        string entity_name
        string entity_type
        float confidence
        jsonb attributes
        timestamp created_at
    }
    
    report_stats {
        uuid id PK
        uuid report_id FK
        int page_count
        int word_count
        int entity_count
        jsonb entity_type_counts
        timestamp created_at
    }
    
    user_saved_items {
        uuid id PK
        uuid user_id FK
        uuid item_id
        string item_type
        jsonb metadata
        timestamp created_at
    }
    
    custom_reports {
        uuid id PK
        uuid creator_id FK
        string title
        string description
        jsonb structure
        timestamp created_at
        timestamp updated_at
    }
    
    chat_conversations {
        uuid id PK
        uuid user_id FK
        string title
        timestamp created_at
        timestamp updated_at
    }
    
    chat_messages {
        uuid id PK
        uuid conversation_id FK
        string role
        text content
        jsonb citations
        int tokens
        timestamp created_at
    }
    
    user_searches {
        uuid id PK
        uuid user_id FK
        string query
        jsonb filters
        int result_count
        timestamp created_at
    }
    
    users ||--o{ reports : uploads
    users ||--o{ user_saved_items : saves
    users ||--o{ custom_reports : creates
    users ||--o{ chat_conversations : initiates
    reports ||--o{ report_sections : contains
    reports ||--o{ report_entities : has
    reports ||--o{ report_stats : generates
    report_sections ||--o{ report_entities : contains
    chat_conversations ||--o{ chat_messages : contains
```

### Neo4j Graph Schema

```mermaid
graph TD
    A[ThreatActor] -->|targets| B[Organization]
    A -->|uses| C[Malware]
    A -->|employs| D[AttackTechnique]
    A -->|operates| E[Infrastructure]
    C -->|exploits| F[Vulnerability]
    D -->|affects| G[Technology]
    H[CounterMeasure] -->|mitigates| F
    H -->|protects| B
    I[Industry] -->|includes| B
    J[Event] -->|involves| A
    J -->|affects| B
    K[Report] -->|mentions| A
    K -->|describes| J
    K -->|analyzes| C
```

## External Service Integrations

### OpenAI API Integration

**Services Used**:
- **text-embedding-3-large**: For generating document embeddings
- **gpt-4**: For chat completions and analysis

**Authentication**:
- API key stored in Supabase environment variables

**Data Flow**:
1. System makes API requests with document chunks or user queries
2. API returns embeddings or completions
3. System stores embeddings in pgvector or processes completions for user

### Diffbot API Integration

**Services Used**:
- **Knowledge Graph API**: For entity enrichment
- **Article API**: For additional context from web

**Authentication**:
- API key stored in Supabase environment variables

**Data Flow**:
1. Entity data sent to Diffbot for enrichment
2. Diffbot returns additional entity information
3. System integrates Diffbot data with existing entity records

### Additional External Services

- **VirusTotal API**: For malware and indicator enrichment
- **MITRE CVE Database**: For vulnerability information
- **AlienVault OTX**: For threat intelligence context
- **News APIs**: For current event correlation

## Cross-Cutting Concerns

### Error Handling

Error flows are implemented across all data paths:

1. **Input Validation Errors**: Returned as 400 Bad Request
2. **Authentication Errors**: Returned as 401 Unauthorized
3. **Permission Errors**: Returned as 403 Forbidden
4. **Resource Errors**: Returned as 404 Not Found
5. **Processing Errors**: Logged and returned as 500 Internal Server Error

### Logging and Monitoring

Data flows are logged at these key points:

1. **API Request/Response**: Timing, user ID, endpoint
2. **Processing Jobs**: Status changes, completion, errors
3. **LLM Interactions**: Prompts, responses, token usage
4. **User Actions**: Searches, report views, chat interactions

### Caching Strategy

Caching is implemented at these points:

1. **Frontend Caching**: Client-side caching of UI components
2. **API Response Caching**: Common queries and responses
3. **Search Results Caching**: Recent or common searches
4. **Embedding Caching**: To prevent duplicate API calls
5. **Visualization Caching**: Dashboard charts and graphs

## Data Flow for Key User Journeys

### Journey 1: Uploading and Processing a New Report

```mermaid
sequenceDiagram
    participant User as Security Analyst
    participant UI as Upload Interface
    participant API as Reports API
    participant Queue as Processing Queue
    participant DB as Database
    
    User->>UI: Upload cybersecurity report
    UI->>API: Submit report with metadata
    API->>DB: Store report record
    API->>Queue: Enqueue processing job
    API->>UI: Return confirmation
    Queue->>Queue: Process report (text extraction)
    Queue->>Queue: Generate sections
    Queue->>Queue: Extract entities
    Queue->>Queue: Generate embeddings
    Queue->>Queue: Build knowledge graph
    Queue->>DB: Update processing status
    Queue->>UI: Send notification
    UI->>User: Show completion notification
```

### Journey 2: Searching and Analyzing Reports

```mermaid
sequenceDiagram
    participant User as Security Analyst
    participant UI as Search Interface
    participant API as Search API
    participant DB as Database
    participant Vis as Visualization Service
    
    User->>UI: Enter search query
    UI->>API: Submit search request
    API->>DB: Perform hybrid search
    DB->>API: Return search results
    API->>UI: Display results
    User->>UI: Select report section
    UI->>API: Request detail view
    API->>DB: Get section and context
    DB->>API: Return detailed data
    API->>UI: Display section with context
    User->>UI: Request visualization
    UI->>Vis: Generate visualization
    Vis->>UI: Return visualization
    UI->>User: Display interactive chart
```

### Journey 3: Chatting with Report Knowledge

```mermaid
sequenceDiagram
    participant User as Security Analyst
    participant UI as Chat Interface
    participant API as Chat API
    participant Search as Search Service
    participant LLM as OpenAI
    participant DB as Database
    
    User->>UI: Ask question about trends
    UI->>API: Send message
    API->>Search: Find relevant context
    Search->>DB: Semantic search
    DB->>Search: Return contextual passages
    Search->>API: Provide context
    API->>LLM: Send prompt with context
    LLM->>API: Stream response
    API->>UI: Stream response with citations
    UI->>User: Display cited response
    User->>UI: Follow-up question
    UI->>API: Send with conversation history
    API->>LLM: Send updated prompt
    LLM->>API: Stream new response
    API->>UI: Stream with updated citations
    UI->>User: Display continued conversation
```

## Conclusion

This data flow architecture document provides a comprehensive view of how data moves through the CyberInsightHub system. It serves as a guide for implementation, ensuring that all components work together seamlessly and that data is transformed appropriately at each stage of processing.

Key points to remember during implementation:

1. All data flows should be secured with proper authentication and authorization
2. Error handling must be implemented at each transition point
3. Logging should capture key events in the data lifecycle
4. Performance monitoring should be implemented at critical flow points
5. Data quality should be validated at each transformation stage