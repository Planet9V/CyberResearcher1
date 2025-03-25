# Comprehensive Prompt for Cybersecurity Reports Aggregation Platform

I'll create an actionable prompt for developing a full web application that aggregates cybersecurity reports using Supabase as the backend.

## Application Overview and Purpose

```
Create a comprehensive web application called "CyberInsightHub" that aggregates, analyzes, and presents insights from hundreds of annual cybersecurity reports. The platform will serve as a single source of truth for cybersecurity professionals who lack time to read numerous reports individually. The application must feature intuitive navigation, powerful search capabilities, AI-driven analysis, and custom report generation - all backed by a Supabase infrastructure.
```

## Technical Architecture Requirements

```
Develop a React-based web application with the following architecture:

1. Frontend: 
   - Next.js framework with TypeScript
   - Responsive design using Tailwind CSS
   - Component library (e.g., Shadcn UI or Radix UI)

2. Backend/Database (Supabase):
   - Authentication system for user management
   - PostgreSQL database for metadata and user data
   - Supabase Storage for raw report files
   - Supabase Vector database for searchable content (pgvector)
   - Row-level security policies for proper access control

3. AI Integration:
   - OpenAI API integration for chat and analysis
   - Vector embeddings generation for semantic search
   - Chart and visualization generation based on report data
```

## Core Features

```
Implement the following key features:

1. User Management:
   - Registration/login system with email verification
   - Role-based access (admin, analyst, basic user)
   - User profile management and preferences

2. Report Management:
   - Upload interface for adding new cybersecurity reports
   - Automatic processing pipeline to extract text, generate embeddings, and store metadata
   - Report categorization by vendor, year, threat type, and industry
   - Citation system to track sources of information

3. Dashboard and Visualization:
   - Main dashboard showing aggregated trends across reports
   - Interactive charts for threat evolution over time
   - Comparative analysis between vendors and years
   - Geographical threat maps
   - Custom visualization generation

4. AI-Powered Chat Interface:
   - Context-aware chat that references specific reports
   - Citation of sources for all information provided
   - Ability to ask complex analytical questions across multiple reports
   - Option to generate visualizations from chat queries

5. Custom Report Builder:
   - Interface to select and combine insights from multiple reports
   - Drag-and-drop content organization
   - Automatic citation management
   - Export options (PDF, PPTX, HTML)

6. Admin Interface:
   - API configuration for LLM, search engines, and news sources
   - User management and access control
   - System monitoring and usage statistics
   - Data enrichment configuration
```

## Supabase Implementation Details

```
Configure Supabase with the following structure:

1. Tables:
   - users: Extended user profile information
   - reports: Metadata about each report (title, author, year, vendor, etc.)
   - report_sections: Chunked sections of reports with metadata
   - report_stats: Extracted statistics and metrics from reports
   - user_saved_items: User bookmarks and saved searches
   - custom_reports: User-generated reports

2. Storage Buckets:
   - raw_reports: Original PDF/DOCX files
   - processed_reports: Converted markdown versions
   - generated_assets: AI-generated charts and visualizations

3. Vector Store:
   - report_embeddings: Vector embeddings of report chunks for semantic search
   - Create appropriate indexes for efficient vector similarity search

4. Authentication:
   - Email/password authentication
   - Optional OAuth providers (Google, Microsoft)
   - Row-level security policies for all tables and buckets
```

## UI/UX Guidelines

```
Design the user interface with the following principles:

1. Information Architecture:
   - Minimal, clean design with focus on content
   - Progressive disclosure of complex features
   - Consistent navigation and breadcrumbs
   - Clear visual hierarchy and typography

2. Dashboard Design:
   - Card-based layout for main metrics
   - Interactive, filterable visualizations
   - Drill-down capability from overview to detailed data
   - Dark/light mode toggle

3. Chat Interface:
   - Persistent chat panel with clear context indicators
   - Visual citation links to source reports
   - Split-screen option to view chat and reports simultaneously
   - Code block and markdown support for complex responses

4. Report Builder:
   - Canvas-based drag-and-drop interface
   - Live preview of generated report
   - Template selection for different report types
   - Clear feedback on citation status
```

## AI and Data Processing

```
Implement the following AI components:

1. Document Processing Pipeline:
   - PDF/DOCX text extraction with layout preservation
   - Automatic section identification and chunking
   - Named entity recognition for companies, threats, CVEs
   - Statistic and metric extraction
   - Vector embedding generation

2. Chat Intelligence:
   - Integration with OpenAI API (GPT-4 or equivalent)
   - Context window management for referencing multiple reports
   - Citation generation for all statements
   - Prompt engineering for analytical queries
   - Follow-up question handling

3. Visualization Generation:
   - Data aggregation across report sources
   - Dynamic chart generation based on query intent
   - Consistent styling and branding
   - Exportable high-resolution assets
```

## Implementation Approach

```
Build the application using the following development approach:

1. Phase 1: Core Infrastructure
   - Set up Supabase project with authentication and database schema
   - Create basic UI with authentication flows
   - Implement report upload and processing pipeline

2. Phase 2: Search and Analysis
   - Develop vector search capabilities
   - Implement basic chat interface with report context
   - Create initial dashboard visualizations

3. Phase 3: Advanced Features
   - Build custom report generator
   - Enhance chat with multi-report analysis
   - Implement advanced visualizations
   - Add data enrichment from external sources

4. Phase 4: Polish and Optimization
   - Refine UI/UX based on testing
   - Optimize performance for large report collections
   - Implement advanced security measures
   - Add export and sharing capabilities
```

Would you like me to provide more specific technical details for any particular component of this application, such as the data schema, vector embedding approach, or the document processing pipeline?