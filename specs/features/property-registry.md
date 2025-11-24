# Feature Requirements Document (FRD)

## Feature: Property Registry Management

**Feature ID**: FRD-001  
**Phase**: Phase 1 - Foundation  
**Priority**: High (Foundational)  
**Status**: Draft  
**Related PRD Requirements**: [REQ-1], [REQ-6], [REQ-7]

---

## 1. Overview

### **Purpose**
Provide a centralized registry that stores canonical property names **exactly as they are expected by Albert LIMS**. This registry serves as the single source of truth for property definitions, ensuring 100% consistency across all test types and schemas.

### **Business Value**
- Eliminates duplicate property definitions with inconsistent naming
- Ensures all schemas reference the same canonical property names used by Albert
- Enables reuse of property definitions across multiple test types
- Provides foundation for schema generation and mapping features
- Supports data quality and consistency enforcement

### **User Personas**
- **Schema Owner**: Queries registry when creating new schemas; adds new properties as tests evolve
- **Developer**: Uses registry API for programmatic access; exports for CLP integration
- **Domain Scientist**: Validates that property definitions match scientific standards
- **Product Owner**: Monitors registry growth and adoption metrics

---

## 2. Functional Requirements

### **Core Capabilities**

#### **2.1 Property Storage**
The registry must store for each property:
- **Canonical Name**: Exact property name as used in Albert (e.g., `Tensile_Shear_Strength_MPa`, `Viscosity`)
- **Description**: Human-readable definition of what the property measures
- **Standard Unit**: Canonical unit for this property (e.g., `MPa`, `Pa·s`, `mm`)
- **Data Type**: Expected data type (`float`, `integer`, `string`, `boolean`)
- **Test Types**: List of test types that use this property (e.g., `["Lap Shear", "Tensile"]`)
- **Albert Template Reference**: Source template name in Albert (e.g., `"Albert Lap Shear Template"`)
- **Required/Optional**: Flag indicating if property is mandatory for its test types
- **Validation Rules**: Min/max ranges, regex patterns, or other constraints
- **Created Date**: Timestamp when property was added
- **Last Modified**: Timestamp of last update
- **Created By**: User who added the property

#### **2.2 CRUD Operations**

**Create Property**
- Input: All property fields (name, description, unit, type, etc.)
- Validation:
  - Canonical name must be unique (case-sensitive)
  - Name must match Albert naming convention (alphanumeric + underscore)
  - Standard unit must be valid (validated against unit catalog)
  - Data type must be from allowed list
- Output: Property ID and confirmation
- Error handling: Reject duplicates with clear error message

**Read/Query Property**
- Query by:
  - Exact canonical name
  - Partial name match (search)
  - Test type
  - Unit
  - Albert template reference
- Return: Full property details or list of matching properties
- Performance: Search results within 1 second

**Update Property**
- Input: Property ID + fields to update
- Validation: Same as Create, plus audit trail requirement
- Output: Updated property + change log entry
- Restrictions: Canonical name changes require special approval workflow

**Delete Property**
- Input: Property ID
- Validation: Check if property is referenced in any existing schemas
- Output: Deletion confirmation or error if in use
- Safety: Soft delete (mark as inactive) rather than hard delete

#### **2.3 Property Search & Discovery**

**Full-Text Search**
- Search across: canonical name, description, test types
- Support wildcards and partial matches
- Return results ranked by relevance
- Performance: <1 second for any query

**Filter & Sort**
- Filter by: test type, unit, data type, required/optional
- Sort by: name (alphabetical), creation date, usage count
- Combine multiple filters (AND logic)

**Property Suggestions**
- Input: Partial property name or description
- Output: Top 5 most likely matches with confidence scores
- Use case: Help users find existing properties before creating new ones

#### **2.4 Bulk Operations**

**Bulk Import**
- Input: CSV or JSON file with property definitions
- Use case: Seed registry from Albert template exports
- Validation: Same rules as Create, with batch error reporting
- Output: Import summary (success count, errors with line numbers)

**Bulk Export**
- Output formats: JSON, CSV, Excel
- Filters: Export all or filtered subset
- Use case: Share with CLP team, backup, version control
- Include metadata: export timestamp, user, filter criteria

#### **2.5 Data Consistency Enforcement**

**Duplicate Prevention**
- Before creating property, check for:
  - Exact canonical name match (reject)
  - Similar names (warn user, suggest existing)
- Provide "merge" workflow if user confirms duplicate

