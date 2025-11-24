# Feature Requirements Document (FRD)

## Feature: AI-Powered Schema Generation

**Feature ID**: FRD-002  
**Phase**: Phase 1 - Foundation  
**Priority**: Critical (Core Value Proposition)  
**Status**: Draft  
**Related PRD Requirements**: [REQ-2], [REQ-7], [REQ-8]

---

## 1. Overview

### **Purpose**
Automate the creation of CLP-compatible JSON schemas for new test types by analyzing raw machine data files and leveraging AI to match properties to the Property Registry. This feature reduces schema creation time from weeks/months to hours.

### **Business Value**
- Reduces schema creation time from 6 months to <4 hours (target)
- Enables rapid onboarding of new test types
- Ensures generated schemas use canonical property names from registry
- Eliminates manual translation of multilingual headers
- Provides foundation for scaling across all Henkel test types

### **User Personas**
- **Schema Owner**: Primary user who uploads files, reviews AI suggestions, and approves schemas
- **Domain Scientist**: Validates scientific accuracy of property mappings
- **Developer**: Consumes generated schemas for CLP integration

---

## 2. Functional Requirements

### **Core Capabilities**

#### **2.1 File Upload & Analysis**

**Supported File Types**
- TXT (tab or comma-delimited)
- CSV (various encodings: UTF-8, Windows-1252, ISO-8859-1)
- Excel (XLSX, XLS)
- PDF (initially simple tables; complex layouts in Parser Builder phase)

**Upload Process**
- Input: Single or multiple files for the same test type (e.g., all Lap Shear variants)
- Validation:
  - File size limit: 50MB per file
  - Maximum 20 files per batch
  - File type validation
- Storage: Temporary storage in Azure Blob Storage during processing
- Output: File processing summary (files accepted, rejected, errors)

**Property Extraction**
- Extract all unique column headers from data tables
- Detect and extract units (embedded in headers or separate rows)
- Identify data types from sample values
- Detect required vs. optional fields (based on null frequency)
- Group properties by category (per-specimen, per-test, time-series)

**Example Input** (Real Zwick Lap Shear with cryptic notation):
```
Auftrags-Nr.;30245;Prüfnorm;ISO 4587;Werkstoff;LapShear-A
a{lo 0};b{lo 0};S{lo 0};Fmax;F{lo max};dL @Fmax;Kommentar
mm;mm;mm²;N;MPa;mm
"12,5";"25,0";"312,5";"6123,45";"19,59";"0,45";"CF"
```

**Alternative Format** (Real Instron output, German):
```
Probenname;Zugscherfestigkeit;Maximalkraft;Fläche
;(N/mm^2);(N);(mm²)
Probe 1;19,59;6123,45;312,5
```

**Expected Extraction**:
- **Zwick Headers**: `a{lo 0}`, `b{lo 0}`, `S{lo 0}`, `Fmax`, `F{lo max}`, `dL @Fmax`, `Kommentar`
- **Instron Headers**: `Zugscherfestigkeit`, `Maximalkraft`, `Fläche`
- **Units**: Handle inline units `(N/mm^2)`, separate row units, and decimal commas `"19,59"`
- **Data Types**: `float`, `float`, `float`, `float`, `float`, `float`, `string`
- **Multi-language**: German (`Zugscherfestigkeit`, `Fläche`) and cryptic notation (`F{lo max}`)

#### **2.2 AI-Powered Property Matching**

**AI Backend Architecture** (Pluggable, Modular Design)

**Primary Option: Microsoft Foundry Platform**
- **Model**: GPT-5.1 or o4-mini (reasoning capabilities for complex property matching)
- **Deployment**: Foundry project with managed model deployment
- **Benefits**: 
  - Integrated prompt flow for multi-stage processing
  - Built-in evaluation and monitoring
  - Agent framework for complex synonym resolution
  - Traceability and observability out-of-the-box
