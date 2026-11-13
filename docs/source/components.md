# Pipeline components

The Nextflow code is split into one module per tool or step (`modules/local/`) and subworkflows
that chain them (`subworkflows/local/`). The diagram is on [Pipeline architecture](architecture.md).

## Main workflow

**File:** `workflows/kpfhir.nf`

Reads the samplesheet (or the reads folder), checks the sample ids and routes each sample to the
branch for its platform. All samples run in parallel. Sample ids are passed to every step
explicitly and matched exactly; an input file that belongs to another sample is an error.

## Illumina (short-read) branch

**File:** `subworkflows/local/assembly_illumina.nf`

1. **Read QC**: FastQC.
2. **Trimming**: fastp with adapter detection for paired-end reads, sliding-window trimming from
   both ends (window 4, mean quality 20), bases of quality ≥ 20 qualified, at most 30% unqualified
   bases per read and a minimum read length of 50.
3. **Assembly**: SPAdes in `--isolate` mode. Contigs shorter than 500 bp are dropped, and SPAdes'
   working folder (including its corrected reads) is deleted once the contigs are copied out.
4. **Polishing**: the trimmed reads are mapped back with bwa mem, sorted with samtools, and Pilon
   corrects the assembly. The Java heap is 80% of the task memory.

```{note}
2.0 assembles Illumina reads with SPAdes instead of MEGAHIT, which 1.4 used. SPAdes is built for
single isolates, MEGAHIT for metagenomes. Results from Illumina reads therefore differ from 1.4.
```

## Nanopore (long-read) branch

**File:** `subworkflows/local/assembly_nanopore.nf`

1. **Read filtering**: chopper, minimum read quality `--nanopore_min_q` (10) and minimum length
   `--nanopore_min_l` (500).
2. **Read QC**: NanoPlot. The read coverage is estimated as the bases left after chopper divided
   by `--genome_size`.
3. **Assembly**: Flye (`--nano-raw`, `--genome-size`, two polishing iterations).
4. **Polishing**: medaka with the model given in `--medaka_model`. If inference fails, medaka is
   retried on the CPU. If polishing still fails, the unpolished assembly is used and the failure is
   reported.
5. **Assembly QC**: the coverage status is `PASS`, `LOW` (below `--min_nanopore_coverage`) or
   `UNKNOWN`. Low coverage and failed polishing become quality warnings in the FHIR report.

## FASTA input

**File:** `modules/local/prepare_assembly.nf`

A submitted assembly (gzipped or not, local or a URL) is checked to be a FASTA file and used as is.
It skips read QC and assembly.

## Typing

**File:** `subworkflows/local/typing.nf`

1. **Kleborate 3** on the assembly, with these modules:
   - MLST (`klebsiella_pneumo_complex__mlst`);
   - acquired resistance genes and point mutations, the resistance score and the gene and class
     counts (`klebsiella_pneumo_complex__amr`, `__resistance_score`, `__resistance_gene_count`,
     `__resistance_class_count`);
   - virulence loci: yersiniabactin, colibactin, aerobactin, salmochelin, *rmpADC* and *rmpA2*
     (`klebsiella__ybst`, `__cbst`, `__abst`, `__smst`, `__rmst`, `__rmpa2`) and the virulence
     score (`klebsiella_pneumo_complex__virulence_score`);
   - capsule (K) and O loci with Kaptive (`klebsiella_pneumo_complex__kaptive`), and *wzi*
     (`klebsiella_pneumo_complex__wzi`).
2. **Parsing** (`bin/kp_parse_kleborate.py`): the Kleborate table becomes
   `typing/<id>_typing.json` (MLST, capsule, O locus, virulence, resistance genes and mutations)
   and `typing/<id>_lineage.json`. Kleborate's `Bla_chr` genes are marked intrinsic. A
   header-only or non-Kleborate table is an error.

## Plasmid analysis

**File:** `subworkflows/local/plasmid.nf`. Runs unless `--plasmid_analysis false`.

