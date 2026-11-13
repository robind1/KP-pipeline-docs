# Installation

## Requirements

- [Nextflow](https://www.nextflow.io/) **25.04 or later**, with Java 17 or later. On 26.04, use
  **26.04.2 or later**.
- **One** of the following to provide the tools:

| Profile | Provides the tools via |
| --- | --- |
| `-profile docker` (or `podman`) | pinned [BioContainers](https://biocontainers.pro/) images |
| `-profile singularity` (or `apptainer`) | the same images, e.g. on HPC |
| `-profile conda` (or `mamba`) | pinned bioconda / conda-forge packages |
| none | tools already on `PATH` (see [Running without a profile](#running-without-a-profile)) |

A `slurm` profile is also provided for clusters. Combine it with one of the profiles above, e.g.
`-profile slurm,singularity`.

Install Nextflow:

```bash
curl -s https://get.nextflow.io | bash
./nextflow -version
```

## Get the pipeline

Run it straight from GitHub:

```bash
nextflow run oucru-id/kp-to-fhir-full -r 2.0.0 -profile docker --input samplesheet.csv --outdir results
```

Or clone it and run the local copy:

```bash
git clone -b 2.0.0 https://github.com/oucru-id/kp-to-fhir-full.git
cd kp-to-fhir-full
nextflow run . -profile docker --input samplesheet.csv --outdir results
```

## Check the installation

Two test profiles are provided:

```bash
nextflow run . -profile test_stub -stub-run      # wiring only, no tools needed
nextflow run . -profile test,docker              # HS11286 (ST11, KPC-2) RefSeq assembly, downloaded at run time
```

The `test` run should report ST11 and KPC-2 in `results_test/typing/HS11286_typing.json`, and an
XDR classification in `results_test/fhir/genomic/HS11286.genomic.fhir.json`

## Databases

### MOB-suite

The plasmid step needs the MOB-suite database (about 3 GB):
- Give an initialised one with `--mobsuite_db <dir>`.
- Otherwise `mob_init` downloads it once into `--mobsuite_db_cache`. The default is `db/mob_suite`
  in the pipeline folder; when the pipeline runs from GitHub that folder is under
  `~/.nextflow/assets/`, so set `--mobsuite_db_cache` to a lasting location.
- `--plasmid_analysis false` skips the plasmid step and the download.

### cgMLST schema

cgMLST runs only with `--cgmlst_schema <dir>`. To build the schema from the cgMLST.org
*K. pneumoniae* / *variicola* / *quasipneumoniae* scheme
([2187931](https://www.cgmlst.org/ncs/schema/2187931/), 2,358 targets):

```bash
tools/cgmlst/fetch_schema.sh db/cgmlst_schema    # downloads the scheme and runs chewBBACA PrepExternalSchema
nextflow run . -profile docker --input samplesheet.csv --cgmlst_schema db/cgmlst_schema
```

## Running without a profile

Without a profile, these tools must be on `PATH`:
- for all runs: `kleborate`, `multiqc`, Python 3 (standard library only) and Java 17 or later
- for Illumina input: `fastqc`, `fastp`, `spades.py`, `bwa`, `samtools`, `pilon`
- for ONT input: `chopper`, `NanoPlot`, `flye`, `medaka_consensus`
- for the plasmid step: `mob_init`, `mob_recon`, `minimap2`
- for cgMLST: `chewBBACA.py`

## Uploading to a FHIR server

Upload is optional and runs only when `--fhir_server_url` is given. It sends only linked bundles
(which need [clinical metadata](clinical_metadata.md)) that passed validation. Credentials are
[Nextflow secrets](https://docs.seqera.io/nextflow/secrets/), never parameters or files in the
repository:

```bash
nextflow secrets set KPFHIR_CLIENT_ID     '<client id>'
nextflow secrets set KPFHIR_CLIENT_SECRET '<client secret>'

nextflow run . -profile docker --input samplesheet.csv \
  --patient_metadata site/patients.csv \
  --organization_metadata site/organization.csv \
  --practitioner_metadata site/practitioner.csv \
  --fhir_profile satusehat \
  --fhir_server_url https://<server>/fhir \
  --fhir_auth_url https://<sso>/realms/<realm>
```

- To also send an `X-Api-Key` header, set the `KPFHIR_API_KEY` secret and add
  `--fhir_upload_api_key`.
- `--bundle_request_method POST` sends server-assigned-id transactions (with `ifNoneExist` on
  NIK). `--external_refs` maps samples to Patients that already exist on the server.

The pipeline fetches its own token from `--fhir_auth_url`. `tools/satusehat/get_access_token.py`
is only for manual checks. It keeps the token in `~/.kpfhir`, not in the repository.