- **Endpoint**: Microsoft Foundry SDK (`azure-ai-projects` Python package)

**Alternative Option: Direct Azure OpenAI Service API**
- **Model**: GPT-4o or GPT-4.1 (multimodal, high-context)
- **Deployment**: Standard Azure OpenAI resource deployment
- **Benefits**:
  - Direct API access for customers requiring it
  - Lower abstraction, more control
  - Simpler authentication (API key or Managed Identity)
- **Endpoint**: Azure OpenAI REST API (`openai` Python SDK)

**Implementation Pattern: AI Service Adapter**
```python
# Abstract interface
class IAIService:
    def generate_property_matches(self, headers, registry_properties) -> List[Match]:
        pass

# Concrete implementations
class FoundryAIService(IAIService):
    # Uses Microsoft Foundry SDK
    pass

class AzureOpenAIService(IAIService):
    # Uses Azure OpenAI API directly
    pass

# Configuration-driven selection
ai_service = create_ai_service(config["ai_backend"])  # "foundry" or "azure_openai"
```

**Model Selection Rationale**:
- **For Henkel (customer)**: Azure OpenAI direct API (GPT-4o or GPT-4.1-mini)
  - Simple, proven, direct control
  - 128K-1M token context for large schemas
- **For Your Platform**: Microsoft Foundry with GPT-5.1 or o4-mini
  - Advanced reasoning for ambiguous property matches
  - Better accuracy on cryptic notation (a{lo 0}, F{lo max})
  - Agent orchestration for multi-stage workflows

**Matching Process**

1. **Query Property Registry**
   - Fetch all canonical properties for the test type (if known)
   - Fetch properties from similar test types (e.g., all shear tests)

2. **AI Prompt Construction**
   - System prompt: Define task (match raw headers to canonical properties)
   - Context: Provide Property Registry canonical names, descriptions, units
   - Input: Raw headers with units extracted from files
   - Instructions:
     - Match each raw header to best canonical property
     - Handle multi-language (German, English)
     - Consider units and context
     - Provide confidence score (0-100%) per match
     - Suggest new property creation if no match >70% confidence

3. **AI Response Parsing**
   - Extract matched pairs: `raw_header → canonical_property`
   - Extract confidence scores
   - Extract justifications
   - Identify unmapped headers (confidence <70%)

**Example AI Matching** (Real Lap Shear samples):

| Raw Header | Unit | AI Matched Property | Confidence | Justification |
|------------|------|---------------------|------------|---------------|
| `F{lo max}` | `MPa` | `Tensile_Shear_Strength_MPa` | 92% | Zwick cryptic notation for max force/area; unit match |
| `Zugscherfestigkeit` | `N/mm^2` | `Tensile_Shear_Strength_MPa` | 98% | German for "tensile shear strength"; N/mm^2 = MPa |
| `Fmax` | `N` | `Maximum_Force_N` | 95% | Common abbreviation; exact unit match |
| `a{lo 0}` | `mm` | `Overlap_Length_mm` | 88% | Zwick notation for overlap dimension 'a'; ISO 4587 standard |
| `b{lo 0}` | `mm` | `Width_mm` | 88% | Zwick notation for width dimension 'b'; ISO 4587 standard |
| `S{lo 0}` | `mm²` | `Cross_Section_Area_mm2` | 85% | Zwick notation for area 'S'; mm² unit match |
| `dL @Fmax` | `mm` | `Displacement_At_Max_mm` | 80% | Displacement at max force; context-based match |
| `Kommentar` | - | `Comment` | 90% | German for "comment"; common metadata field |
| `Maximalkraft` | `N` | `Maximum_Force_N` | 95% | German for "maximum force"; exact unit match |
| `Fläche` | `mm²` | `Cross_Section_Area_mm2` | 92% | German for "area"; unit match |

#### **2.3 User Review & Editing**

