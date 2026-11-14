# Outputs

## Output directory

```text
results/
├── pipeline_info/          execution report, timeline, trace, software_versions.yml
├── assembly/               <id>_contigs.fasta (Illumina), <id>_assembly.fasta (ONT, FASTA input)
├── qc/fastqc/              FastQC reports (Illumina)
├── qc/fastp/               <id>_fastp.json, <id>_fastp.html (Illumina)
├── qc/nanoplot/            <id>_NanoStats.txt, <id>_NanoPlot-report.html, read coverage (ONT)
├── qc/assembly/            <id>_assembly_qc.json: coverage status and warnings (ONT)
├── qc/multiqc/             multiqc_report.html
├── typing/                 <id>_kleborate_full.txt, <id>_typing.json, <id>_lineage.json
├── plasmid/                <id>/ (MOB-suite reports, amr_hits.paf), <id>_plasmid.json, plasmid_sharing.tsv
├── cgmlst/                 results_alleles.tsv, cgmlst_statistics.txt
├── reports/                <id>.summary_report.txt
└── fhir/
    ├── genomic/            <id>.genomic.fhir.json                      (no patient data)
    ├── clinical/           clinical resources, clinical_summary.tsv    (personal data)
    ├── bundle/             <id>.bundle.fhir.json, link_summary.tsv     (personal data)
    ├── validation/         validation_summary.tsv, validation_<n>.json
    └── upload/             <id>.bundle.upload.json
```

`plasmid/` is written unless `--plasmid_analysis false`, and `cgmlst/` only with
`--cgmlst_schema`.

## FHIR output layers

| Step | Output | Contains |
| --- | --- | --- |
| genomic (always) | `fhir/genomic/<id>.genomic.fhir.json` | DiagnosticReport; Observations for resistance genes and mutations, the susceptibility panel, MLST, virulence score, capsule, O antigen, cgMLST, plasmids and sequencing QC; Device; Provenance. **No patient data**: the subject is the reference `Patient/<id>-patient`. |
| clinical (with `--patient_metadata`) | `fhir/clinical/` | Patient, Specimen, Encounter and ServiceRequest per sample; Organization, Practitioner and PractitionerRole once. **Personal data.** |
| link (with `--patient_metadata`) | `fhir/bundle/<id>.bundle.fhir.json` | One FHIR `transaction` per sample, with every reference resolved inside the bundle. **Personal data.** |

The genomic bundle is a FHIR `collection`. Resource ids and `fullUrl`s are derived from the sample
id, so they are the same in every run. Timestamps are the run time, or a fixed time when the
environment variable `KPFHIR_FIXED_TIMESTAMP` or `SOURCE_DATE_EPOCH` is set. The codes in each
resource are listed on [FHIR standards](FHIR.md).

## Genomic bundle

### Resistance gene and mutation Observations

One per resistance gene or point mutation reported by Kleborate, LOINC `69548-6` (Genetic variant
assessment) with the value `LA9633-4` Present, and the Genomics Reporting `variant` profile.
Components:
- **Gene studied** (LOINC `48018-6`): the gene, e.g. *KPC-2*, or the gene followed by the
  mutation as Kleborate reports it. In `generic` the gene is text only (bacterial genes have no
  HGNC code).
