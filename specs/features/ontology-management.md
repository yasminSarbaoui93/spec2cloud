# Feature Requirements Document (FRD)

## Feature: Ontology Generation & Management

**Feature ID**: FRD-007  
**Phase**: Future/Advanced (Post Phase 3)  
**Priority**: Medium (Strategic Enhancement)  
**Status**: Draft  
**Related PRD Requirements**: [REQ-7], [REQ-11], [REQ-12]

---

## 1. Overview

### **Purpose**
Build a comprehensive knowledge graph (ontology) that captures semantic relationships between properties, test types, standards, machine formats, units, and domain concepts. This goes beyond the simple Property Registry to enable intelligent synonym resolution, relationship traversal, and semantic querying across Henkel's laboratory data ecosystem.

### **Business Value**
- Automates synonym mapping across multiple languages (German, English, French, etc.)
- Enables semantic search: "Find all properties related to adhesive strength"
- Supports advanced analytics: "Which tests use ISO 4587 standard with Zwick machines?"
- Facilitates knowledge discovery: "What are all known synonyms for viscosity across all sources?"
- Provides foundation for AI-powered data harmonization at scale
- Enables interoperability with external ontologies (e.g., MatOnto, ChemOnto)

### **User Personas**
- **Data Scientist**: Queries ontology for analytics and insights
- **Schema Owner**: Uses ontology for automated synonym resolution
- **Domain Scientist**: Validates and enriches ontology with domain knowledge
- **Product Owner**: Leverages ontology for strategic data harmonization roadmap

---

## 2. Functional Requirements

### **Core Capabilities**

#### **2.1 Knowledge Graph Data Model**

**Entities (Nodes)**
- **Property**: Canonical property (from Property Registry)
- **Synonym**: Alternative name for a property (multi-language)
- **Test Type**: E.g., Lap Shear, Tensile, Rheology
- **Standard**: E.g., ISO 4587, ASTM D905, DIN 53455
- **Machine**: Manufacturer (Zwick, Instron, Shimadzu)
- **Unit**: Measurement unit (MPa, N, mm, Pa·s)
- **Dimension**: Physical dimension (Force, Length, Pressure, Viscosity)
- **Material**: Adhesive formulation or substrate
- **Concept**: High-level domain concept (Strength, Elasticity, Curing)

**Relationships (Edges)**
- **synonym_of**: Synonym → Property (e.g., "Zugscherfestigkeit" synonym_of "Tensile_Shear_Strength_MPa")
- **measured_in**: Property → Unit (e.g., "Tensile_Shear_Strength_MPa" measured_in "MPa")
- **unit_of_dimension**: Unit → Dimension (e.g., "MPa" unit_of_dimension "Pressure")
- **convertible_to**: Unit → Unit (e.g., "MPa" convertible_to "psi", factor: 145.038)
- **used_in_test**: Property → Test Type (e.g., "Maximum_Force_N" used_in_test "Lap Shear")
- **defined_by_standard**: Test Type → Standard (e.g., "Lap Shear" defined_by_standard "ISO 4587")
- **produced_by_machine**: Test Type → Machine (e.g., "Lap Shear" produced_by_machine "Zwick")
- **related_to_concept**: Property → Concept (e.g., "Tensile_Shear_Strength_MPa" related_to_concept "Strength")
- **broader_than**: Concept → Concept (e.g., "Mechanical Properties" broader_than "Strength")
- **measured_on_material**: Test Type → Material (e.g., "Lap Shear" measured_on_material "Epoxy Adhesive")

**Example Graph Structure**:
```
(Synonym:Zugscherfestigkeit) --[synonym_of]--> (Property:Tensile_Shear_Strength_MPa)
(Property:Tensile_Shear_Strength_MPa) --[measured_in]--> (Unit:MPa)
(Unit:MPa) --[unit_of_dimension]--> (Dimension:Pressure)
(Unit:MPa) --[convertible_to {factor:145.038}]--> (Unit:psi)
(Property:Tensile_Shear_Strength_MPa) --[used_in_test]--> (TestType:Lap Shear)
(TestType:Lap Shear) --[defined_by_standard]--> (Standard:ISO 4587)
(TestType:Lap Shear) --[produced_by_machine]--> (Machine:Zwick)
(Property:Tensile_Shear_Strength_MPa) --[related_to_concept]--> (Concept:Adhesive Strength)
```

#### **2.2 AI-Powered Ontology Enrichment**