**Review Interface** (via Frontend UI)
- Display table with columns:
  - Raw Header
  - Extracted Unit
  - AI Suggested Property (from registry)
  - Confidence Score (color-coded: green >85%, yellow 70-85%, red <70%)
  - Justification
  - Actions: Accept / Edit / Reject / Create New

**Editing Actions**
- **Accept**: Confirm AI suggestion; add to schema
- **Edit**: Choose different property from registry dropdown
- **Reject**: Mark header as "not a property" (e.g., metadata, ID fields)
- **Create New**: Open form to add new property to registry

**Unmapped Header Handling**
- Headers with confidence <70% flagged as "Needs Review"
- User must either:
  - Manually select property from registry
  - Create new property (triggers Property Registry workflow)
  - Mark as "exclude from schema"

#### **2.4 JSON Schema Generation**

**Schema Structure** (CLP-Compatible with Henkel ontology extensions)
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://henkel.example.com/schemas/lap-shear-v1.json",
  "title": "Lap Shear Test Data Schema",
  "description": "Schema for ISO 4587 Lap Shear adhesion test results",
  "type": "object",
  "properties": {
    "Maximum_Force_N": {
      "type": "number",
      "description": "Maximum force applied during test",
      "unit": "N",
      "required": true,
      "source_headers": ["Fmax", "Maximalkraft"],
      "$henkel.property-class": "http://ontobond.henkelgroup.io/ontology/entity#MaximumForce"
    },
    "Tensile_Shear_Strength_MPa": {
      "type": "number",
      "description": "Tensile shear strength",
      "unit": "MPa",
      "required": true,
      "source_headers": ["F{lo max}", "Zugscherfestigkeit"],
      "$henkel.property-class": "http://ontobond.henkelgroup.io/ontology/entity#TensileShearStrength"
    },
    "Overlap_Length_mm": {
      "type": "number",
      "description": "Overlap length dimension a (ISO 4587)",
      "unit": "mm",
      "required": true,
      "source_headers": ["a{lo 0}", "L"],
      "$henkel.property-class": "http://ontobond.henkelgroup.io/ontology/entity#OverlapLength"
    },
    "Width_mm": {
      "type": "number",
      "description": "Specimen width dimension b",
      "unit": "mm",
      "required": true,
      "source_headers": ["b{lo 0}", "b"],
      "$henkel.property-class": "http://ontobond.henkelgroup.io/ontology/entity#SpecimenWidth"
    },
    "Cross_Section_Area_mm2": {
      "type": "number",
      "description": "Cross-sectional area",
      "unit": "mm²",
      "required": true,
      "source_headers": ["S{lo 0}", "A", "Fläche"],
      "$henkel.property-class": "http://ontobond.henkelgroup.io/ontology/entity#CrossSectionArea"
    },
    "Comment": {
      "type": "string",
      "description": "User comment (e.g., CF=cohesive failure, AF=adhesive failure)",
      "required": false,
      "source_headers": ["Kommentar", "Comment"],
      "$henkel.property-class": "http://ontobond.henkelgroup.io/ontology/entity#Comment"
    }
  },
  "required": ["Maximum_Force_N", "Tensile_Shear_Strength_MPa"],
  "metadata": {
    "created_date": "2025-11-24T10:30:00Z",
    "created_by": "schema_owner_123",
    "version": "1.0",
    "source_files": ["Zwick_AMC.txt", "Instron_ACC.csv"],
    "test_standard": "ISO 4587"
  }
}
```

**Generation Logic**
- Include only accepted/edited properties (exclude rejected)
- Use canonical names from Property Registry
- Infer data types from registry + sample values
- Mark properties as required if present in >90% of sample files
- Use canonical names from Property Registry
- Infer data types from registry + sample values
- Mark properties as required if present in >90% of sample files
- Include `source_headers` array to maintain traceability
- Add metadata for audit trail

#### **2.5 Schema Export**

**Export Formats**
- **JSON**: CLP-compatible JSON Schema (primary)
- **YAML**: For human readability and version control
- **Excel**: Tabular view for documentation

**Export Destinations**
- Local download (Phase 1)
- Azure Blob Storage container (Phase 2)
- Direct import to CLP via API (Phase 3)

**Versioning**
- Each schema has version number (semantic versioning: 1.0, 1.1, 2.0)
- Changes tracked in metadata
- Previous versions archived

---

## 3. Technical Architecture

### **Technology Stack (Azure-First)**

#### **AI/ML: Azure OpenAI Service**
- **Model**: GPT-4 or GPT-4 Turbo
- **Endpoint**: Customer-specific Azure OpenAI deployment
- **Pricing**: Pay-per-token (input + output)
- **Rate Limits**: 10K tokens/min (sufficient for schema generation)
- **Prompt Engineering**: Structured prompts with few-shot examples

#### **Backend: Azure Functions (Python)**
- **Function**: `GenerateSchema` (HTTP POST)
- **Triggers**: User uploads files via UI
- **Processing**:
  1. Parse uploaded files
  2. Extract properties
  3. Query Property Registry API
  4. Call Azure OpenAI for matching
  5. Return suggestions to UI
  6. Generate final schema on user approval

#### **File Processing Libraries** (Python)
- **pandas**: CSV/Excel parsing
- **PyPDF2** or **pdfplumber**: PDF text extraction (basic)
- **chardet**: Encoding detection for TXT files
- **openpyxl**: Excel file handling

#### **Storage: Azure Blob Storage**
- **Container**: `schema-generation-uploads`
- **Lifecycle**: Auto-delete after 7 days
- **Container**: `schema-generation-outputs`
- **Lifecycle**: Retain indefinitely; user can manually delete

#### **Caching: Azure Cache for Redis** (Optional, Phase 2)
- Cache Property Registry queries
- Cache AI responses for identical inputs (reduce costs)

### **AI Prompt Design**

**System Prompt**:
```
You are an expert in laboratory data standardization. Your task is to match raw column headers from machine-exported test files to canonical property names from a property registry.