**Referential Integrity**
- Track which schemas reference each property
- Prevent deletion of properties in use
- Show usage count in property details

**Unit Validation**
- Maintain separate unit catalog with real units from Lap Shear samples:
  - Pressure/Strength: `MPa`, `N/mm²`, `N/mm^2` (equivalent notations)
  - Force: `N`
  - Length: `mm`
  - Area: `mm²`, `mm^2` (equivalent notations)
  - Others: `psi`, `Pa·s` (viscosity), `s` (time)
- Only allow units from catalog
- Support unit conversion metadata (future integration with Mapping Assistant)
- Handle notation variations: `mm²` = `mm^2`, `N/mm²` = `MPa`

---

## 3. Technical Architecture

### **Technology Stack (Azure-First)**

#### **Database: Azure SQL Database** (Recommended)
- **Why**: Structured data with ACID guarantees; excellent for relational property definitions
- **Tier**: Basic or Standard (start small, scale as needed)
- **Schema**: Single `properties` table with indexes on name, test_type, unit
- **Alternative**: Azure Cosmos DB (SQL API) if NoSQL flexibility preferred

#### **Backend API: Azure Functions (Python)**
- **Why**: Serverless, cost-effective for Phase 1 local execution; easy to migrate to cloud later
- **Functions**:
  - `CreateProperty` (HTTP POST)
  - `GetProperty` (HTTP GET)
  - `SearchProperties` (HTTP GET with query params)
  - `UpdateProperty` (HTTP PUT)
  - `DeleteProperty` (HTTP DELETE)
  - `BulkImport` (HTTP POST with file upload)
  - `BulkExport` (HTTP GET with format param)
- **Local Development**: Azure Functions Core Tools for laptop execution

#### **Storage: Azure Blob Storage**
- **Use case**: Store bulk import/export files temporarily
- **Container**: `property-registry-exports`

#### **Search: Azure AI Search** (Future Enhancement)
- **Use case**: Advanced full-text search with ranking
- **Phase 1**: Use SQL `LIKE` queries; migrate to Azure AI Search if performance issues

#### **Monitoring: Azure Application Insights**
- Track API call latency, error rates, usage patterns
- Set up alerts for failures or slow queries

### **Data Model**

```sql
CREATE TABLE properties (
    property_id         UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    canonical_name      NVARCHAR(255) NOT NULL UNIQUE,
    description         NVARCHAR(1000),
    standard_unit       NVARCHAR(50) NOT NULL,
    data_type           NVARCHAR(50) NOT NULL CHECK (data_type IN ('float', 'integer', 'string', 'boolean', 'datetime')),
    test_types          NVARCHAR(500),  -- JSON array stored as string
    albert_template_ref NVARCHAR(255),
    is_required         BIT DEFAULT 0,
    validation_rules    NVARCHAR(MAX),  -- JSON object
    created_date        DATETIME2 DEFAULT GETUTCDATE(),
    last_modified       DATETIME2 DEFAULT GETUTCDATE(),
    created_by          NVARCHAR(255),
    is_active           BIT DEFAULT 1,
    usage_count         INT DEFAULT 0
);

CREATE INDEX idx_canonical_name ON properties(canonical_name);
CREATE INDEX idx_test_types ON properties(test_types);
CREATE INDEX idx_unit ON properties(standard_unit);
```

### **API Design**

**Base URL**: `http://localhost:7071/api` (local) or `https://<function-app>.azurewebsites.net/api` (cloud)

#### **Endpoints**

```
POST   /properties              Create new property
GET    /properties/{id}         Get property by ID
GET    /properties              Search/list properties (query params: name, test_type, unit)
PUT    /properties/{id}         Update property
DELETE /properties/{id}         Delete property
POST   /properties/bulk-import  Upload CSV/JSON for bulk import
GET    /properties/export       Export properties (query param: format=json|csv|excel)
GET    /properties/suggest      Get suggestions (query param: q=partial_name)
```

#### **Example Request/Response**

**Real Lap Shear Properties** (from actual Albert export):
- `Maximum_Force_N` - Maximum force applied during test
- `Tensile_Shear_Strength_MPa` - Tensile shear strength
- `Overlap_Length_mm` - Length of adhesive overlap (dimension a)
- `Width_mm` - Width of specimen (dimension b)
- `Cross_Section_Area_mm2` - Cross-sectional area (S or A)
- `Displacement_At_Max_mm` - Displacement at maximum force
- `Comment` - User observation or test note

