# FHIR Standards

## Standard Terminologies Used

### LOINC Codes

| Code | Display Name | Usage |
| :--- | :--- | :--- |
| **69548-6** | Genetic variant assessment | Observation resource code |
| **81247-9** | Master HL7 genetic variant reporting panel | DiagnosticReport resource code |
| **48018-6** | Gene studied [ID] | Affected gene. |
| **51961-1** | Genetic variation's effect on drug efficacy | Component code linking gene to drug class. |
| **612-2** | Bacterial strain [Type] in Isolate by Bacteria subtyping | MLST Sequence Type (ST) |
| **104274-6** | Abnormal clonal sequences [Nucleotide sequence] | Clonal Complex |
| **52107-0** | Bacterial capsule [Identifier] in Specimen | Capsule Type |
| **SP000682** | Core genome MLST [Type] in Isolate by Sequencing | cgMLST |


### SNOMED CT Codes

**Antimicrobial Classes & Substances**

| Code | Display Name |
| :--- | :--- |
| **765422000** | Product containing beta-lactam (product) |
| **350134005** | Product containing carbapenem (product) |
| **324116004** | Product containing aminoglycoside (product) |
| **1010205001** | Medicinal product containing fluoroquinolone and acting as antibacterial agent |
| **763878009** | Product containing macrolide (product) |
| **66261008** | Product containing tetracycline (medicinal product) |
| **763875007** | Product containing sulfonamide (product) |
| **32792001** | Product containing trimethoprim (medicinal product) |
| **57191001** | Product containing chloramphenicol (medicinal product) |
| **73074003** | Product containing colistin (medicinal product) |

**Clinical Findings & Organisms**
| Code | Display Name | Usage |
| :--- | :--- | :--- |
| **1098201000112108** | Carbapenemase-producing *Klebsiella pneumoniae* | Diagnostic conclusion (CRE) |
| **409801009** | Extended spectrum beta-lactamase producing *Klebsiella pneumoniae* | Diagnostic conclusion (ESBL) |
| **714315002** | Multidrug-resistant *Klebsiella pneumoniae* | Diagnostic conclusion (MDR) |
| **SP000681** | Extensively drug resistant klebsiella | Diagnostic conclusion (XDR) |

**External Naming Systems**
| System URI | Description |
| :--- | :--- |
| `http://pubmlst.org/klebsiella` | MLST Sequence Types. |
| `http://kaptive.holtlab.net/capsule` | Capsule Typing. |

