# Overview
[This pipeline](https://github.com/oucru-id/kp-to-fhir-full) is a Nextflow-based workflow designed for the analysis of Klebsiella pneumoniae genomic data. It processes raw sequencing data (Illumina or Nanopore) to identify antimicrobial resistance (AMR) genes (kleborate), capsule types (kaptive), MLST typing (kleborate), virulence factors (kleborate), cgMLST schema (Ridom), and generates a FHIR-compliant genomics bundle. 

## Key Features
* **Multi-platform Support**: Processes raw read data from diverse platforms.
* **Comprehensive Typing**: MLST, capsule typing (K/O locus), cgMLST, virulence score.
* **Antimicrobial Resistance Detection**: Resistance genes, including ESBL, carbapenemases, and other acquired resistance.
* **Virulence Factor Assessment**:Yersiniabactin, colibactin, aerobactin, salmochelin, and hypermucoidy markers.
* **Clinical Integration**: Merges genomic data with clinical metadata.
* **Quality Control**: QC reporting with MultiQC.

## Key Outputs
* Genome assembly
* Typing and resistance characterization
* FHIR-compliant genomic reports 
* Quality control metrics
* cgMLST schema

## Directory Structure

```
kp-to-fhir
├── main.nf                             # Main workflow
├── nextflow.config                     # Configuration and parameters
├── workflows/
│   ├── illumina.nf                     # Illumina sub-workflow
│   ├── nanopore.nf                     # Nanopore sub-workflow
│   ├── typing.nf                       # Kleborate typing sub-workflow
│   ├── cgmlst.nf                       # cgMLST sub-workflow
│   ├── fhir.nf                         # FHIR bundle generation
│   ├── validate_fhir.nf                # FHIR validation
│   ├── merge_clinical_data.nf          # Clinical metadata merge
│   ├── upload_fhir.nf                  # FHIR server upload
│   ├── report.nf                       # QC and sample report generation
│   └── utils.nf                        # Utility functions
├── scripts/
│   ├── annotated_to_fhir.py            # Kleborate JSON-to-FHIR converter
│   ├── clinical_metadata_parser.py     # clinical metadata parser
│   ├── generate_sample_report.py       # Per-sample text report
│   ├── merge_clinical_fhir.py          # FHIR genomics + clinical data merger
│   ├── upload_fhir.py                  # FHIR uploader (OAuth 2.0)
│   ├── get_access_token.py             # Standalone token fetcher
│   └── get_versions.py                 # Software version collector
├── data/
│   ├── NGS/                            # Input FASTQ files
│   ├── cgmlst_schema/                  # chewBBACA cgMLST schema
│   ├── patient_clinical_metadata.csv   # Patient metadata
│   ├── organization_metadata.csv       # Organization metadata
│   └── practitioner_metadata.csv       # Practitioner metadata
└── tools/
    └── fhir-validator.jar              # HL7 FHIR validator
```
