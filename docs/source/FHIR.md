# FHIR Standards

## Standard Terminologies Used

### LOINC Codes
| Code | Display Name | Usage |
| :--- | :--- | :--- |
| **69548-6** | Genetic variant assessment | Observation resource code for resistance genes and point mutations |
| **81247-9** | Master HL7 genetic variant reporting panel | DiagnosticReport resource code |
| **48018-6** | Gene studied [ID] | Component: gene symbol (e.g. *blaCTX-M-15*, GyrA) |
| **48005-3** | Amino acid change (pHGVS) | Component: pHGVS notation for point mutations (e.g. `p.(Ser83Leu)`) |
| **51961-1** | Genetic variation's effect on drug efficacy | Component code linking gene to drug class |
| **612-2** | Bacterial strain [Type] in Isolate by Bacteria subtyping | MLST Sequence Type & O-antigen Observation |
| **29576-6** | Bacterial susceptibility panel | Predicted susceptibility panel Observation |
| **SP000678** | Klebsiella pneumoniae capsular type [Identifier] by genomic analysis | Capsule type (K-type) Observation |
| **SP000680** | Klebsiella pneumoniae virulence score [Numeric] by genomic analysis | Virulence score Observation (`valueQuantity`) |
| **SP000682** | Core genome MLST [Type] in Isolate by Sequencing | cgMLST profile Observation |
| **SP000690** | Loci called | cgMLST component: number of called loci |
| **SP000691** | Total loci in scheme | cgMLST component: total loci in scheme |
| **SP000692** | Call rate | cgMLST component: percentage of loci called |

**Susceptibility Interpretation Codes (LOINC)**
| Code | Display Name | Usage |
| :--- | :--- | :--- |
| **LA6676-6** | Resistant | `valueCodeableConcept` in susceptibility panel component |
| **LA24225-7** | Susceptible | `valueCodeableConcept` in susceptibility panel component |
| **LA9633-4** | Present | `valueCodeableConcept` for detected resistance variant Observation |
| **LA9634-2** | Absent | `valueCodeableConcept` for susceptible (no resistance) Observation |

**Susceptibility Panel Drug LOINC Codes**
| Code | Antimicrobial | Usage |
| :--- | :--- | :--- |
| **18864-9** | Ampicillin | Susceptibility panel component |
| **18861-5** | Amoxicillin-Clavulanate | Susceptibility panel component |
| **18970-4** | Piperacillin-Tazobactam | Susceptibility panel component |
| **18876-3** | Cefotaxime | Susceptibility panel component |
| **18879-7** | Ceftazidime / Cefepime | Susceptibility panel component |
| **18943-1** | Meropenem | Susceptibility panel component |
| **18906-8** | Ciprofloxacin | Susceptibility panel component |
| **18928-2** | Gentamicin | Susceptibility panel component |
| **18860-7** | Amikacin | Susceptibility panel component |
| **18998-5** | Trimethoprim-Sulfamethoxazole | Susceptibility panel component |
| **18912-6** | Colistin | Susceptibility panel component |

### SNOMED CT Codes

**Antimicrobial Classes & Substances**
| Code | Display Name | Alias |
| :--- | :--- | :--- |
| **765422000** | Product containing beta-lactam (product) | `beta_lactam`, `bla` |
| **350134005** | Product containing carbapenem (product) | `carbapenem` |
| **324116004** | Product containing aminoglycoside (product) | `aminoglycoside`, `agly` |
| **1010205001** | Medicinal product containing fluoroquinolone and acting as antibacterial agent | `fluoroquinolone`, `flq` |
| **763878009** | Product containing macrolide (product) | `macrolide`, `mls` |
| **66261008** | Product containing tetracycline (medicinal product) | `tetracycline`, `tet` |
| **763875007** | Product containing sulfonamide (product) | `sulfonamide`, `sul` |
| **32792001** | Product containing trimethoprim (medicinal product) | `trimethoprim`, `tmt`, `dfr` |
| **57191001** | Product containing chloramphenicol (medicinal product) | `chloramphenicol`, `phenicol` |
| **73074003** | Product containing colistin (medicinal product) | `colistin`, `col` |
| **387065000** | Product containing fosfomycin (medicinal product) | `fosfomycin`, `fos` |
| **77891000** | Product containing rifampicin (medicinal product) | `rifampicin`, `rif` |

**Clinical Findings**
| Code | Display Name | Usage |
| :--- | :--- | :--- |
| **1098201000112108** | Carbapenemase-producing *Klebsiella pneumoniae* (organism) | DiagnosticReport `conclusionCode` (CRE) |
| **409801009** | Extended spectrum beta-lactamase producing *Klebsiella pneumoniae* (organism) | DiagnosticReport `conclusionCode` (ESBL) |
| **1098101000112102** | AmpC beta-lactamase producing *Klebsiella pneumoniae* (organism) | DiagnosticReport `conclusionCode` (AmpC) |
| **714315002** | Multidrug-resistant *Klebsiella pneumoniae* (organism) | DiagnosticReport `conclusionCode` (MDR) |
| **SP000681** | Extensively drug resistant Klebsiella pneumoniae | DiagnosticReport `conclusionCode` (XDR) |
| **SP000683** | Multidrug-resistant AmpC-producing Klebsiella pneumoniae | DiagnosticReport `conclusionCode` (MDR AmpC) |
| **SP000687** | Multidrug-resistant ESBL-producing Klebsiella pneumoniae | DiagnosticReport `conclusionCode` (MDR ESBL) |
| **KP-SO** | Klebsiella pneumoniae Sensitif Obat | DiagnosticReport `conclusionCode` (Susceptible) |
| **KP-RL** | Klebsiella pneumoniae Resisten Lain | DiagnosticReport `conclusionCode` (Resistant, single/low-class) |

### HL7 Terminology Systems
| System URI | Usage |
| :--- | :--- |
| `http://terminology.hl7.org/CodeSystem/observation-category` | `category` on all Observation resources (`laboratory`) |
| `http://terminology.hl7.org/CodeSystem/v2-0074` | Observation category (`GE` = Genetics) |
| `http://terminology.hl7.org/CodeSystem/v3-ObservationInterpretation` | Susceptibility panel `interpretation` (`N` = Normal, `A` = Abnormal) |
