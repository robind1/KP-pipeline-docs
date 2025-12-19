# Data Processing

## Main Workflow Controller
**File:** `main.nf`
The main workflow handles channel processing and parallel execution. It automatically detects input data types (Illumina or Nanopore) and routes them to the dedicated sub-workflows. All inputs are processed concurrently.

## Nanopore (Long-Read) Workflow
**File:** `nanopore.nf`
For Oxford Nanopore Technologies (ONT) sequencing data.
1.  **Trimming**:
    *   Tool: `chopper`
    *   Actions: Filters reads based on average quality and minimum length.
2.  **Quality Control**: 
    *   Tool: `FastQC`
    *   Metrics: Per-sample quality, GC content, per-base sequence quality.
3.  **Assembly**:
    *   Tool: `Flye`
    *   Process: Generates a de novo assembly from long reads.
4.  **Polishing**:
    *   Tool: `medaka`
    *   Process: Maps reads back to the assembly to correct base errors.

## Illumina (Short-Read) Workflow
**File:** `illumina.nf`
For Illumina paired-end sequencing data.
1.  **Trimming**:
    *   Tool: `fastp`
    *   Actions: Adapter removal, quality trimming.
2.  **QC**:
    *   Tool: `FastQC`
    *   Actions: Per-sample quality, GC content, per-base sequence quality.
3.  **De Novo Assembly**:
    *   Tool: `Megahit` or `SPAdes`
    *   Process: Generates a de novo assembly from short reads.
4.  **Polishing**:
    *   Tool: `Pilon`
    *   Process: Maps reads back to the assembly to correct base errors.

## Typing Analysis
**File:** `typing.nf`
Performs characterization of the assembled contigs.
1.  **Genotyping Tool**: `Kleborate`
    *   **MLST**: Determines Sequence Type (ST) and Clonal Complex.
    *   **Serotyping**: Predicts Capsule (K) and O antigen loci using `Kaptive`.
    *   **Virulence**: Detects key virulence factors and calculates virulence score.
    *   **Resistance**: Identifies acquired resistance genes (ESBLs, Carbapenemases) and chromosomal mutations.

## cgMLST Analysis
**File:** `cgmlst.nf`
Performs core genome Multi-Locus Sequence Typing (cgMLST) analysis to assess strain relatedness.
1.  **Allele Calling**: `chewBBACA`
    *   **Process**: Performs allele calling on assembled genomes against a provided [cgMLST schema](https://www.cgmlst.org/ncs/schema/Kpneumoniae312/).

## FHIR Converter
Converts genomic typing data into HL7 FHIR R4 standard resources.
1.  **Input Parsing**: Reads typing, lineage JSON, and cgMLST files.
2.  **Mapping**:
    *   **Resistance Genes**: Mapped to LOINC codes and SNOMED CT for drug classes.
    *   **Strain Typing**: MLST mapped to LOINC.
    *   **Capsule Typing**: K-type mapped to LOINC.
    *   **Virulence**: Virulence scores and factors mapped to observation components.
3.  **Resource Creation**:
    *   Generates `Observation` resources for resistance genes, strain characteristics, and cgMLST assignment.
    *   Generates two `DiagnosticReport` resources. First, for the conclusion from all resistance genes detected, and second for the cgMLST statistics.


