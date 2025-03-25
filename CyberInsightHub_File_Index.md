# CyberInsightHub File Index

This document provides a comprehensive index of all files in the CyberReportV2 project, detailing what each file contains and how it relates to the overall system architecture. This reference should be used during coding to understand the existing documentation and ensure consistent implementation.

## Core Documentation Files

### 1. CyberReport2_PRD.md
**Purpose**: Product Requirements Document for the CyberInsightHub application
**Contents**:
- Application overview and purpose
- Technical architecture requirements
- Core features (User Management, Report Management, Dashboard, Chat Interface, Report Builder, Admin Interface)
- Supabase implementation details (tables, storage buckets, vector store, authentication)
- UI/UX guidelines
- AI and data processing specifications
- Implementation approach (phased development plan)

### 2. Frontend Architecture and UI Components.md
**Purpose**: Defines the frontend architecture and UI component structure
**Contents**:
- Technology stack (Next.js, TypeScript, Tailwind CSS, Shadcn UI, React Query, Zustand, Recharts)
- Architecture diagram showing component relationships
- Core architecture principles
- Application structure with file organization
- Key pages and routes
- Detailed component implementations including:
  - Layout components (MainLayout, Sidebar)
  - Dashboard components (DashboardOverview, StatCard)
  - Report components (ReportList, ReportCard, ReportViewer)
  - Chat interface components
  - Data visualization components
  - Admin components
- Responsive design approach
- Theme support implementation
- Accessibility considerations
- Performance optimization strategies
- Testing approach for frontend components

### 3. API Endpoint and Integration Design.md
**Purpose**: Defines the API architecture and endpoints
**Contents**:
- API architecture diagram
- Core API layer design with Next.js API routes
- Middleware setup for authentication and authorization
- API response standards
- Authentication and authorization endpoints
- Reports API endpoints
- Chat API endpoints
- Dashboard API endpoints
- Analysis API endpoints
- Admin API endpoints
- Webhook handling
- API integration utilities
- Error handling implementation
- API logging strategy
- Rate limiting implementation
- API versioning strategy
- Documentation and testing approach
- OpenAPI specification

### 4. Data Enrichment and Knowledge Graph Architecture.md
**Purpose**: Details the data enrichment services and Neo4j knowledge graph implementation
**Contents**:
- System architecture for data enrichment
- Entity extraction and recognition system
- External enrichment services integration (Diffbot, VirusTotal, MITRE CVE, AlienVault, News API)
- Neo4j knowledge graph implementation details
- Enrichment jobs and scheduling
- Admin interface for enrichment configuration
- Integration with LLM and chat system
- Performance and scaling considerations
- Security considerations for API keys and data validation

### 5. Database Schema and Vector Store Design.md
**Purpose**: Defines the database schema and vector embedding storage design
**Contents**:
- Database schema design for Supabase PostgreSQL
- Table definitions with relationships:
  - users table
  - reports table
  - report_sections table
  - report_entities table
  - report_stats table
  - user_saved_items table
  - custom_reports table
- Vector store implementation with pgvector
- Indexing strategy for efficient vector search
- Query optimization techniques
- Row-level security policies
- Database migration strategy
- Backup and recovery approach
- Performance considerations for scaling

### 6. Document Processing Pipeline Architecture.md
**Purpose**: Details the document processing workflow
**Contents**:
- Document processing pipeline architecture
- PDF/DOCX text extraction implementation
- Document chunking and section identification
- NER (Named Entity Recognition) for cybersecurity entities
- Metadata extraction and storage
- Processing job queue implementation
- Batch processing for large reports
- Error handling and retry logic
- Performance optimization for processing
- Monitoring and observability

### 7. Deployment Architecture and Infrastructure.md
**Purpose**: Defines the deployment architecture and infrastructure requirements
**Contents**:
- Architecture diagram showing system components
- Infrastructure components for frontend and backend
- Cloud provider considerations (AWS, GCP, Azure options)
- Containerization strategy using Docker
- Kubernetes deployment configuration
- Scaling strategy (horizontal and vertical)
- High availability and disaster recovery plans
- Performance optimization techniques
- Monitoring and observability setup
- Security measures
- Deployment workflow and CI/CD pipeline
- Cost optimization recommendations
- Maintenance procedures
- Compliance and governance considerations

