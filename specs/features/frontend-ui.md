# Feature Requirements Document (FRD)

## Feature: Interactive Frontend UI

**Feature ID**: FRD-003  
**Phase**: Phase 1 - Foundation  
**Priority**: High (User Interface)  
**Status**: Draft  
**Related PRD Requirements**: [REQ-5], [REQ-9], [REQ-12]

---

## 1. Overview

### **Purpose**
Provide an intuitive web-based user interface that enables CLP team members to interact with the Property Registry, Schema Generation, and Mapping Assistant features without requiring command-line or API knowledge.

### **Business Value**
- Reduces learning curve for non-technical users (domain scientists)
- Centralizes all workflows in one accessible location
- Provides visual feedback for AI suggestions and confidence scores
- Enables collaboration through shared workspace
- Reduces training time from days to <30 minutes

### **User Personas**
- **Schema Owner**: Primary user for schema generation and property management
- **Domain Scientist**: Reviews and validates property mappings
- **Developer**: Uses for quick testing and manual operations
- **Product Owner**: Views audit logs and usage metrics

---

## 2. Functional Requirements

### **Core Pages & Features**

#### **2.1 Dashboard** (`/`)
- **Welcome Message**: Personalized greeting with user role
- **Quick Stats**:
  - Total properties in registry
  - Total schemas created
  - Recent activity (last 7 days)
- **Quick Actions**:
  - Button: "Create New Schema"
  - Button: "Browse Property Registry"
  - Button: "View Audit Logs"
- **Recent Schemas**: List of 5 most recently created/edited schemas
- **System Status**: Health indicators for backend services (Property Registry API, Azure OpenAI, Blob Storage)

#### **2.2 Property Registry Browser** (`/properties`)

**Property List View**
- **Search Bar**: Full-text search with autocomplete
- **Filters**:
  - Test Type (dropdown: All, Lap Shear, Tensile, Rheology, etc.)
  - Unit (dropdown: All, MPa, N, mm, Pa·s, etc.)
  - Data Type (dropdown: All, float, integer, string, boolean)
- **Table Columns**:
  - Canonical Name
  - Description (truncated with "..." and tooltip on hover)
  - Unit
  - Data Type
  - Test Types (pills/tags)
  - Actions: View | Edit | Delete
- **Pagination**: 50 properties per page
- **Sort**: Click column headers to sort (ascending/descending)

