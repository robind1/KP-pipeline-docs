# Output Files

The pipeline generates a structured set of results organized by data type. The primary output is the **HL7 FHIR** bundle, which integrates genomic typing, virulence, and resistance findings with clinical metadata.

## FHIR Genomic Observations

Converts annotated genomic features into standardized FHIR **Observation** resources.

### Observation Resources
Each detected feature generates an Observation resource containing:
*   **Resistance Genes**: 
    *   **Gene ID**: Standard gene symbol (e.g., *blaCTX-M-15*).
    *   **Drug Class**: SNOMED CT codes for affected drug classes (e.g., Beta-lactams, Aminoglycosides, Fluoroquinolones).
    *   **Context**: Distinguishes between acquired resistance genes and intrinsic chromosomal variants (e.g., *blaSHV-1*).
*   **Strain Typing**:
    *   **MLST**: Sequence Type (ST) (e.g., ST258).
    *   **Clonal Complex**: High-level grouping (e.g., CC258).
*   **Capsule & Serotyping**:
    *   **K-Type**: Capsule type (e.g., K64).
    *   **wzi Allele**: Specific allele of the *wzi* capsule.
*   **Virulence**:
    *   **Virulence Score**: Quantitative score (0-5) based on the presence of key virulence factors (Yersiniabactin, Colibactin, Aerobactin, Salmochelin, RmpADC, RmpA2).

## Clinical Data Integration

Merges the FHIR Genomics Observations with patient, facility, and practitioner information to create a complete genomic diagnostic report document.

### DiagnosticReport Resource
*   **Conclusion**: A human-readable summary including Resistance Classification, MLST, and Virulence Score.
*   **Presentation**: Contains a Base64 encoded HTML representation of the report.
*   **Links**: References the Patient, Specimen, and all Variant Observations.

## Drug Resistance Classification

| Classification | Definition | Logic |
| :--- | :--- | :--- |
| **Susceptible** | No significant resistance | No acquired resistance genes detected |
| **ESBL** | Extended-Spectrum Beta-Lactamase | Presence of *blaCTX-M*, *blaSHV*, or *blaTEM* variants with ESBL activity |
| **MDR** | Multidrug-Resistant | Resistance genes detected for ≥3 antimicrobial classes |
| **CRE** | Carbapenem-Resistant | Presence of carbapenemase genes (e.g., *blaKPC*, *blaNDM*, *blaVIM*, *blaIMP*, *blaOXA-48*) |
| **XDR** | Extensively Drug-Resistant | CRE + Resistance to Colistin |

## Output Directory Structure

```text
results/
├── qc/
│   └── multiqc_report.html       
├── typing/
│   └── *.typing.json             
├── lineage/
│   └── *.lineage.json           
├── fhir/
│   └── *.fhir.json               
├── fhir_merged/
│   └── *.merged.fhir.json      
├── fhir_validated/
│   ├── *.validation.txt  
├── reports/
│   └── *.summary_report.txt  
```