# 📝 Product Requirements Document (PRD)

## Project: AI-Enhanced Laboratory Data Harmonization Platform for Henkel CLP

---

## 1. Purpose

Henkel's research and development teams generate vast amounts of laboratory test data from diverse machines, business units, and testing standards. This data is currently difficult to standardize and integrate into the central LIMS (Albert) due to inconsistent naming conventions, multiple languages, varying units, and heterogeneous file formats. The manual process of creating data schemas and mappings for each new test type takes months and prevents scalability.

**This product solves:**
- The bottleneck in onboarding new test types to the Connected Lab Platform (CLP)
- Inconsistent property naming across machines, languages, and business units
- Manual, error-prone schema creation and data mapping processes
- Lack of data consistency and traceability across experiments

**Who it's for:**
- **Primary users**: CLP development team (product owner, schema owner, developers, domain scientists)
- **Secondary users**: Laboratory scientists and technicians who upload test data
- **Beneficiaries**: R&D leadership seeking cross-project analytics and benchmarking

---

## 2. Scope

### **In Scope:**

**Phase 1 - Foundation (Schema Studio)**
- AI-assisted JSON schema generation for new test types
- Property registry/database to maintain canonical property names and synonyms
- Interactive frontend for uploading raw test files and reviewing AI-generated schemas
- Synonym detection and mapping (e.g., "Viskosität" → "Viscosity")
- Support for Lap Shear test type as the reference implementation
- Extensible architecture to onboard additional test types
- Export schemas compatible with CLP's JSON schema format
- Standalone Python tools and Jupyter notebooks for local execution

**Phase 2 - Intelligent Mapping (Mapping Assistant)**
- AI-suggested mappings from raw file headers → canonical schema properties
- AI-suggested mappings from schema properties → Albert template fields
- Confidence scoring and justification for mapping suggestions
- Multi-language support (English, German, with extensibility)
- Unit normalization and conversion suggestions
- User review and approval workflow
- Export mapping configurations for CLP import

**Phase 3 - Automation (Parser Builder)**
- AI-generated parsing configurations for new machine/file formats
- Support for TXT, CSV, Excel, and PDF formats
- Automatic detection of delimiters, encodings, and header structures
- Extraction of units from header lines or separate unit rows
- Validation and test case generation for parsers

**Cross-Phase Requirements:**
- Property registry maintenance and synonym management
- Data consistency enforcement (same property = same name across all tests)
- Audit logging of schema and mapping decisions
- Integration-ready design for future CLP UI incorporation

### **Out of Scope:**
- Direct integration with Albert LIMS (Phase 1-3)
- Changes to CLP core architecture
- Changes to Albert data templates or structure
- Laboratory hardware or machine software upgrades
- Real-time data streaming from instruments
- Production deployment to cloud infrastructure (Phase 1 uses local execution)
- Semantic analytics layer and scientist assistant (future roadmap)

---

## 3. Goals & Success Criteria

### **Business Goals**
1. **Reduce time-to-onboard** new test types from 6 months to 2-4 weeks
2. **Achieve 100% data consistency** for property names across all experiments
3. **Enable scalability** to support all test types across Henkel's R&D portfolio
4. **Improve data quality** by reducing manual mapping errors by 80%
5. **Unlock analytics** by standardizing data for cross-project benchmarking

### **Success Criteria**

**Phase 1 Success (Schema Studio)**
- [ ] Successfully generate JSON schema for Lap Shear test type using AI
- [ ] Property registry contains 50+ canonical properties from Albert templates
- [ ] AI correctly identifies 90%+ synonyms in German and English
- [ ] Frontend allows users to upload files, review schemas, and export JSON
- [ ] Schema generation time reduced from weeks to hours

**Phase 2 Success (Mapping Assistant)**
- [ ] AI suggests correct mappings with 85%+ accuracy for Lap Shear
- [ ] User can approve/reject mappings through interactive UI
- [ ] Exported mapping configurations successfully imported into CLP
- [ ] Support for 3+ machine types (Zwick, Instron, etc.)

**Phase 3 Success (Parser Builder)**
- [ ] AI generates working parser code for 90%+ of file formats
- [ ] Parsers correctly extract tables and units from PDFs and CSVs
- [ ] Generated parsers pass validation test suites

**Cross-Phase Success**
- [ ] Property naming consistency maintained across 100% of new schemas
- [ ] All AI decisions logged with justifications for audit trail
- [ ] 3+ additional test types onboarded beyond Lap Shear
- [ ] CLP team adoption of tools for day-to-day schema/mapping work

