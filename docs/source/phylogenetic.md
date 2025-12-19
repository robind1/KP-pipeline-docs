# FHIR Phylogenetic Analysis

## Overview
The [phylogenetic analysis pipeline](https://github.com/oucru-id/kp-phylo-analysis) processes FHIR JSON bundle files containing Core Genome MLST (cgMLST) data. The core logic transforms allelic profiles into a comparative genomic analysis to infer evolutionary relationships using Minimum Spanning Trees (MST).

```{image} _static/kpphyloflow.png
:alt: Phylogenetic Analysis Architecture Diagram
:width: 700px
:align: center
```

## Data Processing
### 1. FHIR Data Ingestion & Parsing
The pipeline iterates through input FHIR JSON bundle. For each file, it extracts:

*   **Metadata:**
    *   **Patient ID:** Extracted from `Patient` resources.
    *   **Geolocation:** Latitude and Longitude extracted from `Patient` address extensions.
    *   **Sequence Type (ST):** Extracted from 'Observation' resources (code 612-2).
    *   **Conclusion:** Extracted from `DiagnosticReport` resources.
*   **cgMLST Profiles:**
    *   The script scans `Observation` resources for LOINC code `SP000682` (Core Genome MLST Profile).
    *   It parses components to extract locus names and allele calls. Missing values or codes like 'LNF', 'NIPH' are standardized to '-'.

### 2. Profile Aggregation
1.  **Loci Union:** The script identifies the union of all loci found across the input dataset.
2.  **Profile Matrix:** A table is generated where rows represent samples and columns represent loci. Cells contain the specific allele identifier for that locus.

## Algorithm
### Distance Matrix Calculation
*   **Metric:** Hamming Distance (Allelic Distance).
*   **Calculation:** For every pair of samples, the distance is the count of loci where the allele calls differ (ignoring missing values).
*   **Output:** Matrix with `cgmlst-dists` format.

### Phylogenetic Tree Inference
1.  **Tree Construction:** The GrapeTree tool is used with the MSTreeV2 (Minimum Spanning Tree) algorithm based on the cgMLST profile.
2.  **Output:** The resulting tree is saved in **Newick (.nwk)** format.

## Tools & Libraries
| Library | Purpose |
| :--- | :--- |
| **GrapeTree** | 	Inferring phylogenetic trees using the MSTreeV2 algorithm from allelic profiles. |
| **Python Standard Library** | `json` for parsing FHIR, `re` for regex parsing of HGVS strings, `csv` for matrix output. |
| **Biopython** | Parsing Newick trees. |

## Outputs
1.  **`distance_matrix.tsv`**: Representing the number of allelic differences between every pair of samples.
2.  **`cgmlst_profile.tsv`**:  Matrix of samples vs. loci alleles.
2.  **`grapetree.nwk`**
3.  **`metadata.tsv`**: Sample metadata including Patient ID, Geolocation, and MLST/Conclusion.
4.  **Visualization:** Rectangular and Radial (Force-directed) renderings of the MST, colored by Sequence Type, heatmap of allelic distances, histogram of allelic distances, and violin plot.

```{image} _static/grapetree.png
:alt: Radial phylogenetic tree KP example
:width: 1200px
:align: center
```
Example of a phylogenetic tree generated from the pipeline. Data used: [Foster-Nyarko et al. 2023](https://doi.org/10.1099/mgen.0.000936), [Azra et al. 2025](https://doi.org/10.1038/s41598-025-27122-6), [Mejía-Limones et al. 2024](https://link.springer.com/article/10.1186/s12864-024-10835-9)