### 8. Implementation and Roadmap Project Plan.md
**Purpose**: Provides the project timeline and phased implementation approach
**Contents**:
- Overall project timeline
- Phase 1: Foundation implementation details
- Phase 2: Core functionality implementation details
- Phase 3: Advanced features implementation details
- Phase 4: Polish and optimization details
- Milestone definitions
- Dependency management
- Risk assessment and mitigation strategies
- Resource allocation recommendations
- Success criteria and KPIs

### 9. LLM Integration and Chat System Design.md
**Purpose**: Details the LLM integration and chat interface implementation
**Contents**:
- LLM integration architecture
- OpenAI API integration details
- Context management for queries
- Prompt engineering templates
- Vector search for relevant context
- Citation generation and handling
- Conversation management
- Token usage optimization
- Fallback strategies
- Caching implementation
- Streaming response handling
- Performance considerations
- Security and privacy measures

### 10. System Security Plan.md
**Purpose**: Defines the security controls and compliance measures
**Contents**:
- Security architecture overview
- Authentication and authorization controls
- Data protection measures
- Network security configuration
- API security controls
- Vulnerability management approach
- Security monitoring and incident response
- Compliance requirements and controls
- Security testing methodology
- User security guidelines
- Admin security procedures
- Data retention and disposal policies

### 11. secrets.md
**Purpose**: Contains information about API keys and credential management
**Contents**:
- API key management approach
- Secret rotation policies
- Environment variable usage
- Secure storage recommendations
- Access control for credentials
- Monitoring for credential usage
- Development vs. production credentials handling

## Sample Data

### Sample Annual Reports Folder
**Purpose**: Provides sample cybersecurity reports for testing and development
**Contents**:
- Markdown subfolder: 12 cybersecurity reports in markdown format
- Normal subfolder: 12 corresponding PDF versions of the same reports
- Reports from various vendors including Arctic Wolf Labs, Check Point, Cisco, Crowdstrike, etc.
- Various report types (threat assessments, predictions, state of cybersecurity, etc.)

## Reusable Component Strategy

To ensure efficient development, the following components should be built as reusable modules rather than creating multiple similar implementations:

### UI Components

1. **Layout Components**
   - MainLayout - Base layout with navigation, header, and footer
   - Sidebar - Navigation sidebar with role-based menu items
   - Header - Application header with user controls
   - Footer - Standard footer component

2. **Data Display Components**
   - DataTable - Reusable table with sorting, filtering, and pagination
   - StatCard - Metric display with optional icon and trend indicator
   - InfoCard - Card component for displaying information blocks
   - Badge - Reusable badge component for status and categories

3. **Chart Components**
   - LineChart - Time-series visualization component
   - BarChart - Comparison visualization component
   - PieChart - Distribution visualization component
   - GeoMap - Geographic data visualization component
   - HeatMap - Intensity visualization component

4. **Form Components**
   - InputField - Standard text input with validation
   - SelectField - Dropdown selection component
   - FileUpload - File upload component with progress
   - SearchField - Search input with suggestions
   - DatePicker - Date selection component

5. **Interactive Components**
   - Modal - Reusable modal dialog component
   - Tooltip - Information tooltip component
   - Dropdown - Dropdown menu component
   - Tabs - Tab navigation component
   - Accordion - Expandable content component

6. **Specialty Components**
   - ChatMessage - Message display for chat interface
   - ReportCard - Report preview card
   - EntityTag - Entity display with type indication
   - Breadcrumbs - Navigation breadcrumbs component
   - AlertBox - Notification and alert component

### Backend Services

1. **Authentication Services**
   - userAuthentication - Handles user authentication flows
   - roleBasedAccess - Manages role-based permissions
   - sessionManagement - Handles user sessions

2. **Data Access Services**
   - databaseClient - Centralized Supabase database access
   - vectorStoreClient - Vector database query service
   - storageClient - File storage access service
   - graphDatabaseClient - Neo4j access service

3. **Processing Services**
   - documentProcessor - Text extraction from documents
   - embeddingGenerator - Vector embedding generation
   - entityExtractor - Named entity recognition service
   - contentChunker - Document chunking service

4. **Integration Services**
   - llmService - OpenAI API integration
   - enrichmentService - External data enrichment
   - notificationService - User notification handling
   - exportService - Report export functionality

5. **Utility Services**
   - errorHandler - Centralized error handling
   - logger - Logging service
   - apiClient - API request client
   - cacheManager - Data caching service