### **Key Performance Indicators (KPIs)**
- Time to create new schema (target: <4 hours vs. current weeks)
- Synonym detection accuracy (target: >90%)
- Mapping suggestion accuracy (target: >85%)
- Property name consistency rate (target: 100%)
- Number of test types onboarded per quarter (target: 6+ vs. current 1-2/year)
- User satisfaction score from CLP team (target: 8+/10)

---

## 4. High-Level Requirements

### **Functional Requirements**

#### **[REQ-1] Property Registry Management**
The system must maintain a centralized registry of canonical properties with:
- Canonical property names (e.g., "Viscosity", "Tensile_Shear_Strength_MPa")
- Descriptions and metadata
- Standard units for each property
- Lists of known synonyms (multi-language)
- Associated test types and standards
- Source reference (e.g., "Albert Lap Shear Template")

The registry must support:
- Querying by property name or synonym
- Adding new properties with user validation
- Merging synonyms under canonical names
- Exporting to JSON/CSV for CLP integration

#### **[REQ-2] AI-Powered Schema Generation**
Given a set of raw test files and an optional Albert template export, the system must:
- Extract all unique property names and units from raw files
- Match properties to existing registry entries using AI-powered synonym detection
- Identify unmapped properties and prompt user for classification
- Generate a CLP-compatible JSON schema with:
  - Property names and data types
  - Units and constraints
  - Required vs. optional flags
  - Structural relationships (per-specimen, per-test, time series)
- Allow user review, editing, and approval of generated schemas
- Export schemas in CLP's expected JSON Schema format

#### **[REQ-3] Intelligent Data Mapping**
The system must suggest mappings between:
- **Raw file headers** → **Canonical schema properties**
- **Canonical schema properties** → **Albert template fields**

For each mapping suggestion, provide:
- Confidence score (0-100%)
- Justification (e.g., "German synonym for 'Maximum Force'; same unit (N); appears in same context")
- Alternative suggestions if confidence is low
- Unit conversion recommendations

Support multi-language detection (German, English) with extensibility for additional languages.

#### **[REQ-4] Parser Generation**
Given raw files in various formats (TXT, CSV, Excel, PDF), the system must:
- Analyze file structure and identify:
  - Delimiter and encoding
  - Header row(s) location
  - Data table boundaries
  - Unit rows or embedded unit information
- Generate parser configuration (YAML) or Python code to extract structured data
- Validate parser output against expected schema
- Support machine-specific quirks (multi-line headers, embedded metadata, etc.)

#### **[REQ-5] Interactive Frontend**
Provide a web-based UI for:
- Uploading raw test files (TXT, CSV, Excel, PDF)
- Viewing extracted properties and suggested mappings
- Searching property registry and adding new properties
- Reviewing and editing AI-generated schemas
- Approving or rejecting mapping suggestions
- Exporting schemas and mappings for CLP import
- Viewing audit logs of decisions

#### **[REQ-6] Data Consistency Enforcement**
The system must:
- Prevent creation of duplicate properties with different names
- Flag potential inconsistencies (e.g., same property with different units)
- Enforce use of canonical names across all new schemas
- Validate that exported schemas comply with CLP format requirements

#### **[REQ-7] Extensibility for New Test Types**
The architecture must support:
- Easy onboarding of new test types beyond Lap Shear
- Reusable property definitions across test types
- Test-specific property groupings and constraints
- Import of existing test schemas for similarity-based generation

### **Non-Functional Requirements**

#### **[REQ-8] Performance**
- Schema generation completes within 2 minutes for typical test files (<100 properties)
- Mapping suggestions generated within 10 seconds
- Frontend UI responsive for file uploads up to 50MB
- Property registry search returns results within 1 second

#### **[REQ-9] Usability**
- Intuitive UI requiring <30 minutes training for CLP team
- Clear AI justifications for all suggestions
- Undo/redo functionality for schema editing
- Export functionality one-click accessible

#### **[REQ-10] Security & Privacy**
- Local execution on user laptops (no cloud data storage in Phase 1)
- Azure OpenAI API calls encrypted in transit
- No sensitive lab data stored by AI models
- Audit logs for all schema and mapping decisions

#### **[REQ-11] Maintainability**
- Modular Python codebase with clear separation of concerns
- Comprehensive unit tests for AI prompt engineering
- Documentation for extending to new test types
- Version control for property registry and schemas

#### **[REQ-12] Integration Readiness**
- Schema and mapping exports compatible with current CLP data format
- Property registry exportable for future Albert integration
- API-ready architecture for future CLP UI integration
- Standard file formats (JSON, CSV, YAML) for all exports

---

## 5. User Stories

### **As a Schema Owner:**