**Automated Synonym Discovery**
- **Input**: Raw headers from all ingested files
- **Process**:
  1. Extract unique headers across all files
  2. Use Azure OpenAI to cluster synonyms:
     - "Fmax", "Maximum force", "Kraftmaximum" → cluster under `Maximum_Force_N`
  3. Prompt: "Given these headers from adhesion tests, identify synonyms and group them"
  4. Generate `synonym_of` relationships
- **Output**: Enriched ontology with new synonyms

**Relationship Inference**
- **Input**: Existing properties, test types, standards
- **Process**:
  1. Analyze schema metadata (which properties appear in which test types)
  2. Infer `used_in_test` relationships
  3. Use AI to suggest `related_to_concept` relationships
     - Example: "Tensile_Shear_Strength_MPa" likely related to "Strength" concept
- **Output**: New relationships added to graph

**External Ontology Alignment**
- **Input**: External ontologies (e.g., MatOnto for materials science)
- **Process**:
  1. Map Henkel properties to external ontology concepts
  2. Use Azure OpenAI for semantic matching
  3. Create `same_as` or `related_to` relationships
- **Output**: Interoperable ontology

#### **2.3 Semantic Querying**

**Query Interface** (Gremlin or Cypher)

**Example Queries**:

1. **Find all synonyms for a property**:
```gremlin
g.V().hasLabel('Property').has('canonical_name', 'Tensile_Shear_Strength_MPa')
  .in('synonym_of').values('name')
```
Result: `["Zugscherfestigkeit", "Tensile shear strength", "Shear strength", "TSS"]`

2. **Find all properties used in Lap Shear tests**:
```gremlin
g.V().hasLabel('TestType').has('name', 'Lap Shear')
  .in('used_in_test').values('canonical_name')
```
Result: `["Maximum_Force_N", "Tensile_Shear_Strength_MPa", "Overlap_Length_mm", ...]`

3. **Find all units convertible to MPa**:
```gremlin
g.V().hasLabel('Unit').has('symbol', 'MPa')
  .both('convertible_to').values('symbol')
```
Result: `["N/mm²", "psi", "kPa", "bar"]`

4. **Find all tests using ISO standards**:
```gremlin
g.V().hasLabel('Standard').has('name', Text.textContains('ISO'))
  .in('defined_by_standard').values('name')
```
Result: `["Lap Shear", "Peel Test", "Impact Test", ...]`

5. **Semantic property search** (via Azure OpenAI embedding + vector search):
   - Query: "properties related to adhesive bonding quality"
   - AI embeds query, searches vector space of property descriptions
   - Returns: `["Tensile_Shear_Strength_MPa", "Peel_Strength_N_mm", "Tack_Force_N", ...]`

#### **2.4 Ontology Visualization**

**Graph Visualization UI**
- **Technology**: D3.js or Cytoscape.js for interactive graph rendering
- **Features**:
  - Zoom, pan, drag nodes
  - Filter by entity type (show only Properties and Synonyms)
  - Search and highlight nodes
  - Show relationship types on hover
  - Click node to see details
- **Use Case**: Domain scientists explore relationships visually

**Example View**:
- Center node: "Tensile_Shear_Strength_MPa" (Property)
- Connected nodes:
  - Synonyms: "Zugscherfestigkeit", "TSS", "Shear strength"
  - Unit: "MPa"
  - Test Types: "Lap Shear", "Adhesion Test"
  - Concepts: "Mechanical Properties", "Adhesive Strength"

#### **2.5 Ontology Versioning & Governance**

**Version Control**
- Each ontology change tracked with version number
- Export ontology snapshots (RDF, JSON-LD formats)
- Roll back to previous versions if needed

**Approval Workflow**
- AI-suggested relationships flagged as "Pending Review"
- Domain scientists approve or reject
- Approved relationships marked "Verified"
- Confidence score for each relationship

**Change Log**
- Track who added/modified each relationship
- Track why (user justification or AI reasoning)
- Audit trail for compliance

#### **2.6 Integration with Existing Features**

**Property Registry Integration**
- Ontology syncs with Property Registry
- Every registry property becomes a node in ontology
- Synonyms from ontology used in Schema Generation and Mapping Assistant

**Mapping Assistant Enhancement**
- When matching raw header "Viskosität", query ontology:
  ```gremlin
  g.V().has('name', 'Viskosität').out('synonym_of').values('canonical_name')
  ```
- Return: `Viscosity_PaS`
- Use canonical name in mapping

