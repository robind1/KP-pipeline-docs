# Overview

[KPtoFHIR](https://github.com/oucru-id/kp-to-fhir-full) is a Nextflow pipeline for
*Klebsiella pneumoniae* genomic analysis from raw sequencing data or finished assemblies. It
assembles the genome, then reports:
- [Kleborate](https://github.com/klebgenomics/Kleborate) typing: MLST, capsule (K) and O loci,
  virulence loci, resistance genes and point mutations;
- where each resistance gene sits: on a plasmid, on the chromosome, or unresolved
  ([MOB-suite](https://github.com/phac-nml/mob-suite));
- [chewBBACA](https://github.com/B-UMMI/chewBBACA) cgMLST.

The results are HL7 FHIR R4 resources that conform to the
[Genomics Reporting IG 3.0.0](https://hl7.org/fhir/uv/genomics-reporting/STU3/).

```{note}
**Version 2.0.0** changes the inputs, the output layout and the FHIR structure of 1.4. Read the
[breaking changes](https://github.com/oucru-id/kp-to-fhir-full/blob/2.0.0-dev/CHANGELOG.md#breaking-changes)
before upgrading.
```

## Key features

- **Three kinds of input**, given in one samplesheet:
  - Illumina paired-end reads, assembled with fastp, SPAdes and Pilon;
  - Oxford Nanopore reads, assembled with chopper, Flye and medaka;
  - finished assemblies (FASTA), which go straight to typing.
- **Kleborate 3 typing**:
  - MLST, capsule (K) and O loci, virulence loci, and Kleborate's virulence and resistance scores;
  - acquired resistance genes and point mutations;
  - a predicted 12-drug susceptibility panel and an overall resistance classification (CRE, XDR,
    ESBL, AmpC, MDR and so on).
- **Plasmid analysis**:
  - MOB-suite reconstructs the plasmids and gives the replicon, relaxase, mobility and cluster of
    each;
  - every resistance gene is placed on a plasmid or the chromosome, or reported as unresolved;
  - a table of the plasmid clusters shared between samples.
- **cgMLST** with chewBBACA and the cgMLST.org *K. pneumoniae* sensu lato scheme (2,358 loci).
- **QC**: FastQC and fastp (Illumina) or NanoPlot (ONT), collected in MultiQC. For ONT reads, a
  low read coverage is carried into the FHIR report as a quality warning.
- **Genomic FHIR without patient data, by default.** The genomic report refers to the patient by
  id only. Clinical resources are built in a separate, optional step and joined to the genomic
  report in a third step.
- **Two FHIR profiles**:
  - `generic` (default): plain HL7 Genomics Reporting.
  - `satusehat`: the 1.4 output for the Indonesian Ministry of Health SATUSEHAT platform (Kemkes
    profiles, NIK identifiers, Kemkes codings).

## What a run produces

For each sample:
- an assembly
- the Kleborate typing results
- the plasmids found and the location of each resistance gene
- a FHIR genomic bundle (`fhir/genomic/<id>.genomic.fhir.json`) with no patient data
- a plain-text summary report
- with clinical metadata, a FHIR `transaction` bundle that links the genomic report to the
  patient, specimen and encounter

For the whole run: cgMLST allele calls (with a cgMLST schema), the plasmid-sharing table, MultiQC
and the Nextflow run reports.

## Repository layout

```text
kp-to-fhir-full
├── main.nf                  entry point
├── workflows/kpfhir.nf      main workflow
├── subworkflows/local/      assembly_illumina, assembly_nanopore, typing, plasmid, cgmlst, fhir
├── modules/local/           one process per tool or step
├── bin/                     Python steps (Kleborate parsing, plasmids, QC, FHIR genomic / clinical / link, upload)
├── conf/                    resources, output layout, test profiles
├── assets/                  samplesheet schema, FHIR CodeSystems, metadata templates
├── tools/cgmlst/            build the chewBBACA schema
├── tools/satusehat/         SATUSEHAT token helper
├── tools/dev/               PHI and credential guard (also a pre-commit hook)
├── tests/                   pytest suite, synthetic fixtures, v1 equivalence goldens
├── nextflow.config          parameters and profiles
└── nextflow_schema.json     parameter schema
```