**POST /properties**
```json
{
  "canonical_name": "Tensile_Shear_Strength_MPa",
  "description": "Maximum tensile shear strength measured during lap shear test",
  "standard_unit": "MPa",
  "data_type": "float",
  "test_types": ["Lap Shear", "Adhesion Test"],
  "albert_template_ref": "Albert Lap Shear Template",
  "is_required": true,
  "validation_rules": {
    "min": 0,
    "max": 100,
    "decimal_places": 2
  },
  "known_synonyms_note": "Handled by Ontology (FRD-007): Zugscherfestigkeit, F{lo max}, Tensile shear strength"
}
```

**Response (201 Created)**
```json
{
  "property_id": "a3f2e8c1-4b5d-4c3e-9a2b-1c3d4e5f6a7b",
  "canonical_name": "Tensile_Shear_Strength_MPa",
  "message": "Property created successfully",
  "created_date": "2025-11-24T10:30:00Z"
}
```

---

## 4. Integration Points

### **Inputs**

| Source | Format | Frequency | Purpose |
|--------|--------|-----------|---------|
| Albert Template Exports | Excel/CSV | One-time + ad-hoc | Seed registry with existing properties |
| Schema Owner (Manual Entry) | JSON via UI/API | As needed | Add new properties for novel tests |
| Bulk Import Files | CSV/JSON | Occasional | Migrate properties from other systems |

### **Outputs**

| Destination | Format | Frequency | Purpose |
|-------------|--------|-----------|---------|
| Schema Generation Module | JSON API | Per schema creation | Query properties for matching |
| Mapping Assistant | JSON API | Per mapping session | Provide target property list |
| CLP System | JSON/CSV Export | On-demand | Integration with CLP data model |
| Frontend UI | JSON API | Real-time | Display in search/browse interfaces |

### **Dependencies**

- **Azure SQL Database**: Must be provisioned and accessible
- **Azure Functions Runtime**: Python 3.9+ with Azure Functions Core Tools
- **Unit Catalog**: Separate configuration file or table (can be static initially)
- **Albert Template Access**: Need Excel exports from Albert to seed registry

---

## 5. User Workflows

### **Workflow 1: Seed Registry from Albert Templates**

**Actor**: Schema Owner

1. Export properties from Albert Lap Shear template to Excel
2. Convert Excel to CSV with columns: `canonical_name`, `description`, `standard_unit`, `data_type`, `test_types`, `albert_template_ref`
3. Use bulk import API or UI to upload CSV
4. Review import summary (50 properties created, 2 duplicates skipped)
5. Manually review and edit any properties with warnings

**Acceptance Criteria**:
- Import completes within 30 seconds for 100 properties
- Duplicates detected and skipped with clear error messages
- All valid properties created with correct data types

### **Workflow 2: Add New Property for Novel Test**

**Actor**: Schema Owner + Domain Scientist

1. Schema Owner searches registry: "Is 'Gel_Time' already defined?"
2. Search returns empty results
3. Schema Owner clicks "Add New Property"
4. Fills form:
   - Canonical Name: `Gel_Time_Seconds`
   - Description: "Time taken for adhesive to gel at room temperature"
   - Standard Unit: `s` (seconds)
   - Data Type: `float`
   - Test Types: `["Curing Test"]`
   - Albert Template: `"Albert Curing Template v1.0"`
5. Domain Scientist reviews and approves
6. Property saved to registry

**Acceptance Criteria**:
- Search correctly returns no results for new properties
- Form validates all required fields
- Property appears immediately in search after creation

### **Workflow 3: Reuse Existing Property in New Schema**

**Actor**: Schema Generation Module (automated)

1. Schema Generation extracts "Viscosity" from raw file
2. Queries Property Registry API: `GET /properties?name=Viscosity`
3. Receives match: `Viscosity_PaS` (canonical name)
4. Uses canonical name in generated schema
5. Logs property reuse for audit trail

**Acceptance Criteria**:
- API returns result within 500ms
- Canonical name used consistently in schema output

### **Workflow 4: Export Registry for CLP Integration**

**Actor**: Developer

1. Developer calls: `GET /properties/export?format=json&test_type=Lap%20Shear`
2. API generates JSON file with all Lap Shear properties
3. Developer downloads file
4. Imports into CLP configuration

**Acceptance Criteria**:
- Export completes within 10 seconds for 500 properties
- JSON format matches CLP import specification
- File includes metadata (export date, filter criteria)

---

## 6. Acceptance Criteria

### **Functional Acceptance**

