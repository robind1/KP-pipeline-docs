# Overview
This Pipeline is a Nextflow-based workflow designed for the analysis of Klebsiella pneumoniae genomic data. It processes raw sequencing data (Illumina or Nanopore) to identify antimicrobial resistance (AMR) genes (kleborate), capsule types (kaptive), MLST typing (kleborate), virulence factors (kleborate), cgMLST schema (Ridom), and generates a FHIR-compliant genomics bundle. 

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
