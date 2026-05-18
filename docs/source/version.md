# Version

## Pipeline Version
* **Current Version:** 1.4.1

## Software Dependencies
The pipeline integrates several bioinformatics tools. Specific versions used in your run are automatically captured in the `software_versions.yml`.

## Changelog

### 1.4.1
- Upload to FHIR server using Auth2, including helper scripts: get_access_token.py
- expanded metadata CSV to fill the FHIR resources.

### v1.4.0
- Added O-antigen as a FHIR Observation
- Included Sequence Type (ST) and O-antigen in the text sample report