- [ ] Create property API successfully adds properties with all required fields
- [ ] Duplicate canonical names are rejected with appropriate error message
- [ ] Search returns results within 1 second for any query
- [ ] Bulk import processes 100 properties in <30 seconds
- [ ] Export generates valid JSON/CSV files compatible with CLP
- [ ] Property updates create audit log entries
- [ ] Delete attempts on in-use properties are blocked with explanation
- [ ] All APIs return proper HTTP status codes and error messages

### **Non-Functional Acceptance**

- [ ] API response time <500ms for single property queries (95th percentile)
- [ ] Search response time <1 second for any query (99th percentile)
- [ ] System handles 1000 properties without performance degradation
- [ ] Azure SQL Database queries optimized with proper indexes
- [ ] Azure Application Insights tracking all API calls
- [ ] API documentation complete with OpenAPI/Swagger spec
- [ ] Unit tests cover 80%+ of API logic

### **Data Quality Acceptance**

- [ ] No duplicate canonical names exist in registry
- [ ] All properties have valid standard units from unit catalog
- [ ] All properties reference valid Albert templates
- [ ] Property naming follows consistent convention (CamelCase_with_Unit)
- [ ] 50+ properties seeded from Albert Lap Shear template

---

## 7. Out of Scope

- **Synonym management**: Handled by Ontology Management feature (FRD-007)
- **Property relationship modeling**: Graph database in Ontology feature
- **Multi-language property names**: Only canonical English names stored
- **Version history for properties**: Simple audit log only; full versioning deferred
- **Complex validation rules engine**: Basic min/max/regex only; advanced rules deferred
- **Direct Albert API integration**: Work with exported files only in Phase 1

---

## 8. Testing Strategy

### **Unit Tests**
- Test each CRUD operation with valid/invalid inputs
- Test duplicate detection logic
- Test search and filter combinations
- Test bulk import with various CSV formats

### **Integration Tests**
- Test Azure SQL Database connection and queries
- Test Azure Functions local runtime execution
- Test file upload to Azure Blob Storage
- Test Application Insights telemetry

### **User Acceptance Tests**
- Schema Owner performs full workflow: seed from Albert, add new property, search, export
- Developer tests API programmatically from Python client
- Domain Scientist validates property definitions match scientific standards

### **Performance Tests**
- Load test: 100 concurrent API requests
- Stress test: 10,000 properties in database
- Search performance with various query patterns

---

## 9. Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Albert template exports inconsistent | High | Medium | Work with schema owner to standardize export format; create import validation rules |
| Property naming conventions not enforced | High | Medium | Implement strict validation rules; provide naming guidelines documentation |
| Database schema changes break existing integrations | Medium | Low | Version API endpoints; provide migration scripts |
| Performance degrades with large registry | Medium | Medium | Use Azure SQL indexing; consider Azure AI Search if needed |
| Duplicate properties created by different users | High | Medium | Real-time duplicate detection; approval workflow for new properties |

---

## 10. Success Metrics

### **Adoption Metrics**
- 50+ properties in registry within 2 weeks of launch
- 100% of new schemas reference registry properties (no manual property definitions)
- 3+ test types using shared properties from registry

### **Quality Metrics**
- 0 duplicate canonical names in registry
- 100% of properties have valid units and Albert references
- <5% property update/correction rate (indicates good initial quality)

### **Performance Metrics**
- API response time: p95 <500ms, p99 <1s
- Search response time: p99 <1s
- Bulk import throughput: >100 properties/30s

### **User Satisfaction**
- Schema owners report <5 minutes to add new property (vs. previous undefined process)
- Developers report easy API integration (<2 hours to implement client)
- 8+/10 satisfaction score from CLP team

---

## 11. Implementation Notes

### **Phase 1 (Local Execution)**
- Use Azure Functions Core Tools to run API locally
- Use Azure SQL Database Emulator or lightweight Azure SQL tier
- Store exports in local filesystem (Blob Storage integration deferred)

### **Phase 2 (Cloud Migration)**
- Deploy Azure Functions to Azure Functions Premium Plan
- Enable Azure Application Insights monitoring
- Configure Azure Blob Storage for exports
- Set up Azure API Management for rate limiting and security

### **Phase 3 (Advanced Features)**
- Integrate with Ontology Management (FRD-007) for synonym resolution
- Add Azure AI Search for semantic search capabilities
- Implement property versioning with Azure Cosmos DB change feed
- Add role-based access control with Azure AD integration

---

**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025  
**Owner**: PM Agent (spec2cloud workflow)  
**Dependencies**: None (foundational feature)