**Schema Generation Enhancement**
- Query ontology for properties commonly used in test type
- Pre-populate AI suggestions with ontology context

---

## 3. Technical Architecture

### **Technology Stack (Azure-First)**

#### **Graph Database: Azure Cosmos DB (Gremlin API)**
- **Why**: Managed graph database, global distribution, scalable, native Azure integration
- **Pricing**: Pay-per-RU (Request Unit); start small, scale as needed
- **API**: Apache TinkerPop Gremlin for graph queries
- **Alternative**: Neo4j on Azure VM (if Cosmos DB Gremlin insufficient)

**Cosmos DB Structure**:
- **Database**: `henkel-ontology`
- **Container**: `knowledge-graph` (partitioned by entity type)
- **Vertices**: Properties, Synonyms, Test Types, Standards, etc.
- **Edges**: Relationships as defined above

#### **AI Enrichment: Azure OpenAI Service**
- **Use Cases**:
  - Synonym clustering
  - Relationship inference
  - Semantic matching to external ontologies
- **Embeddings**: text-embedding-ada-002 for semantic search

#### **Vector Search: Azure AI Search** (Optional)
- **Use Case**: Semantic property search by description
- **Index**: Property descriptions as vectors
- **Query**: Natural language → embedding → vector search

#### **Visualization: React + Cytoscape.js**
- **Frontend**: React component for graph rendering
- **Library**: Cytoscape.js (powerful graph visualization)
- **Integration**: Query Cosmos DB via API, render in browser

#### **Backend API: Azure Functions (Python)**
- **Functions**:
  - `QueryOntology` (HTTP POST with Gremlin query)
  - `AddRelationship` (HTTP POST)
  - `EnrichOntology` (Batch processing with AI)
  - `ExportOntology` (RDF, JSON-LD export)

#### **Data Formats**
- **Internal**: Cosmos DB Gremlin graph
- **Export**: RDF/OWL, JSON-LD (standard ontology formats)
- **Import**: CSV (for bulk synonym upload), RDF (for external ontologies)

---

## 4. Integration Points

### **Inputs**

| Source | Format | Purpose |
|--------|--------|---------|
| Property Registry | JSON API | Sync canonical properties |
| Schema Generation | Raw headers | Extract new synonyms |
| Mapping Assistant | Mapping history | Learn synonym relationships |
| External Ontologies | RDF/OWL | Align with industry standards |
| Domain Scientists | Manual entry | Add verified relationships |

### **Outputs**

| Destination | Format | Purpose |
|-------------|--------|---------|
| Mapping Assistant | Gremlin query results | Synonym resolution |
| Schema Generation | Gremlin query results | Property suggestions |
| Analytics Tools | JSON, RDF | Advanced queries |
| Frontend UI | JSON | Graph visualization |
| Research Community | RDF/OWL | Share Henkel ontology |

### **Dependencies**

- **Azure Cosmos DB (Gremlin API)**: Must be provisioned
- **Property Registry**: Sync properties to ontology
- **Azure OpenAI**: For enrichment
- **Optional: Azure AI Search**: For semantic search

---

## 5. User Workflows

### **Workflow 1: Discover Synonyms for Property**

**Actor**: Schema Owner

1. Navigate to "Ontology Explorer" in UI
2. Search for property: "Tensile_Shear_Strength_MPa"
3. Click on property node
4. See connected synonym nodes:
   - "Zugscherfestigkeit" (German)
   - "Tensile shear strength" (English)
   - "TSS" (Abbreviation)
   - "Shear strength" (Variant)
5. Use this knowledge when reviewing AI mapping suggestions

**Acceptance Criteria**:
- Search finds property quickly
- All synonyms displayed
- Can navigate graph visually

### **Workflow 2: AI Enriches Ontology with New Synonyms**

**Actor**: System (Automated)

1. Schema Generation processes 50 new files
2. Extracts 200 unique raw headers
3. Ontology Enrichment job runs nightly
4. AI clusters headers by similarity:
   - Cluster 1: "Fmax", "Maximum force", "Kraftmaximum", "Peak force"
   - Cluster 2: "Viscosity", "Viskosität", "Visc", "η"
5. AI suggests relationships:
   - "Kraftmaximum" → synonym_of → "Maximum_Force_N" (confidence: 95%)
   - "Viskosität" → synonym_of → "Viscosity_PaS" (confidence: 98%)
6. Relationships flagged as "Pending Review"
7. Domain scientist reviews, approves
8. Relationships added to ontology

