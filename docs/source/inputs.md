# Inputs

Give the samples either in a samplesheet (recommended) or as a folder with fixed file names.

## Samplesheet

A CSV with the columns `sample,platform,fastq_1,fastq_2,fasta`:

```text
sample,platform,fastq_1,fastq_2,fasta
KP0001,illumina,/data/KP0001_R1.fastq.gz,/data/KP0001_R2.fastq.gz,
KP0002,ont,/data/KP0002.fastq.gz,,
KP0003,fasta,,,/data/KP0003.fasta
```

```bash
nextflow run oucru-id/kp-to-fhir-full -r 2.0.0-dev -profile docker \
  --input samplesheet.csv --outdir results
```

| `platform` | Needs |
| --- | --- |
| `illumina` | paired-end reads in `fastq_1` and `fastq_2` (`.fastq.gz` or `.fq.gz`) |
| `ont` | one reads file in `fastq_1` (`.fastq.gz` or `.fq.gz`) |
| `fasta` | an assembly in `fasta` (`.fasta`, `.fa` or `.fna`, optionally gzipped; a local path or a URL) |

A `fasta` sample skips assembly and read QC, and goes straight to typing, plasmid analysis and
cgMLST.

**Sample ids** may use letters, digits, `.`, `_` and `-`, must start with a letter or digit, and
may be up to 40 characters long. Each id must be unique.

```{warning}
Use laboratory accessions as sample ids. Sample ids appear in the genomic FHIR output, which is
meant to contain no patient data. The pipeline warns when a sample id looks like a 16-digit
national ID number (NIK).
```

## Folder (1.4 naming)

```bash
nextflow run . -profile docker --reads_dir /data/NGS
```

The files in the folder must be named:
- Illumina: `<id>_1_illumina.fastq.gz` and `<id>_2_illumina.fastq.gz`
- ONT: `<id>_ont.fastq.gz`

The default `data/NGS` folder of 1.4 is gone: give `--reads_dir` or `--input`.

## Nanopore reads

- `--medaka_model` must match the flow cell and basecaller. The default, `r941_min_sup_g507`, suits
  R9.4.1 flow cells; set the model for R10 data. List the available models with
  `medaka tools list_models`.
- `--genome_size` (default `5.5m`) is passed to Flye and used to estimate read coverage.
- Below `--min_nanopore_coverage` (default 30×) after read filtering, the report carries a quality
  warning.

## Clinical metadata (optional)

Patient, organization and practitioner tables turn the genomic reports into linked FHIR bundles.
They hold personal data and are described in [Clinical metadata](clinical_metadata.md).