- **Effect on drug efficacy** (LOINC `51961-1`): the drug class, coded with SNOMED CT (see
  [FHIR standards](FHIR.md#drug-classes)). Point mutations get the class of their gene: *GyrA* and
  *ParC* fluoroquinolone, *OmpK35* and *OmpK36* carbapenem, *MgrB*, *PhoP*, *PhoQ*, *PmrA*, *PmrB*,
  *CrrA* and *CrrB* colistin.
- **Amino acid change** (LOINC `48005-3`) for point mutations, in pHGVS, e.g. `p.(Ser83Ile)`.
  Filled only when Kleborate's notation is a substitution.
- **Genomic location** (with the plasmid step): `plasmid` or `chromosome` (Sequence Ontology), or
  `Unresolved`, and the plasmid cluster and replicons for plasmid-borne genes.
- **Virulence factors**: the sample's virulence loci, repeated on each Observation as in 1.4.

The narrative names the gene and its drug class, the sample's ST and capsule, and where the gene
lies (the plasmid cluster and replicons, the chromosome, or unresolved). Intrinsic chromosomal
genes (Kleborate's `Bla_chr`, e.g. *SHV-11*) are marked as such.

A sample with no resistance gene or mutation gets one Observation instead, LOINC `69548-6` with
the values `LA9634-2` Absent and SNOMED CT `444018003` No antimicrobial resistance detected.

### Predicted susceptibility panel

One Observation, LOINC `29576-6` (Bacterial susceptibility panel), with a component per drug
(see [FHIR standards](FHIR.md#susceptibility-panel-drug-codes)). Each drug is **Resistant**
(LOINC `LA6676-6`) or **Susceptible** (LOINC `LA24225-7`). The panel's `interpretation` is `N`
(no acquired resistance) or `A` (acquired resistance to at least one drug).

| Drug | Predicted resistant when |
| --- | --- |
| Ampicillin | always (intrinsic resistance of *K. pneumoniae*) |
| Amoxicillin-clavulanate | a carbapenemase gene |
| Piperacillin-tazobactam | a carbapenemase or ESBL gene |
| Cefotaxime, Ceftazidime, Cefepime | a carbapenemase or ESBL gene |
| Meropenem | a carbapenemase gene, or an ESBL gene together with mutations in both *OmpK35* and *OmpK36* |
| Ciprofloxacin | a fluoroquinolone resistance gene (e.g. *qnr*, *aac(6')-Ib-cr*) or a *GyrA* / *ParC* mutation |
| Gentamicin | an aminoglycoside gene active against gentamicin: *aac(3)* genes, *aac(6')-II*, *ant(2'')*, and the 16S rRNA methylases *armA*, *rmtA*–*rmtH*, *npmA*. Any other unknown *aac* / *ant* / *aph* gene also counts, as a precaution |
| Amikacin | an aminoglycoside gene active against amikacin: *aac(6')-I* genes including *aac(6')-Ib-cr*, *aac(6')-II*, *aac(3)-IV*, *ant(4')*, *aph(3')-VI*, and the 16S rRNA methylases |
| Trimethoprim-sulfamethoxazole | a *dfr* (trimethoprim) or *sul* (sulfonamide) gene |
| Colistin | an *mcr* or other colistin resistance gene, or a mutation in *MgrB*, *PhoP*, *PhoQ*, *PmrA*, *PmrB*, *CrrA* or *CrrB* |

AmpC genes do not change the panel; they count towards the [classification](#resistance-classification).

### Typing Observations

| Observation | Code | Value |
| --- | --- | --- |
| **MLST** | LOINC `612-2` (Bacterial strain [Type] in Isolate by Bacteria subtyping) | the sequence type, e.g. `ST11`, with a component per MLST allele. Without an exact ST: *Novel ST (new allele combination)* or *ST not determined (inexact or missing alleles)*, as text |
| **Virulence score** | `virulence-score` (`generic`), `SP000680` (`satusehat`) | Kleborate's virulence score, 0–5 |
| **Capsule** | `capsule-type` (`generic`), `SP000678` (`satusehat`) | the K type, e.g. `K64`; for a capsule-null or untyped locus, the K locus, e.g. `KL64 (capsule null)` |
| **O antigen** | LOINC `612-2` | the O type, e.g. `O1/O2v1` |

### cgMLST Observation

With `--cgmlst_schema`. Code `cgmlst-profile` (`generic`) or `SP000682` (`satusehat`).
Components:
- loci called, loci in the scheme and the call rate (%);
- one component per locus, coded with the locus name under `https://www.cgmlst.org/ncs`. The
  allele number is a `valueInteger`; chewBBACA's special classes (e.g. `LNF`, `NIPH`) are a
  `valueString`.

### Plasmid Observations

With the plasmid step:
- **Plasmid analysis summary** (`kp-plasmid#plasmid-analysis`): the number of plasmids found, the
  analysis status, the assembly type and confidence, the MOB-suite version and the resistance-gene
  reference. If MOB-suite failed, the value is absent with the reason `error`.
- **One Observation per plasmid** (`kp-plasmid#plasmid`): the replicons as the value, and
  components for the cluster, size, number of contigs, relaxase, MPF and oriT types, predicted
  mobility, nearest reference plasmid (with the Mash distance), predicted host range, and the
  resistance, carbapenemase and ESBL genes it carries. `hasMember` points to the resistance
  Observations of the genes on it.

The confidence is `long-read` for ONT, `predicted (short-read)` for Illumina and
`predicted (submitted assembly)` for FASTA input. Short-read assemblies break plasmids at repeats,
so read Illumina results as predictions.

### Sequencing QC Observation

For ONT samples, code `sequencing-qc`. The value is the coverage status (`PASS`, `LOW` or
`UNKNOWN`). Components: platform, estimated and minimum read coverage, polishing result, and each
quality warning. With a warning, the interpretation is `A` (abnormal), and the DiagnosticReport
conclusion says to interpret the results with caution.

### DiagnosticReport

- **Profile and code**:
  - `generic`: `genomic-report`, LOINC `51969-4` (Genetic analysis report), with `81247-9` as a
    second coding
  - `satusehat`: `genomics-report`, LOINC `81247-9`
- **Conclusion**: the [resistance classification](#resistance-classification) with the genes behind
  it and the drugs predicted resistant, then the ST, capsule, virulence score, plasmid-borne
  carbapenemase or ESBL genes, and any quality warning.
- **conclusionCode**: the classification (coded where a code exists, see
  [FHIR standards](FHIR.md#conclusion-codes)), plus text entries for the ST, O antigen, capsule,
  plasmid-borne carbapenemase or ESBL, and a sequencing quality warning.
- **presentedForm**: an HTML report, base64-encoded.
- **subject** `Patient/<id>-patient` and **specimen** `Specimen/<id>-specimen`, by reference only.
- **performer**: the Organization given by `--performer_org_id`, if any.
- **identifier**: in `generic` only with `--identifier_system_base`; in `satusehat` under
  `http://sys-ids.kemkes.go.id`.

### Device and Provenance

- **Device**: KPtoFHIR, its version and source repository, SNOMED CT `706689003` (Application
  program software).
- **Provenance**: one per sample, targeting the DiagnosticReport and every Observation it rests
  on, with the software as `assembler` and the performing Organization as `performer`.

## Clinical resources and linked bundle

With `--patient_metadata`, the clinical step adds Patient, Specimen, Encounter and
ServiceRequest per sample, and Organization, Practitioner and PractitionerRole. With
`--fhir_profile satusehat` they carry the Kemkes profiles, NIK identifiers and Kemkes extensions.

The link step writes one FHIR `transaction` per sample in `fhir/bundle/`, joining the genomic
report to the ServiceRequest, Encounter, Organization and Practitioner. See
[Clinical metadata](clinical_metadata.md#what-the-link-step-adds).

## Resistance classification

The DiagnosticReport conclusion is the first class that applies, in this order. Drug classes are
counted over the resistance genes and mutations, for example beta-lactam, carbapenem,
aminoglycoside, fluoroquinolone, sulfonamide and trimethoprim.

| Classification | Criteria |
| --- | --- |
| **Extensively Drug-Resistant (XDR)** | A carbapenemase gene, plus colistin resistance (an *mcr* or other colistin gene, or a colistin resistance mutation) |
| **Carbapenem-Resistant Enterobacteriaceae (CRE)** | A carbapenemase gene: *KPC*, *NDM*, *OXA-48*, *OXA-181*, *OXA-232*, *VIM*, *IMP* or *GES* |
| **Multidrug-Resistant ESBL** | An ESBL gene, and resistance in 3 or more drug classes |
| **Extended-Spectrum Beta-Lactamase (ESBL)** | An ESBL gene: *CTX-M*, or an ESBL variant of *SHV* or *TEM* |
| **Multidrug-Resistant AmpC** | A plasmid AmpC gene, and resistance in 3 or more drug classes |
| **AmpC Beta-Lactamase (AmpC)** | A plasmid AmpC gene: *DHA*, *CMY*, *MOX*, *FOX*, *ACC*, *ACT*, *MIR*, *LAT*, *BIL* or *CFE* |
| **Multidrug-Resistant (MDR)** | Resistance in 3 or more drug classes, without the genes above |
| **Other Resistant** | Any other acquired resistance gene |
| **Susceptible** | No acquired resistance gene: intrinsic ampicillin resistance only |

```{note}
Narrow-spectrum *SHV* variants (e.g. *SHV-1*, *SHV-11*) are intrinsic to *K. pneumoniae* and are
left out of the classification. Other intrinsic chromosomal genes count only when they are a
carbapenemase, ESBL, AmpC or colistin resistance gene.
```

## Other per-sample files

| File | Content |
| --- | --- |
| `typing/<id>_kleborate_full.txt` | Kleborate's full results table |
| `typing/<id>_typing.json` | parsed typing: MLST, capsule, O locus, virulence, resistance genes and mutations |
| `plasmid/<id>_plasmid.json` | plasmids, gene locations and confidence |
| `plasmid/<id>/` | MOB-suite reports (`contig_report.txt`, `mobtyper_results.txt`, `recon_status.txt`, `mob_recon.log`) and the CARD hits (`amr_hits.paf`) |
| `reports/<id>.summary_report.txt` | plain-text summary of the sample |
