# Version

## Pipeline version

**2.0.0** (in development on the
[`2.0.0-dev`](https://github.com/oucru-id/kp-to-fhir-full/tree/2.0.0-dev) branch)

## Software

| Tool | Version |
| --- | --- |
| FastQC | 0.12.1 |
| fastp | 0.23.4 |
| SPAdes | 4.3.0 (new in 2.0; replaces MEGAHIT 1.2.9) |
| bwa | 0.7.17 |
| samtools | 1.21 |
| Pilon | 1.24 |
| chopper | 0.12.0 |
| NanoPlot | 1.48.0 |
| Flye | 2.9.6 |
| medaka | 2.1.0 |
| Kleborate | 3.2.4 |
| MOB-suite | 3.1.9 |
| minimap2 | 2.26 |
| chewBBACA | 3.4.2 |
| MultiQC | 1.30 |
| HL7 FHIR validator | 6.5.26 |
| Python / Java | 3.12 / 21 |

Apart from SPAdes, these are the versions that produced the 1.4.2 results. The versions used in a
run are collected in `pipeline_info/software_versions.yml`.

## Databases

- **Kleborate 3.2.4**: its bundled MLST, Kaptive and resistance databases, including the CARD
  alleles used to locate resistance genes.
- **MOB-suite**: the database downloaded by `mob_init` (about 3 GB), or the one given with
  `--mobsuite_db`.
- **cgMLST**: the cgMLST.org *K. pneumoniae* / *variicola* / *quasipneumoniae* scheme
  ([2187931](https://www.cgmlst.org/ncs/schema/2187931/), 2,358 targets), converted with chewBBACA
  `PrepExternalSchema`.
- **FHIR**: the Genomics Reporting IG package `hl7.fhir.uv.genomics-reporting#3.0.0`.

## What changed in 2.0

- **Inputs**: a samplesheet (`--input`) with Illumina, ONT and FASTA samples; the 1.4 folder input
  is kept as `--reads_dir`.
- **Outputs**: `--outdir` and a new layout (`assembly/`, `qc/`, `typing/`, `plasmid/`, `cgmlst/`,
  `reports/`, `fhir/`, `pipeline_info/`).
- **FHIR**: one merged bundle per sample is replaced by a genomic bundle without patient data, and
  optional clinical resources and linked bundles. Two profiles, `generic` (default) and
  `satusehat`.
- **Illumina assembly** with SPAdes instead of MEGAHIT.
- **Plasmid analysis** with MOB-suite, and the location of every resistance gene.
- **Validation** of every bundle with the HL7 FHIR validator, offline by default.
- **Credentials** as Nextflow secrets; upload only with `--fhir_server_url`.
- **Containers or conda** for every process.

See the
[changelog](https://github.com/oucru-id/kp-to-fhir-full/blob/2.0.0-dev/CHANGELOG.md) for every
change, including the breaking changes and fixes.

## References

1. Lam, M. M. C., et al. (2021). *A genomic surveillance framework and genotyping tool for
   Klebsiella pneumoniae and its related species complex*.
   [Nature Communications](https://doi.org/10.1038/s41467-021-24448-3).
2. Wyres, K. L., et al. (2016). *Identification of Klebsiella capsule synthesis loci from whole
   genome data*. [Microbial Genomics](https://doi.org/10.1099/mgen.0.000102).
3. Robertson, J., & Nash, J. H. E. (2018). *MOB-suite: software tools for clustering,
   reconstruction and typing of plasmids from draft assemblies*.
   [Microbial Genomics](https://doi.org/10.1099/mgen.0.000206).
4. Silva, M., et al. (2018). *chewBBACA: A complete suite for gene-by-gene schema creation and
   strain identification*. [Microbial Genomics](https://doi.org/10.1099/mgen.0.000166).
5. Prjibelski, A., et al. (2020). *Using SPAdes de novo assembler*.
   [Current Protocols in Bioinformatics](https://doi.org/10.1002/cpbi.102).
6. Kolmogorov, M., et al. (2019). *Assembly of long, error-prone reads using repeat graphs*.
   [Nature Biotechnology](https://doi.org/10.1038/s41587-019-0072-8).
7. Walker, B. J., et al. (2014). *Pilon: an integrated tool for comprehensive microbial variant
   detection and genome assembly improvement*.
   [PLOS ONE](https://doi.org/10.1371/journal.pone.0112963).
8. Chen, S., et al. (2018). *fastp: an ultra-fast all-in-one FASTQ preprocessor*.
   [Bioinformatics](https://doi.org/10.1093/bioinformatics/bty560).
9. Di Tommaso, P., et al. (2017). *Nextflow enables reproducible computational workflows*.
   [Nature Biotechnology](https://www.nature.com/articles/nbt.3820).
10. HL7 International. *Genomics Reporting Implementation Guide 3.0.0*.
    [hl7.org](https://hl7.org/fhir/uv/genomics-reporting/STU3/).
