# Configuration

Every parameter is defined in
[`nextflow_schema.json`](https://github.com/oucru-id/kp-to-fhir-full/blob/2.0.0-dev/nextflow_schema.json)
and listed by `nextflow run . --help`. Values are checked against the schema before anything runs,
on Nextflow 25.04, 25.10 and 26.04 alike. Switches accept `--plasmid_analysis`,
`--plasmid_analysis true` and `--plasmid_analysis false`.

:::{note}
On Nextflow 25.04 and 25.10, a command-line value that looks like a number is turned into one, so
leading zeros are lost (`--performer_org_id 0123` becomes `123`). Give such ids in a params file
instead:

```bash
printf 'performer_org_id: "0123"\n' > params.yml
nextflow run . -params-file params.yml --input samplesheet.csv
```
:::

## Most used parameters

| Parameter | Default | Purpose |
| --- | --- | --- |
| `--input` | | samplesheet (see [Inputs](inputs.md)) |
| `--outdir` | `results` | output folder |
| `--fhir_profile` | `generic` | `generic` or `satusehat` (see [FHIR standards](FHIR.md)) |
| `--patient_metadata` | | enables the clinical and link steps |
| `--performer_org_id` | | performing laboratory, as the genomic report's performer |
| `--genome_size` | `5.5m` | Flye genome size and Nanopore coverage estimate |
| `--medaka_model` | `r941_min_sup_g507` | must match the flow cell and basecaller |
| `--min_nanopore_coverage` | `30` | below this, the report carries a quality warning |
| `--plasmid_analysis` | `true` | MOB-suite and gene location |
| `--cgmlst_schema` | | enables cgMLST |
| `--skip_fhir_validation` | `false` | skip the HL7 FHIR validator |
| `--max_cpus` / `--max_memory` / `--max_time` | `8` / `16 GB` / `24h` | cap on any single task |

## All parameters

### Input and output

| Parameter | Default | Description |
| --- | --- | --- |
| `input` | | Samplesheet CSV with columns `sample,platform,fastq_1,fastq_2,fasta` (platform `illumina`, `ont` or `fasta`). |
| `reads_dir` | | Folder input: `<id>_1_illumina.fastq.gz` / `<id>_2_illumina.fastq.gz` and `<id>_ont.fastq.gz` files. |
| `outdir` | `results` | Output directory. (`--results_dir` was removed in 2.0.0.) |

### Read processing and assembly

| Parameter | Default | Description |
| --- | --- | --- |
| `genome_size` | `5.5m` | Expected genome size, for Flye (`--genome-size`) and the Nanopore coverage estimate. |
| `nanopore_min_q` | `10` | chopper minimum read quality. |
| `nanopore_min_l` | `500` | chopper minimum read length. |
| `medaka_model` | `r941_min_sup_g507` | medaka consensus model; must match the flow cell and basecaller of the ONT reads. |
| `medaka_batch_size` | `20` | medaka inference batch size (lower it when a GPU runs out of memory). |
| `min_nanopore_coverage` | `30` | Warn, and flag the FHIR report, below this Nanopore read coverage (after chopper). |

### Plasmid analysis

| Parameter | Default | Description |
| --- | --- | --- |
| `plasmid_analysis` | `true` | Run MOB-suite and locate Kleborate resistance genes on plasmids or the chromosome. |
| `mobsuite_db` | | Initialised MOB-suite database directory. When unset, `mob_init` downloads it (about 3 GB) into `--mobsuite_db_cache` once. |
| `mobsuite_db_cache` | `db/mob_suite` in the pipeline folder | Where `mob_init` stores the MOB-suite database when `--mobsuite_db` is unset. |
| `plasmid_chrom_min_contig` | `20000` | Shortest chromosome contig (bp) on which a gene counts as chromosomal; on shorter contigs it is `unresolved`. |
| `amr_locate_min_identity` | `90` | Minimum identity (%) of a CARD allele hit used to locate a gene. |
| `amr_locate_min_coverage` | `80` | Minimum allele coverage (%) of a CARD allele hit used to locate a gene. |

### cgMLST

| Parameter | Default | Description |
| --- | --- | --- |
| `cgmlst_schema` | | chewBBACA schema directory (see [Installation](installation.md#cgmlst-schema)). cgMLST is skipped when unset. |

### FHIR output

| Parameter | Default | Description |
| --- | --- | --- |
| `fhir_profile` | `generic` | `generic`: HL7 Genomics Reporting IG 3.0.0 only. `satusehat`: 1.4 output with Kemkes profiles, NIK identifiers and codings. |
| `performer_org_id` | | FHIR id of the performing laboratory Organization, used as performer in the genomic bundles. |
| `performer_org_name` | | Display name for the performing laboratory. |
| `identifier_system_base` | | Base URI for business identifiers in the generic profile (none are emitted if unset). |

### Clinical metadata (optional, contains personal data)

| Parameter | Default | Description |
| --- | --- | --- |
| `patient_metadata` | | Patient table (CSV or XLSX); see [Clinical metadata](clinical_metadata.md). |
| `organization_metadata` | | Performing organization (first row is used). |
| `practitioner_metadata` | | Responsible practitioner (first row is used). |
| `external_refs` | | JSON `{fhir_id: {"Patient": "<server id>", "Encounter": "<server id>"}}` for resources that already exist on the server. |
| `require_clinical` | `false` | Fail when a sample has no row in the patient table (default: warn, genomic bundle only). |
| `bundle_request_method` | `PUT` | `PUT` or `POST`. POST uses server-assigned ids and `ifNoneExist` on NIK (SATUSEHAT). |
| `publish_clinical` | `true` | Publish the clinical resources to `<outdir>/fhir/clinical`. |

### FHIR validation

| Parameter | Default | Description |
| --- | --- | --- |
| `skip_fhir_validation` | `false` | Do not run the HL7 FHIR validator. |
| `fhir_validator` | HAPI FHIR core 6.5.26 release | `validator_cli.jar`: a local path or a URL. |
| `fhir_ig` | `hl7.fhir.uv.genomics-reporting#3.0.0` | Implementation guide package to validate against. |
| `fhir_tx_server` | `n/a` | Terminology server for code validation; `n/a` validates offline and sends nothing. |
| `fail_on_validation_error` | `true` for generic, `false` for satusehat | Stop the run when a bundle has validation errors. |
| `fhir_validation_batch_size` | `50` | Files validated per validator process. |

### FHIR upload

| Parameter | Default | Description |
| --- | --- | --- |
| `fhir_server_url` | | FHIR server base URL. Upload runs only when this is set. |
| `fhir_auth_url` | | OAuth2/OIDC realm URL for the client-credentials token. |
| `fhir_upload_api_key` | `false` | Also send the `KPFHIR_API_KEY` secret as an `X-Api-Key` header. |

Credentials are Nextflow secrets; see [Installation](installation.md#uploading-to-a-fhir-server).

### Resource limits

| Parameter | Default | Description |
| --- | --- | --- |
| `max_cpus` | `8` (fewer if the machine has fewer) | Most CPUs any task may request. |
| `max_memory` | `16 GB` | Most memory any task may request. |
| `max_time` | `24h` | Longest any task may run. |
| `trace_report_suffix` | start time | Suffix for the `pipeline_info` report file names. |

Tasks that run out of memory are retried once with twice the memory, up to `--max_memory`.

`--max_cpus` and `--max_memory` cap each task, not the run. To cap the whole run, for example on a
machine shared with other work, set the executor limits in a config file:

```text
executor {
    cpus   = 4
    memory = 16.GB
}
```

and pass it with `-c <file>`.

## Removed and renamed parameters

| 1.4 | 2.0 |
| --- | --- |
| `--results_dir` | `--outdir` |
| `--skip_validation` | `--skip_fhir_validation` |
| `--skip_upload` | upload runs only with `--fhir_server_url` |
| `genome_size_bp`, Flye's fixed `5.5m` | `--genome_size` |
| `--api_key`, `data/access_token.json`, `data/input_sso.json` | Nextflow secrets `KPFHIR_CLIENT_ID`, `KPFHIR_CLIENT_SECRET`, `KPFHIR_API_KEY` |
| `--nanoplot_env`, `--mobsuite_env` | removed; tools come from containers or conda |