```gherkin
As a schema owner, I want to upload raw Lap Shear files from multiple machines, so that the system can automatically generate a unified JSON schema for me to review.
```

```gherkin
As a schema owner, I want to see which property names are already in the registry, so that I can reuse canonical names and avoid creating duplicates.
```

```gherkin
As a schema owner, I want the AI to suggest canonical names for German headers like "Zugscherfestigkeit", so that I don't have to manually translate and map every field.
```

```gherkin
As a schema owner, I want to approve or edit AI-generated schemas before export, so that I maintain control over data quality.
```

### **As a Developer:**

```gherkin
As a developer, I want parser generation for new machine formats, so that I can quickly integrate new data sources without writing custom code.
```

```gherkin
As a developer, I want exported schemas to be CLP-compatible JSON, so that I can import them directly without manual reformatting.
```

```gherkin
As a developer, I want comprehensive logs of AI decisions, so that I can debug and validate the system's behavior.
```

### **As a Domain Scientist:**

```gherkin
As a domain scientist, I want to validate that AI-suggested property mappings are scientifically accurate, so that data integrity is maintained.
```

```gherkin
As a domain scientist, I want to add new properties to the registry when we develop novel tests, so that the system stays current with our research.
```

### **As a Product Owner:**

```gherkin
As a product owner, I want to see how long schema creation takes, so that I can measure ROI and plan roadmap priorities.
```

```gherkin
As a product owner, I want the system to be extensible to new test types, so that we can scale across all of Henkel's R&D portfolio.
```

```gherkin
As a product owner, I want data consistency enforced across all schemas, so that we can enable cross-project analytics.
```

### **As a Lab Scientist (End User - Future):**

```gherkin
As a lab scientist, I want the system to automatically handle my test uploads, so that I don't need to understand schema or mapping complexity.
```

```gherkin
As a lab scientist, I want consistent property names in Albert, so that I can compare my results with colleagues' experiments.
```

---

## 6. Assumptions & Constraints

### **Assumptions**
- Azure OpenAI API access is available for CLP team members
- Lap Shear raw files and Albert template exports are representative of broader test data
- CLP's current JSON schema format will remain stable during Phase 1-3 development
- Users (Rayhan, Ramandeep, Denis) have Python and Jupyter notebook skills
- Property names in Albert templates are sufficiently canonical to seed the registry
- German and English are the primary languages (extensibility for others is deferred)
- Test standards (ASTM, ISO, DIN) metadata can be stored but detailed normalization is deferred

### **Constraints**
- **Deployment**: Local execution only (laptop-based) in Phase 1-3; no cloud infrastructure
- **Data Access**: No direct Albert API access; must work with exported files (Excel, CSV, JSON)
- **Integration**: No modifications to CLP core architecture or Albert LIMS in Phase 1-3
- **File Formats**: Support limited to TXT, CSV, Excel, PDF (binary formats like proprietary machine files excluded)
- **AI Model**: Must use Azure OpenAI (no on-premises LLMs due to infra constraints)
- **Team Capacity**: CLP development team with limited bandwidth for extensive manual validation
- **Timeline**: Phased approach with Lap Shear as MVP; broader test coverage in subsequent phases

### **Dependencies**
- Azure OpenAI API availability and rate limits
- Access to Albert template exports (Excel files with property names and units)
- Collection of representative raw files from multiple machines and BUs
- CLP team availability for user testing and feedback sessions
- Existing CLP JSON schema format documentation

### **Risks & Mitigation**
- **Risk**: AI mapping accuracy below 85% for edge cases → **Mitigation**: User review workflow; iterative prompt improvement; confidence thresholds
- **Risk**: Property registry becomes inconsistent if not governed → **Mitigation**: Approval workflow for new properties; version control; periodic audits
- **Risk**: Parser generation fails for complex PDF layouts → **Mitigation**: Fallback to manual parser definition; iterative AI prompt tuning
- **Risk**: Slow adoption due to UI/UX friction → **Mitigation**: Co-design with CLP team; prioritize usability in frontend design
- **Risk**: Azure OpenAI API costs exceed budget → **Mitigation**: Caching; batch processing; local fallback for simple mappings

---

## 7. Technical Architecture (High-Level)

### **Components**

1. **Property Registry Service**
   - SQLite database (local) storing canonical properties and synonyms
   - CRUD APIs for property management
   - Export to JSON/CSV for CLP integration

2. **AI Orchestration Layer**
   - Azure OpenAI integration (GPT-4 or Claude via Azure)
   - Prompt templates for schema generation, mapping, parsing
   - Confidence scoring and justification generation