Consider:
- Multi-language headers (German, English)
- Common abbreviations (Fmax = Maximum Force)
- Units (must match or be convertible)
- Scientific context of the test type

Provide confidence scores (0-100%) and justifications for each match.
```

**User Prompt Template**:
```
Test Type: Lap Shear
Standard: ISO 4587

Raw headers extracted from files:
1. "Fmax" (unit: N)
2. "Zugscherfestigkeit" (unit: MPa)
3. "Kommentar" (unit: none)
4. "a{lo 0}" (unit: mm)

Available canonical properties from registry:
- Maximum_Force_N (description: Maximum force applied, unit: N)
- Tensile_Shear_Strength_MPa (description: Tensile shear strength, unit: MPa)
- Comment (description: User observation, unit: none)
- Overlap_Length_mm (description: Length of adhesive overlap, unit: mm)
- Bonding_Thickness_mm (description: Thickness of adhesive layer, unit: mm)

Match each raw header to the best canonical property. Provide confidence and justification.
```

**Expected Response Format** (JSON):
```json
{
  "matches": [
    {
      "raw_header": "Fmax",
      "canonical_property": "Maximum_Force_N",
      "confidence": 95,
      "justification": "Common abbreviation for maximum force; exact unit match (N)"
    },
    {
      "raw_header": "Zugscherfestigkeit",
      "canonical_property": "Tensile_Shear_Strength_MPa",
      "confidence": 98,
      "justification": "German for 'tensile shear strength'; exact unit match (MPa)"
    }
  ]
}
```

---

## 4. Integration Points

### **Inputs**

| Source | Format | Purpose |
|--------|--------|---------|
| User (via UI) | TXT, CSV, Excel, PDF | Raw test files to analyze |
| Property Registry API | JSON | Canonical properties for matching |
| Albert Template Export (optional) | Excel | Additional context for matching |

### **Outputs**

| Destination | Format | Purpose |
|-------------|--------|---------|
| Frontend UI | JSON | Display AI suggestions for user review |
| CLP System | JSON Schema | Import schema for data mapping |
| Azure Blob Storage | JSON, YAML, Excel | Archive generated schemas |
| Audit Log | JSON | Record of all AI decisions |

### **Dependencies**

- **Property Registry API**: Must be operational with populated properties
- **Azure OpenAI Service**: Deployment configured with GPT-4
- **Azure Blob Storage**: Containers provisioned
- **Azure Functions**: Deployed and accessible
- **Sample Files**: Lap Shear test files for initial testing

---

## 5. User Workflows

### **Workflow 1: Generate Schema for Lap Shear (Happy Path)**

**Actor**: Schema Owner

1. Navigate to "Schema Generation" in UI
2. Select test type: "Lap Shear"
3. Upload files:
   - `Zwick_LapShear.txt`
   - `Instron_LapShear.csv`
   - `Henkel_LapShear_Report.pdf`
4. Click "Analyze Files"
5. System processes files, extracts 12 unique headers
6. AI matches 10 headers with >85% confidence, 2 with <70%
7. Review table shows:
   - 10 green rows (high confidence) → all "Accept"
   - 2 yellow rows (low confidence) → manually select from dropdown
8. For unmapped header "NewMetric_X", click "Create New Property"
9. Fill property form, save to registry
10. Return to schema generation, map "NewMetric_X" to new property
11. Click "Generate Schema"
12. Preview JSON schema
13. Click "Export" → download `LapShear_Schema_v1.0.json`

**Acceptance Criteria**:
- File upload completes in <10 seconds
- AI matching completes in <60 seconds for 12 headers
- All 12 headers successfully mapped (AI + manual)
- Generated schema validates against CLP JSON Schema spec
- Export downloads immediately

### **Workflow 2: Handle Multilingual Files**

**Actor**: Schema Owner

1. Upload German Zwick file with headers: "Kraftmaximum", "Zugscherfestigkeit", "Dehnung"
2. Upload English Instron file with headers: "Maximum force", "Tensile shear strength", "Displacement"
3. System detects 6 unique raw headers (3 German, 3 English)
4. AI correctly identifies:
   - "Kraftmaximum" + "Maximum force" → `Maximum_Force_N` (98% confidence)
   - "Zugscherfestigkeit" + "Tensile shear strength" → `Tensile_Shear_Strength_MPa` (98%)
   - "Dehnung" + "Displacement" → `Displacement_mm` (92%)
5. Schema generated with 3 canonical properties, `source_headers` includes both languages

**Acceptance Criteria**:
- AI correctly clusters multilingual synonyms
- Schema uses single canonical property per concept
- `source_headers` array includes all raw variants

### **Workflow 3: Extend Schema for New Machine Type**

**Actor**: Schema Owner

1. Load existing "Lap Shear v1.0" schema (10 properties)
2. Upload new file from previously unsupported machine (Shimadzu)
3. System extracts 12 headers: 8 match existing schema, 4 new
4. AI suggests matches for all 12 headers
5. Review shows:
   - 8 existing properties (green) → auto-accept
   - 3 new headers mapped to existing registry properties (yellow) → accept
   - 1 truly new header → create new property
6. Click "Update Schema" (increments to v1.1)
7. Export updated schema

**Acceptance Criteria**:
- Existing properties preserved
- New properties added without breaking existing mappings
- Version incremented correctly
- Change log documents additions

---

## 6. Acceptance Criteria

### **Functional Acceptance**

- [ ] Upload files in TXT, CSV, Excel formats successfully
- [ ] Extract all unique column headers from uploaded files
- [ ] Detect units from headers or unit rows with 90%+ accuracy
- [ ] Azure OpenAI API returns property matches within 60 seconds
- [ ] AI correctly matches 90%+ of Lap Shear headers (German + English)
- [ ] User can accept, edit, or reject each AI suggestion
- [ ] User can create new properties from unmapped headers
- [ ] Generated JSON schema validates against JSON Schema Draft 07
- [ ] Schema includes all required CLP fields (title, test_type, properties, metadata)
- [ ] Export to JSON, YAML, Excel works without errors
- [ ] Schema versioning increments correctly on updates

### **Non-Functional Acceptance**

- [ ] File upload handles files up to 50MB
- [ ] Processing completes within 2 minutes for 20 files
- [ ] AI matching achieves 90%+ accuracy on Lap Shear test case
- [ ] Azure OpenAI costs <$0.50 per schema generation (estimated)
- [ ] System handles temporary Azure OpenAI rate limit errors gracefully (retry logic)
- [ ] All AI prompts and responses logged for audit

### **Quality Acceptance**

- [ ] Generated schema reviewed by domain scientist confirms scientific accuracy
- [ ] No duplicate properties in schema (canonical names unique)
- [ ] All properties in schema exist in Property Registry
- [ ] Schema imports successfully into CLP without errors

---

## 7. Out of Scope

- **Complex PDF layouts**: Advanced PDF parsing deferred to Parser Builder (FRD-005)
- **Time-series data**: Focus on summary tables; time-series schema deferred
- **Automated schema updates**: Manual user approval required; auto-updates deferred
- **Schema validation against test standards**: Validate structure only; scientific validation manual
- **Multi-test-type schemas**: One schema per test type; combined schemas deferred

---

## 8. Testing Strategy

### **Unit Tests**
- Test file parsing for each format (TXT, CSV, Excel)
- Test header extraction with various delimiters
- Test unit detection (embedded vs. separate rows)
- Mock Azure OpenAI responses to test matching logic

### **Integration Tests**
- End-to-end: upload files → AI matching → schema generation → export
- Test Azure OpenAI API integration (live calls)
- Test Property Registry API queries
- Test Azure Blob Storage upload/download

### **AI Prompt Testing**
- Test with Lap Shear files (German, English, mixed)
- Test with edge cases (typos, unusual abbreviations)
- Measure accuracy: % of correct matches vs. manual baseline
- Measure confidence calibration: high confidence = high accuracy

### **User Acceptance Testing**
- Schema Owner performs Lap Shear schema creation end-to-end
- Domain Scientist validates scientific accuracy of mappings
- Developer imports generated schema into CLP

---

## 9. Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Azure OpenAI accuracy <90% | High | Medium | Iterative prompt engineering; few-shot examples; fallback to manual review |
| Azure OpenAI rate limits exceeded | Medium | Low | Implement retry with exponential backoff; queue processing |
| File parsing fails for unusual formats | Medium | Medium | Graceful error handling; manual upload option for problematic files |
| Property Registry incomplete | High | Medium | Seed registry thoroughly before launch; provide easy "create property" flow |
| User rejects all AI suggestions | Medium | Low | Clear justifications; allow manual mapping; collect feedback to improve prompts |

---

## 10. Success Metrics

### **Performance Metrics**
- Schema generation time: <4 hours (vs. weeks/months currently)
- AI matching accuracy: >90% for Lap Shear
- Processing time: <2 minutes for typical batch (10 files, 20 headers)

### **Adoption Metrics**
- 1+ test type schema generated within 2 weeks
- 3+ test types within 2 months
- 100% of new schemas use this tool (vs. manual creation)

### **Quality Metrics**
- 0 schemas with duplicate properties
- 100% of schemas validate against CLP JSON Schema spec
- <10% user override rate (most AI suggestions accepted)

### **Cost Metrics**
- Azure OpenAI cost per schema: <$0.50
- Total cost for 10 test types: <$50

---

**Document Status**: Draft v1.0  
**Last Updated**: November 24, 2025  
**Owner**: PM Agent (spec2cloud workflow)  
**Dependencies**: Property Registry Management (FRD-001)
