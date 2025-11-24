# Sample Data Analysis Summary

## Files Analyzed

### Lap Shear Raw Files
1. **Instron (ACC.csv)** - German format, semicolon-delimited
2. **Zwick Roell (AMC.TXT)** - English headers, semicolon-delimited, cryptic abbreviations
3. **Zwick Roell (AMC 2.TXT)** - English headers, full names, semicolon-delimited
4. **Zwick Roell (AMC 3.TXT)** - (not yet analyzed)
5. **Zwick Roell (AID.pdf & AMO.pdf)** - PDF reports

### Schema Samples
- **tensile-test.schema.json** - Existing CLP schema with ontology references
- **core.schema.json, hierarchy.schema.json, units.schema.json** - Supporting schemas

### Albert Export
- **Lap Shear properties from Albert.xlsx** - Canonical property names

---

## Key Findings

### 1. **Naming Convention Variations**

#### **Same Property, Different Names:**

| Property Concept | Instron (German) | Zwick AMC (Cryptic) | Zwick AMC 2 (Full) | Likely Albert Canonical |
|------------------|------------------|---------------------|---------------------|------------------------|
| Maximum Force | (in summary only) | `Fmax` | `Maximum force` | `Maximum_Force_N` |
| Tensile Shear Strength | `Zugscherfestigkeit` | `F{lo max}` | `Tensile shear strength` | `Tensile_Shear_Strength_MPa` |
| Overlap Length (a) | - | `a{lo 0}` | `L` | `Overlap_Length_mm` |
| Width (b) | - | `b{lo 0}` | `b` | `Width_mm` |
| Area | - | `S{lo 0}` | `A` | `Cross_Section_Area_mm2` |
| Displacement at Max | - | `dL @Fmax` | - | `Displacement_At_Max_mm` |
| Comment | - | `Kommentar` | `Comment` | `Comment` |

#### **Language Variations:**
- **German**: "Zugscherfestigkeit", "Kommentar", "Prüfnorm", "Werkstoff"
- **English**: "Maximum force", "Tensile shear strength", "Comment", "Test standard"

#### **Notation Styles:**
- **Cryptic**: `F{lo max}`, `a{lo 0}`, `S{lo 0}`, `dL @Fmax`
- **Abbreviations**: `Fmax`, `L`, `b`, `A`
- **Full Names**: "Maximum force", "Tensile shear strength"

### 2. **Unit Representation**

#### **Inline Units (Zwick AMC):**
```
a{lo 0};b{lo 0};S{lo 0};Fmax  ;F{lo max};Kommentar;dL @Fmax
mm     ;mm     ;mm²    ;N     ;MPa      ;         ;mm
```
- Units on **separate row** after headers
- Special characters: `mm²` (superscript)

#### **Embedded Units (Zwick AMC 2):**
```
Maximum force;Tensile shear strength;L   ;b   ;A   ;Comment
N            ;MPa                   ;mm  ;mm  ;mm² ;
```
- Units on **separate row**

#### **Embedded in Headers (Instron):**
```
;Zugscherfestigkeit 
;(N/mm^2)
```
- Unit in **parentheses on next line** after header

#### **Unit Variations:**
- `N/mm²` vs `N/mm^2` vs `MPa` (all same dimension)
- `mm²` vs `mm^2` (area)

### 3. **File Structure Patterns**

#### **Zwick AMC.TXT:**
```
[Metadata Block]
Auftrags-Nr.: "..."
Prüfnorm: ""
Werkstoff: "..."
...

[Data Table]
header1;header2;header3;...
unit1  ;unit2  ;unit3  ;...
value1 ;value2 ;value3 ;...
```

#### **Zwick AMC 2.TXT:**
```
[Metadata Block]
Heading: "Test report"
Test standard: "DIN EN 1465"
...

[Data Table]
header1;header2;header3;...
unit1  ;unit2  ;unit3  ;...
value1 ;value2 ;value3 ;...
```

#### **Instron CSV:**
```
Ergebnistabelle 1
;Zugscherfestigkeit 
;(N/mm^2)
1;"3,23"
2;"3,88"
...
Mittelwert;"3,57"
```
- Summary statistics (Mittelwert=mean, Minimale=min, Maximale=max)
- German decimal separator (comma)

### 4. **Existing Schema Pattern (tensile-test.schema.json)**

Key characteristics:
- Uses **JSON Schema Draft 2020-12**
- Custom property: `$henkel.property-class` with ontology URI
- Example: `"$henkel.property-class": "http://ontobond.henkelgroup.io/ontology/entity#tensile_test_document"`
- Hierarchical structure: aggregate document → test document → measurements
- References external schemas via `$ref`

**Property Structure:**
```json
"analyst": {
  "$henkel.property-class": "http://ontobond.henkelgroup.io/ontology/entity#analyst",
  "type": "string"
},
"test standard": {
  "$henkel.property-class": "http://ontobond.henkelgroup.io/ontology/entity#test_standard",
  "type": "string"
}
```

---

## Implications for FRDs

### **Property Registry (FRD-001)**
✅ Must handle multi-language synonyms from day 1  
✅ Unit catalog needs: `MPa`, `N/mm²`, `N/mm^2`, `mm`, `mm²`, `mm^2`, `N`  
✅ Property naming examples based on real Albert export needed

### **Schema Generation (FRD-002)**
✅ AI must handle cryptic notations: `F{lo max}`, `a{lo 0}`, `dL @Fmax`  
✅ Multi-line header detection: units on separate row  
✅ German→English translation: "Zugscherfestigkeit" → "Tensile Shear Strength"  
✅ Generated schemas must include `$henkel.property-class` ontology references

### **Mapping Assistant (FRD-004)**
✅ Example mappings:
- `Fmax` → `Maximum_Force_N`
- `Zugscherfestigkeit` → `Tensile_Shear_Strength_MPa`
- `F{lo max}` → `Tensile_Shear_Strength_MPa`
- `a{lo 0}` → `Overlap_Length_mm`

### **Parser Builder (FRD-005)**
✅ Must detect semicolon delimiters  
✅ Must skip metadata rows (German text blocks)  
✅ Must handle unit rows (separate from headers)  
✅ Must parse German decimal commas: "3,23" → 3.23  
✅ PDF parsing needed for Zwick reports

### **Ontology Management (FRD-007)**
✅ Already references ontology URIs in existing schemas  
✅ Property class format: `http://ontobond.henkelgroup.io/ontology/entity#property_name`  
✅ Synonym relationships needed:
- "Zugscherfestigkeit" ← synonym_of → "Tensile_Shear_Strength_MPa"
- "Fmax" ← synonym_of → "Maximum_Force_N"

---

## Recommended FRD Updates

1. **Add real column name examples** to all FRDs
2. **Update AI prompts** with actual German/English pairs
3. **Add parser complexity examples** (cryptic notation, multi-line headers)
4. **Include `$henkel.property-class` in generated schemas**
5. **Add decimal comma handling** to parser requirements
6. **Reference real test standards**: DIN EN 1465, ISO 4587

---

**Next Steps:**
Update FRDs 001, 002, 004, 005 with concrete examples from these sample files.
