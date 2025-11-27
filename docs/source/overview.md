# Overview

This Pipeline is a Nextflow-based workflow designed for Klebsiella pneumoniae genomic analysis. It processes raw sequencing data (Illumina or Nanopore) to identify antimicrobial resistance (AMR) genes (CARD database), determine capsule types, perform MLST typing, assess virulence factors, and generate FHIR-compliant clinical reports. 

## Key Features
* **Multi-platform Support**: Handles both Illumina paired-end (short-read) and Nanopore long-read sequencing data.
* **Comprehensive Typing**: MLST, capsule typing (K/O locus), wzi typing.
* **Antimicrobial Resistance Detection**: Identifies resistance genes including ESBL, carbapenemases, and other acquired resistance.
* **Virulence Factor Assessment**:Yersiniabactin, colibactin, aerobactin, salmochelin, and hypermucoidy markers.
* **Clinical Integration**: Merges genomic data with clinical metadata.
* **Quality Control**: QC reporting with MultiQC.

## Key Outputs
* Genome assembly
* Typing and resistance characterization
* FHIR-compliant genomic reports 
* Quality control metrics