1. **Database**: `mob_init` downloads the MOB-suite database once, unless `--mobsuite_db` is given
   (see [Installation](installation.md#mob-suite)).
2. **Plasmid reconstruction**: MOB-suite `mob_recon` sorts the contigs into chromosome and plasmids,
   and types each plasmid (replicon, relaxase, MPF, oriT, predicted mobility, cluster, nearest
   reference plasmid).
3. **Gene location**: the CARD alleles bundled with Kleborate are mapped to the assembly with
   minimap2. A hit counts when it reaches `--amr_locate_min_identity` (90%) and
   `--amr_locate_min_coverage` (80%). Each Kleborate resistance gene is then placed, in this
   order of precedence over all its copies:
   - **plasmid**: a copy lies on a contig MOB-suite assigned to a plasmid;
   - **chromosome**: a copy lies on a chromosomal contig of at least `--plasmid_chrom_min_contig`
     (20 kb);
   - **unresolved**: neither, for example a copy on a short chromosomal contig.

   Point mutations are reported as chromosomal.
4. **Summary** (`bin/parse_plasmids.py`): `plasmid/<id>_plasmid.json`, with a confidence that
   depends on the input: `long-read` (ONT), `predicted (short-read)` (Illumina) or
   `predicted (submitted assembly)` (FASTA).
5. **Plasmid sharing**: `plasmid/plasmid_sharing.tsv` lists every MOB-suite plasmid cluster in the
   run with its replicons, mobility, resistance genes and the samples that carry it, most widely
   shared first.

## cgMLST

**File:** `subworkflows/local/cgmlst.nf`. Runs only with `--cgmlst_schema`.

chewBBACA `AlleleCall` calls all assemblies of the run together against the schema, giving
`cgmlst/results_alleles.tsv` (one row per sample) and `cgmlst/cgmlst_statistics.txt`. See
[Installation](installation.md#cgmlst-schema) for building the schema.

## Resistance prediction and classification

**File:** `bin/kpfhir_resistance.py`

From the Kleborate genes and mutations, a fixed rule set predicts a 12-drug susceptibility panel
and an overall resistance classification. The rules are described on
[Outputs](outputs.md#predicted-susceptibility-panel).

## FHIR genomic step

**File:** `bin/kpfhir_genomic.py` (`modules/local/fhir_genomic.nf`)

Builds one bundle per sample, `fhir/genomic/<id>.genomic.fhir.json`, with no patient data:
- an Observation per resistance gene and point mutation, with its drug class and genomic location
- the predicted susceptibility panel
- MLST, virulence score, capsule and O antigen Observations
- the cgMLST profile (with a schema), the plasmid Observations (with the plasmid step) and the
  sequencing QC Observation (ONT)
- the DiagnosticReport with the resistance classification, plus Device and Provenance

The content of each resource is on [Outputs](outputs.md).

## Sample report

**File:** `bin/generate_sample_report.py`

A plain-text summary per sample, `reports/<id>.summary_report.txt`: typing, resistance, plasmids
and gene locations, and the sequencing QC.

## Clinical and link steps (optional)

**Files:** `bin/clinical_to_fhir.py`, `bin/link_fhir_bundle.py`

With `--patient_metadata`, the clinical step builds Patient, Specimen, Encounter and
ServiceRequest per sample, and Organization, Practitioner and PractitionerRole once. The link
step joins them with the genomic bundle into one FHIR `transaction` per sample. See
[Clinical metadata](clinical_metadata.md).

## FHIR validation

**File:** `modules/local/fhir_validate.nf`

Every genomic and linked bundle is checked with the HL7 FHIR validator against
`hl7.fhir.uv.genomics-reporting#3.0.0` and the project CodeSystems in `assets/fhir/`:
- Validation runs offline by default (`--fhir_tx_server n/a`), so no codes are sent to a public
  terminology server. Checks that need a terminology server are reported as not performed.
- Files are validated in batches of `--fhir_validation_batch_size` (50). The results of all
  batches are merged into `fhir/validation/validation_summary.tsv`; each batch's validator output
  is kept as `validation_<n>.json`.
- An error stops the run for `generic` and only warns for `satusehat`, whose Kemkes profiles the
  validator cannot resolve. Override with `--fail_on_validation_error true|false`.
- `--skip_fhir_validation` skips the step.

## Upload (optional)

**Files:** `bin/upload_fhir.py` (`modules/local/fhir_upload.nf`)

Runs only with `--fhir_server_url`. It sends linked bundles that passed validation, one at a
time, with a client-credentials token from `--fhir_auth_url`. A summary of the server's response
is saved in `fhir/upload/<id>.bundle.upload.json`. See
[Installation](installation.md#uploading-to-a-fhir-server).

## Run reports

- **MultiQC** collects the FastQC (Illumina) and NanoPlot (ONT) results into
  `qc/multiqc/multiqc_report.html`. The fastp reports are in `qc/fastp/`.
- **Nextflow reports** (execution report, timeline, trace) and `software_versions.yml`, with every
  tool version used, are in `pipeline_info/`.
