# Output Files
The primary output is the **HL7 FHIR** bundle and per-sample genomic reports.

## FHIR Genomic Bundle
### Observation Resources
Each detected resistance gene generates an Observation resource containing:
*   **Resistance Genes**: 
    *   **Gene ID**: Standard gene symbol (e.g., *blaCTX-M-15*).
    *   **Drug Class**: SNOMED CT codes for affected drug classes.
    *   **Context**: Distinguishes between acquired resistance genes and intrinsic chromosomal variants (e.g., *blaSHV-1*).
*   **Strain Typing**:
    *   **MLST**: Sequence Type (ST) (e.g., ST258).
    *   **cgMLST**: core genome MLST assignment (e.g., KP1_RS06235).
*   **Capsule & Serotyping**:
    *   **K-Type**: Capsule type (e.g., K64).
*   **Virulence**:
    *   **Virulence Score**: Quantitative score (0-5) based on the presence of key virulence factors.

## Clinical Data Integration
Merges the FHIR Genomics Observations with patient, facility, and practitioner information to create a complete genomic diagnostic report document.
### DiagnosticReport Resource
*   **Conclusion**: A human-readable summary including Resistance Classification, MLST, and Virulence Score.
*   **Presentation**: Contains a Base64 encoded HTML representation of the report.
*   **Links**: References the Patient, Specimen, and all Observations.

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
├── cgmlst/
│   └── results_alleles.tsv
│   └── cgmlst_statistics.txt          
├── fhir_merged/
│   └── *.merged.fhir.json      
├── fhir_validated/
│   ├── *.validation.txt  
├── reports/
│   └── *.summary_report.txt  
```
