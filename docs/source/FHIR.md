# FHIR standards

KPtoFHIR writes HL7 FHIR R4 that conforms to the
[Genomics Reporting IG 3.0.0](https://hl7.org/fhir/uv/genomics-reporting/STU3/). Two output
profiles are available, chosen with `--fhir_profile`:

- **`generic`** (default): HL7 Genomics Reporting only.
- **`satusehat`**: the output for the Indonesian Ministry of Health SATUSEHAT platform (Kemkes
  profiles, NIK identifiers, Kemkes extensions and codings).

## Profiles

Genomics Reporting profiles are under
`http://hl7.org/fhir/uv/genomics-reporting/StructureDefinition/`, and Kemkes profiles under
`https://fhir.kemkes.go.id/r4/StructureDefinition/`.

| Resource | `generic` | `satusehat` |
| --- | --- | --- |
| Resistance gene and mutation Observation | Genomics Reporting `variant` | same |
| Susceptibility panel | base `Observation` | Genomics Reporting `therapeutic-implication` |
| MLST, virulence, capsule, O antigen, cgMLST, plasmid and QC Observations | base `Observation` | base `Observation` |
| DiagnosticReport | Genomics Reporting `genomic-report` | Genomics Reporting `genomics-report` |
| Patient, Specimen, Encounter, ServiceRequest, Organization, Practitioner, PractitionerRole | base FHIR R4 | Kemkes profile of the same name |
| Bundle | base FHIR R4 | Kemkes `Bundle` |

`satusehat` also tags the Observations, DiagnosticReport, Device and Provenance with `genomics`
(`http://terminology.kemkes.go.id/sp`).

## LOINC codes

| Code | Display | Used in |
| --- | --- | --- |
| `69548-6` | Genetic variant assessment | Resistance gene and mutation Observations |
| `29576-6` | Bacterial susceptibility panel | Susceptibility panel Observation |
| `612-2` | Bacterial strain [Type] in Isolate by Bacteria subtyping | MLST and O antigen Observations |
| `51969-4` | Genetic analysis report | DiagnosticReport (`generic`) |
| `81247-9` | Master HL7 genetic variant reporting panel | DiagnosticReport (`satusehat`; second coding in `generic`) |
| `48018-6` | Gene studied [ID] | Resistance Observation component |
| `51961-1` | Genetic variation's effect on drug efficacy | Resistance Observation component (drug class) |
| `48005-3` | Amino acid change (pHGVS) | Resistance Observation component (point mutations) |

**Answer codes**

| Code | Display | Used in |
| --- | --- | --- |
| `LA9633-4` | Present | Value of a resistance gene or mutation Observation |
| `LA9634-2` | Absent | Value of the Observation for a sample with no resistance gene |
| `LA6676-6` | Resistant | Susceptibility panel component |
| `LA24225-7` | Susceptible | Susceptibility panel component |

## Susceptibility panel drug codes

The panel has one component per drug, coded with LOINC:

| Drug | LOINC |
| --- | --- |
| Ampicillin | `18864-9` |
| Amoxicillin-Clavulanate | `18861-5` |
| Piperacillin-Tazobactam | `18970-4` |
| Cefotaxime | `18876-3` |
| Ceftazidime | `18879-7` |
| Cefepime | `18879-7` |
| Meropenem | `18943-1` |
| Ciprofloxacin | `18906-8` |
| Gentamicin | `18928-2` |
| Amikacin | `18860-7` |
| Trimethoprim-Sulfamethoxazole | `18998-5` |
| Colistin | `18912-6` |

Ceftazidime and Cefepime share a code, the component text names the drug. See
[Known limitations](limitations.md#susceptibility-panel-codes).

## Drug classes

The *effect on drug efficacy* component codes the drug class of Kleborate's result column with
SNOMED CT:

| Code | Display | Kleborate columns |
| --- | --- | --- |
| `765422000` | Product containing beta-lactam (product) | all `Bla` columns, including carbapenemases and ESBLs |
| `350134005` | Product containing carbapenem (product) | *OmpK35* / *OmpK36* mutations |
| `324116004` | Product containing aminoglycoside (product) | `AGly` |
| `1010205001` | Medicinal product containing fluoroquinolone and acting as antibacterial agent | `Flq`, and *GyrA* / *ParC* mutations |
| `763878009` | Product containing macrolide (product) | `MLS` |
| `66261008` | Product containing tetracycline (medicinal product) | `Tet` |
| `763875007` | Product containing sulfonamide (product) | `Sul` |
| `32792001` | Product containing trimethoprim (medicinal product) | `Tmt` |
| `73074003` | Product containing colistin (medicinal product) | `Col`, and colistin resistance mutations |
| `77891000` | Product containing rifampicin (medicinal product) | `Rif` (*arr* genes) |

Genes from the `Phe`, `Fcyn`, `Gly` and `Tgc` columns have no drug-class component (see
[Known limitations](limitations.md#drug-classes-without-a-code)).

## Conclusion codes

`DiagnosticReport.conclusionCode` carries the resistance classification (see
[Outputs](outputs.md#resistance-classification)):

| Classification | `generic` | `satusehat` |
| --- | --- | --- |
| **CRE** | SNOMED CT `1098201000112108` Carbapenemase-producing *Klebsiella pneumoniae* | same |
| **ESBL** | SNOMED CT `409801009` Extended spectrum beta-lactamase producing *Klebsiella pneumoniae* | same |
| **AmpC** | SNOMED CT `1098101000112102` AmpC beta-lactamase producing *Klebsiella pneumoniae* | same |
| **MDR** | SNOMED CT `714315002` Multidrug-resistant *Klebsiella pneumoniae* | same |
| **XDR** | text | `SP000681` |
| **Multidrug-Resistant ESBL** | text | `SP000687` |
| **Multidrug-Resistant AmpC** | text | `SP000683` |
| **Other Resistant** | text | `KP-RL` Klebsiella pneumoniae Resisten Lain |
| **Susceptible** | text | `KP-SO` Klebsiella pneumoniae Sensitif Obat |

The `satusehat` codes are in `http://terminology.kemkes.go.id/CodeSystem/clinical-term`. They have
no SNOMED CT equivalent, so `generic` reports those classes as text only.

## Typing codes by profile

| Element | `generic` | `satusehat` |
| --- | --- | --- |
| ST (MLST value) | `klebsiella-mlst-st`, e.g. `11` | `http://kaptive.holtlab.net/mlst` |
| K type (capsule value) | `kaptive-k-locus`, e.g. `K64` | `http://kaptive.holtlab.net/capsule` |
| O type (O antigen value) | `kaptive-o-locus` | `http://kaptive.holtlab.net/o-antigen` |
| Capsule Observation code | `kpfhir-observation-code` `capsule-type` | `SP000678` |
| Virulence score Observation code | `kpfhir-observation-code` `virulence-score` (unit `{score}`) | `SP000680` |
| cgMLST Observation code | `kpfhir-observation-code` `cgmlst-profile` | `SP000682` |
| cgMLST loci called, loci in scheme, call rate | `cgmlst-loci-called`, `cgmlst-loci-total`, `cgmlst-call-rate` | `SP000690`, `SP000691`, `SP000692` |
| cgMLST loci | locus name under `https://www.cgmlst.org/ncs` | same |
| Gene studied | text | coding under `http://www.genenames.org/geneId` |

The `SP` codes are in `http://terminology.kemkes.go.id/sp`.

## Other code systems

| System | Codes | Used in |
| --- | --- | --- |
| SNOMED CT | `444018003` No antimicrobial resistance detected | Observation for a sample with no resistance gene |
| SNOMED CT | `706689003` Application program software | Device |
| Sequence Ontology (`http://www.sequenceontology.org`) | `SO:0000155` plasmid, `SO:0000340` chromosome | Genomic location of a resistance gene |
| HGVS (`http://varnomen.hgvs.org`) | pHGVS, e.g. `p.(Ser83Ile)` | Amino acid change |
| UCUM | `{score}`, `{count}`, `%` | Virulence score, cgMLST counts and call rate |
| `observation-category` | `laboratory` | All Observations |
| `v2-0074` | `GE` Genetics | Observations and DiagnosticReport |
| `v3-ObservationInterpretation` | `N` Normal, `A` Abnormal | Susceptibility panel, sequencing QC |
| `data-absent-reason` | `error` | Plasmid summary when MOB-suite failed |
| `provenance-participant-type` | `assembler`, `performer` | Provenance agents |

## Project CodeSystems

Concepts that no standard code covers use CodeSystems published by the project under
`http://terminology.spheres.id/CodeSystem/`.

| CodeSystem | Codes |
| --- | --- |
| `kp-plasmid` | `plasmid-analysis` Plasmid analysis, `plasmid` Plasmid identified, `genomic-location` Genomic location |
| `kpfhir-observation-code` | `sequencing-qc`, `capsule-type`, `virulence-score`, `cgmlst-profile`, `cgmlst-loci-called`, `cgmlst-loci-total`, `cgmlst-call-rate` |
| `pipeline-property` | `source-repository` (Device property) |
| `klebsiella-mlst-st` | *K. pneumoniae* sequence type numbers (not enumerated) |
| `kaptive-k-locus` | Kaptive K loci and K types (not enumerated) |
| `kaptive-o-locus` | Kaptive O loci and O types (not enumerated) |