**Acceptance Criteria**:
- AI correctly clusters synonyms
- Suggestions have justifications
- Approval workflow works
- Approved relationships appear in graph

### **Workflow 3: Query Ontology for Test Type Properties**

**Actor**: Data Scientist

1. Want to analyze all Lap Shear properties across projects
2. Use Ontology Query API:
   ```gremlin
   g.V().hasLabel('TestType').has('name', 'Lap Shear')
     .in('used_in_test').values('canonical_name')
   ```
3. API returns list of 15 properties
4. Use list to query Albert for data
5. Perform cross-project analysis

**Acceptance Criteria**:
- Query returns correct results
- API responds within 2 seconds
- Results usable in downstream analytics

---

## 6. Acceptance Criteria

### **Functional Acceptance**

- [ ] Cosmos DB Gremlin graph populated with 100+ properties
- [ ] Synonyms added for 50+ properties (German + English)
- [ ] Relationships defined: synonym_of, measured_in, used_in_test, etc.
- [ ] Gremlin queries return correct results
- [ ] AI enrichment job adds new synonyms automatically
- [ ] Graph visualization renders correctly in UI
- [ ] Export to RDF/JSON-LD works without errors

### **Non-Functional Acceptance**

- [ ] Gremlin queries return results in <2 seconds
- [ ] Graph visualization loads for graphs with 500+ nodes
- [ ] AI enrichment job completes overnight (batch processing)
- [ ] Cosmos DB costs <$100/month (Phase 1 scale)

### **Quality Acceptance**

- [ ] 90%+ of AI-suggested synonyms confirmed correct by domain scientists
- [ ] No orphaned nodes (all entities connected)
- [ ] Ontology validates against OWL 2 spec (if using OWL export)

---

## 7. Out of Scope

- **Full-fledged ontology editor**: Advanced ontology authoring tools deferred (use specialized tools like Protégé)
- **Reasoning engine**: OWL inference deferred (simple graph traversal only)
- **Real-time ontology updates**: Batch enrichment only; real-time deferred
- **Multi-tenancy**: Single Henkel ontology; partitioning for BUs deferred

---

## 8. Testing Strategy

### **Unit Tests**
- Test Gremlin query construction
- Mock Cosmos DB responses
- Test synonym clustering logic

### **Integration Tests**
- Add nodes/edges to Cosmos DB
- Query ontology via API
- Export ontology to RDF, validate format

### **AI Enrichment Tests**
- Run enrichment on sample data
- Measure synonym clustering accuracy
- Validate suggested relationships

### **User Acceptance Testing**
- Domain scientists review AI-suggested synonyms
- Data scientists use ontology for analytics queries

---

## 9. Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Ontology becomes too complex to maintain | High | Medium | Clear governance; regular audits; versioning |
| Cosmos DB costs escalate | Medium | Medium | Monitor RU usage; optimize queries; archive old data |
| AI suggests incorrect relationships | Medium | Medium | Approval workflow; confidence thresholds |
| Graph visualization performance issues | Low | Medium | Limit displayed nodes; lazy loading; server-side rendering |

---

## 10. Success Metrics

### **Adoption Metrics**
- 200+ properties in ontology within 6 months
- 500+ synonyms defined (multilingual)
- 10+ users querying ontology monthly

### **Quality Metrics**
- 90%+ accuracy for AI synonym suggestions
- 95%+ of relationships verified by domain scientists
- 0 dangling relationships (orphaned nodes)

### **Performance Metrics**
- Query response time: <2 seconds (p95)
- Graph visualization load time: <5 seconds for 500 nodes
- Enrichment job: Completes overnight for 1000 new headers

### **Business Impact**
- 50%+ reduction in manual synonym mapping effort
- Enables cross-project analytics (previously impossible)
- Foundation for AI-powered data harmonization at scale

---

## 11. Future Enhancements

### **Phase 4+ Ideas**
- **Federated Ontology**: Integrate with external material science ontologies (MatOnto, ChemOnto)
- **Ontology-Driven Code Generation**: Auto-generate parsers and mappings from ontology
- **Machine Learning on Graph**: Predict missing relationships using graph neural networks
- **Natural Language Interface**: "What properties measure adhesive strength?" → Query ontology → Return results
- **Regulatory Compliance**: Map properties to FDA/ISO requirements via ontology

---

**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025  
**Owner**: PM Agent (spec2cloud workflow)  
**Dependencies**: Property Registry (FRD-001), Schema Generation (FRD-002), Mapping Assistant (FRD-004)