**Property Detail View** (`/properties/{id}`)
- Display all property fields:
  - Canonical Name
  - Description (full text)
  - Standard Unit
  - Data Type
  - Test Types (list)
  - Albert Template Reference
  - Required/Optional flag
  - Validation Rules (JSON viewer)
  - Created Date & By
  - Last Modified Date
  - Usage Count (# of schemas using this property)
- **Actions**:
  - Button: "Edit Property"
  - Button: "Delete Property" (disabled if usage_count > 0)
  - Button: "View Schemas Using This Property"

**Create/Edit Property Form** (`/properties/new`, `/properties/{id}/edit`)
- **Fields**:
  - Canonical Name (text input, required, validation: alphanumeric + underscore)
  - Description (textarea, required)
  - Standard Unit (dropdown from unit catalog, required)
  - Data Type (dropdown: float, integer, string, boolean, datetime)
  - Test Types (multi-select dropdown, optional)
  - Albert Template Reference (text input, optional)
  - Is Required? (checkbox)
  - Validation Rules (JSON editor with syntax highlighting)
- **Buttons**:
  - "Save Property"
  - "Cancel"
- **Validation Messages**: Display errors below each field (e.g., "Canonical name already exists")

#### **2.3 Schema Generation Workflow** (`/schemas/new`)

**Step 1: Upload Files**
- **Drag-and-Drop Zone**: "Drag files here or click to browse"
- **Supported Formats**: TXT, CSV, Excel, PDF (show icons)
- **File List**: Show uploaded files with:
  - Filename
  - File size
  - Status icon (uploading, uploaded, error)
  - Remove button (X)
- **Test Type Selector**: Dropdown (Lap Shear, Tensile, Rheology, Other)
- **Standard Selector**: Dropdown (ISO 4587, ASTM D905, Other, optional)
- **Button**: "Analyze Files" (enabled when ≥1 file uploaded)

**Step 2: Review AI Suggestions**
- **Table**: Show extracted properties with AI matches
- **Columns**:
  - Raw Header (from file)
  - Extracted Unit
  - AI Suggested Property (canonical name)
  - Confidence Score (progress bar: green >85%, yellow 70-85%, red <70%)
  - Justification (tooltip icon, click to expand)
  - Action (dropdown: Accept | Edit | Reject | Create New)
- **Filters**: Show only low confidence (<70%), Show only unmapped
- **Bulk Actions**:
  - "Accept All High Confidence (>85%)"
  - "Reject All Low Confidence (<70%)"
- **Edit Action**:
  - Opens dropdown with all registry properties
  - Type-ahead search
  - Click to select
- **Create New Action**:
  - Opens modal with "Create Property" form (same as Property Registry)
  - On save, returns to table and auto-maps header to new property

**Step 3: Generate & Preview Schema**
- **Button**: "Generate Schema" (enabled when all headers mapped or rejected)
- **Schema Preview**:
  - JSON view with syntax highlighting
  - Collapsible sections (properties, metadata)
  - Download button for raw JSON
- **Metadata Summary**:
  - Test Type
  - Standard
  - Number of Properties
  - Required Properties (list)
  - Optional Properties (list)
  - Source Files (list)
- **Buttons**:
  - "Back to Edit" (return to Step 2)
  - "Export Schema" (opens export modal)

**Export Modal**
- **Format Options**: JSON (default), YAML, Excel
- **Destination**: Download to local (Phase 1), Azure Blob Storage (Phase 2)
- **Versioning**: Auto-increment version or manual entry
- **Button**: "Export"

#### **2.4 Schema Library** (`/schemas`)
- **List View**: Table of all created schemas
- **Columns**:
  - Schema Name (test type + version)
  - Test Type
  - Standard
  - Version
  - # Properties
  - Created Date
  - Created By
  - Actions: View | Edit | Export | Delete
- **Filters**: Test Type, Standard, Created By, Date Range
- **Search**: By schema name or test type

**Schema Detail View** (`/schemas/{id}`)
- Display full schema JSON (formatted)
- Show metadata
- List all properties with links to Property Registry
- Show source files used
- **Actions**:
  - "Edit Schema" (reload Schema Generation workflow)
  - "Export Schema"
  - "Create Mapping" (navigate to Mapping Assistant)

#### **2.5 Audit Log Viewer** (`/audit`)
- **Table**: All logged actions
- **Columns**:
  - Timestamp
  - User
  - Action (e.g., "Created Property", "Generated Schema", "AI Matched Property")
  - Entity (property name, schema name, etc.)
  - Details (JSON expandable)
- **Filters**: Date range, User, Action type
- **Export**: Download audit log as CSV

### **Non-Functional Requirements**

#### **2.6 Responsive Design**
- Desktop-first (primary use case)
- Tablet support (iPad)
- Mobile: Read-only views (no editing)

#### **2.7 Accessibility**
- WCAG 2.1 AA compliance
- Keyboard navigation for all actions
- Screen reader support
- High-contrast mode

#### **2.8 Performance**
- Page load time <2 seconds
- File upload progress indicator
- Async operations with loading spinners
- No page refresh for AJAX operations

---

## 3. Technical Architecture

### **Technology Stack (Azure-First)**

#### **Frontend Framework: React (TypeScript)**
- **Why**: Modern, component-based, strong Azure integration ecosystem
- **UI Library**: Azure Fluent UI (Microsoft's design system)
- **State Management**: React Query for API calls, Zustand for local state
- **Routing**: React Router
- **Forms**: React Hook Form with Zod validation

#### **Hosting: Azure Static Web Apps**
- **Why**: Serverless, global CDN, built-in authentication, GitHub Actions CI/CD
- **Tier**: Free or Standard (depending on usage)
- **Custom Domain**: Support for custom DNS
- **HTTPS**: Auto-provisioned SSL certificates

**Alternative**: Azure App Service (if dynamic server-side rendering needed)

#### **API Communication**
- **REST API**: Calls to Azure Functions (Property Registry, Schema Generation)
- **HTTP Client**: Axios with retry logic
- **Authentication**: Azure AD B2C (Phase 2) or API Keys (Phase 1)

#### **File Upload**
- **Direct to Azure Blob Storage**: Use SAS tokens for secure upload
- **Progress Tracking**: Blob Storage SDK upload events
- **Validation**: Client-side (file type, size) + server-side

#### **Styling**
- **CSS Framework**: Tailwind CSS + Fluent UI components
- **Icons**: Fluent UI Icons
- **Color Scheme**: Henkel brand colors (provide customization)

#### **Build & Deploy**
- **Build Tool**: Vite (fast builds, HMR)
- **CI/CD**: GitHub Actions (deploy to Azure Static Web Apps on push to main)
- **Environments**: Dev, Staging, Production

### **Component Architecture**

```
src/
├── components/
│   ├── common/
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Table.tsx
│   │   ├── Modal.tsx
│   │   └── FileUpload.tsx
│   ├── properties/
│   │   ├── PropertyList.tsx
│   │   ├── PropertyDetail.tsx
│   │   ├── PropertyForm.tsx
│   │   └── PropertySearch.tsx
│   ├── schemas/
│   │   ├── SchemaWizard.tsx
│   │   ├── FileUploadStep.tsx
│   │   ├── ReviewSuggestionsStep.tsx
│   │   ├── SchemaPreviewStep.tsx
│   │   └── SchemaList.tsx
│   └── audit/
│       └── AuditLogTable.tsx
├── pages/
│   ├── Dashboard.tsx
│   ├── PropertiesPage.tsx
│   ├── SchemaGenerationPage.tsx
│   ├── SchemaLibraryPage.tsx
│   └── AuditLogPage.tsx
├── services/
│   ├── propertyService.ts
│   ├── schemaService.ts
│   ├── fileService.ts
│   └── auditService.ts
├── hooks/
│   ├── useProperties.ts
│   ├── useSchemas.ts
│   └── useFileUpload.ts
├── utils/
│   ├── api.ts
│   ├── validators.ts
│   └── formatters.ts
└── App.tsx
```

---

## 4. Integration Points

### **Inputs**

| Source | Method | Purpose |
|--------|--------|---------|
| User File Upload | Browser File API → Azure Blob | Upload test files for schema generation |
| User Form Input | React forms | Create/edit properties and schemas |

### **Outputs**

| Destination | Method | Purpose |
|-------------|--------|---------|
| Property Registry API | REST API (CRUD) | Manage properties |
| Schema Generation API | REST API | Generate schemas |
| Azure Blob Storage | Direct upload (SAS) | Store uploaded files |
| Local Download | Browser download | Export schemas and audit logs |

### **Dependencies**

- **Property Registry API**: GET /properties, POST /properties, etc.
- **Schema Generation API**: POST /generate-schema
- **Azure Blob Storage**: Container for file uploads
- **Azure OpenAI**: Called by backend (not directly from UI)

---

## 5. User Workflows

### **Workflow 1: Create New Property**

1. Click "Browse Property Registry" on dashboard
2. Click "Add New Property" button
3. Fill form: Name, Description, Unit, Type, Test Types
4. Click "Save Property"
5. See success message: "Property 'Viscosity_PaS' created"
6. Redirected to Property Detail view

**Acceptance Criteria**:
- Form validation prevents invalid inputs
- Duplicate names rejected with clear error
- New property appears in search results immediately

### **Workflow 2: Generate Schema with AI Assistance**

1. Click "Create New Schema" on dashboard
2. Select test type: "Lap Shear"
3. Drag 3 files into upload zone
4. Click "Analyze Files"
5. Wait 30 seconds (loading spinner)
6. Review table: 10 properties extracted, 8 green (high confidence), 2 yellow
7. Click "Accept All High Confidence"
8. Manually select registry property for 2 yellow rows
9. Click "Generate Schema"
10. Preview JSON schema
11. Click "Export" → Download JSON

**Acceptance Criteria**:
- File upload shows progress
- AI suggestions load within 60 seconds
- Confidence colors match thresholds
- JSON preview renders correctly
- Export downloads immediately

### **Workflow 3: Search and Reuse Property**

1. In Schema Generation Step 2, see unmapped header "Viskosität"
2. Click "Edit" dropdown for that row
3. Type "visc" in search box
4. Dropdown shows "Viscosity_PaS" as top result
5. Click to select
6. Row updates with selected property

**Acceptance Criteria**:
- Search is case-insensitive
- Results appear as user types
- Selection updates table immediately

---

## 6. Acceptance Criteria

### **Functional Acceptance**

- [ ] All pages render correctly on desktop (Chrome, Edge, Firefox)
- [ ] Property Registry: CRUD operations work without errors
- [ ] Schema Generation: End-to-end workflow completes successfully
- [ ] File upload supports TXT, CSV, Excel, PDF
- [ ] AI suggestions display with confidence scores and justifications
- [ ] Export generates valid JSON files
- [ ] Audit log displays all user actions
- [ ] Search and filters work across all tables

### **Non-Functional Acceptance**

- [ ] Page load time <2 seconds on good connection
- [ ] File upload shows progress indicator
- [ ] No full-page refreshes for AJAX operations
- [ ] Responsive design works on desktop and tablet
- [ ] Keyboard navigation works for all forms
- [ ] WCAG 2.1 AA accessibility compliance

### **User Experience Acceptance**

- [ ] New user completes full workflow in <30 minutes (with guidance)
- [ ] Error messages are clear and actionable
- [ ] Loading states visible for all async operations
- [ ] Confirmation dialogs for destructive actions (delete)
- [ ] User satisfaction score 8+/10

---

## 7. Out of Scope

- **Real-time collaboration**: Multi-user editing deferred
- **Mobile app**: Mobile web only; native apps deferred
- **Advanced visualizations**: Charts and graphs deferred
- **Workflow automation**: No scheduled tasks or triggers
- **Version control UI**: Manual versioning only; Git integration deferred

---

## 8. Testing Strategy

### **Unit Tests** (Jest + React Testing Library)
- Test each component in isolation
- Mock API calls
- Test form validation logic

### **Integration Tests**
- Test full page workflows (e.g., property creation)
- Test API integration with live endpoints

### **E2E Tests** (Playwright)
- Test critical workflows end-to-end
- Schema generation workflow
- Property creation and search

### **Accessibility Tests**
- Automated: axe-core
- Manual: Keyboard navigation, screen reader testing

### **User Acceptance Testing**
- CLP team performs real workflows
- Collect feedback on usability
- Iterate on pain points

---

## 9. Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Poor UX leads to low adoption | High | Medium | Co-design with CLP team; iterative feedback sessions |
| Large file uploads fail | Medium | Medium | Implement chunked uploads; retry logic |
| API errors break UI | Medium | Low | Robust error handling; fallback UI states |
| Slow performance on large datasets | Medium | Low | Pagination; lazy loading; Azure CDN caching |

---

## 10. Success Metrics

### **Adoption Metrics**
- 100% of CLP team uses UI (vs. API/CLI)
- 50+ property creations via UI in first month
- 10+ schemas generated via UI in first month

### **Usability Metrics**
- User completes schema generation in <30 minutes (first time)
- <5 clicks to create new property
- <10 minutes training time for basic workflows
- User satisfaction: 8+/10

### **Performance Metrics**
- Page load: <2s (p95)
- File upload: 50MB in <30s
- API response rendering: <500ms

---

**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025  
**Owner**: PM Agent (spec2cloud workflow)  
**Dependencies**: Property Registry (FRD-001), Schema Generation (FRD-002)
