# Data Processing

## Main Workflow Controller
**File:** `main.nf`

The main workflow handles channel processing and parallel execution.

*   **Input Detection**: Automatically detects input data types (Illumina or Nanopore) and routes them to the dedicated sub-workflows.
*   **Parallel Processing**: Input streams are processed concurrently.
*   **Channel Merging**: Assemblies from different streams are routed to unified channels for downstream processes:
    *   Typing (MLST, Serotyping, Virulence, Resistance)
    *   FHIR resource generation
    *   Clinical data integration
    *   MultiQC aggregate reporting

## Nanopore (Long-Read) Workflow
**File:** `nanopore.nf`

For Oxford Nanopore Technologies (ONT) sequencing data.

1.  **Quality Control**: 
    *   Tool: `FastQC`
    *   Metrics: Per-sample quality, GC content, per-base sequence quality.
2.  **Assembly and Polishing**:
    *   Tool: `Minimap2` and `Miniasm`
    *   Process: Generates a de novo assembly from long reads.
    *   *Note*: Workflow contains provisions for `Flye` assembly and `Medaka` polishing as alternative configurations.

## Illumina (Short-Read) Workflow
**File:** `illumina.nf`

For Illumina sequencing data.

1.  **Trimming & QC**:
    *   Tool: `fastp`
    *   Actions: Adapter removal, quality trimming.
    *   Tool: `FastQC`
    *   Actions: Per-sample quality, GC content, per-base sequence quality.
2.  **De Novo Assembly**:
    *   Tool: `Megahit` or `SPAdes`
    *   Process: Generates a de novo assembly from short reads.
3.  **Polishing**:
    *   Tool: `Pilon`
    *   Process: Maps reads back to the assembly using to correct base errors.

## Typing Analysis
**File:** `typing.nf`

Performs genomic characterization of the assembled contigs.

1.  **Genotyping Tool**: `Kleborate`
    *   **MLST**: Determines Sequence Type (ST) and Clonal Complex.
    *   **Serotyping**: Predicts Capsule (K) and O antigen loci using `Kaptive`.
    *   **Virulence**: Detects key virulence factors (Yersiniabactin, Colibactin, Aerobactin, Salmochelin, RmpADC, RmpA2) and calculates virulence score.
    *   **Resistance**: Identifies acquired resistance genes (ESBLs, Carbapenemases) and chromosomal mutations.

## FHIR Converter

Converts genomic typing data into HL7 FHIR R4 standard resources.

1.  **Input Parsing**: Reads structured Typing and Lineage JSON files.
2.  **Mapping**:
    *   **Resistance Genes**: Mapped to LOINC codes and SNOMED CT for drug classes.
    *   **Strain Typing**: MLST mapped to LOINC.
    *   **Capsule Typing**: K-type mapped to LOINC.
    *   **Virulence**: Virulence scores and factors mapped to observation components.
3.  **Resource Creation**:
    *   Generates `Observation` resources for resistance genes, susceptibility assessments, and strain characteristics.
    *   Embeds interpretation of "High Risk Clones" and resistance profiles (e.g., ESBL/Carbapenemase producer status).