3. **Schema Studio Module**
   - File upload and parsing (TXT, CSV, Excel, PDF)
   - Property extraction and registry lookup
   - Schema generation and validation
   - Export to CLP JSON Schema format

4. **Mapping Assistant Module**
   - Raw header → schema property mapping
   - Schema property → Albert field mapping
   - Unit conversion suggestions
   - Confidence scoring and alternatives

5. **Parser Builder Module**
   - File structure analysis
   - Parser configuration/code generation (YAML or Python)
   - Validation test case generation

6. **Frontend (Web UI)**
   - React or Streamlit-based interface
   - File upload, property search, schema review
   - Mapping approval workflow
   - Export and audit log viewers

7. **CLI & Notebooks**
   - Jupyter notebooks for exploration and testing
   - CLI tools for batch processing and automation

### **Data Flow**

```
[Raw Files] → [Parser Builder] → [Structured Tables]
                                         ↓
                              [Property Registry Lookup]
                                         ↓
                              [Schema Studio (AI)] → [Draft Schema]
                                         ↓
                              [User Review & Edit] → [Approved Schema]
                                         ↓
                              [Mapping Assistant (AI)] → [Suggested Mappings]
                                         ↓
                              [User Approval] → [Final Mapping Config]
                                         ↓
                              [Export to CLP Format] → [CLP Import]
```

---

## 8. Roadmap & Milestones

### **Phase 1: Schema Studio (Weeks 1-6)**
- **Week 1-2**: Property registry setup, seed with Albert Lap Shear properties
- **Week 3-4**: AI schema generation for Lap Shear, prompt engineering
- **Week 5-6**: Frontend UI for upload, review, export; user testing with CLP team

**Deliverables:**
- Property registry database with 50+ entries
- Jupyter notebook for schema generation
- Basic frontend for Lap Shear schema creation
- Lap Shear schema successfully imported into CLP

### **Phase 2: Mapping Assistant (Weeks 7-12)**
- **Week 7-8**: AI mapping engine (raw → schema, schema → Albert)
- **Week 9-10**: Confidence scoring, multi-language synonym detection
- **Week 11-12**: Mapping review UI, export to CLP mapping format

**Deliverables:**
- Mapping suggestions for Lap Shear with 85%+ accuracy
- Mapping review and approval UI
- Exported mapping config imported into CLP

### **Phase 3: Parser Builder (Weeks 13-16)**
- **Week 13-14**: File structure analysis, parser generation AI
- **Week 15-16**: PDF support, validation test generation, integration

**Deliverables:**
- Parser generator for TXT, CSV, Excel, PDF
- Generated parsers for Zwick and Instron formats
- Validation test suite

### **Phase 4: Scale & Integrate (Weeks 17-24)**
- Onboard 3+ additional test types (rheology, viscosity, tensile variants)
- Refine property registry governance
- Prepare for CLP UI integration
- Documentation and training materials

**Deliverables:**
- 4+ test types fully onboarded
- Property registry with 150+ entries
- Integration-ready APIs for CLP
- User documentation and training videos

---

## 9. Open Questions

1. **Property Registry Governance**: Who approves new canonical properties? Is there a review board?
2. **Albert Template Access**: Can we get automated exports of all Albert data templates, or manual per test type?
3. **Multilingual Expansion**: Beyond German/English, what other languages are priorities (French, Spanish, Chinese)?
4. **CLP Integration Timeline**: When does CLP team plan to integrate these tools into the main CLP UI?
5. **Cloud Deployment**: When will Henkel infrastructure support cloud-hosted versions (Azure App Service, etc.)?
6. **AI Model Selection**: GPT-4, Claude Sonnet, or other models for different tasks? Performance vs. cost tradeoffs?

---

## 10. Appendix

### **Reference Materials**
- Lap Shear raw files: Zwick (TXT, PDF), Instron (CSV)
- Albert Lap Shear template: "Lap Shear properties from Albert.xlsx"
- Existing CLP tensile schema (reference for format)
- HAT data model documentation
- CLP data mapping UI screenshots

### **Glossary**
- **CLP**: Connected Lab Platform - middleware between lab instruments and Albert LIMS
- **Albert**: Henkel's Laboratory Information Management System
- **Schema**: JSON Schema defining canonical property structure for a test type
- **Mapping**: Configuration linking raw file headers → schema properties → Albert fields
- **Property Registry**: Database of canonical property names and synonyms
- **HAT JSON**: Henkel's internal data model format for test results
- **Ontology**: (In CLP context) refers to JSON schema, not a formal semantic ontology

---

**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025  
**Owner**: PM Agent (spec2cloud workflow)  
**Reviewers**: CLP Development Team
