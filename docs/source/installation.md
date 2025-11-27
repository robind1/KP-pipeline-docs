# Installation

## Prerequisites
To run this pipeline, you need the following installed on your system:

*   [Nextflow](https://www.nextflow.io/)
*   [Docker](https://www.docker.com/)
*   [Python](https://www.python.org/)
*   [FastQC](https://github.com/s-andrews/FastQC)
*   [MultiQC](https://github.com/MultiQC/MultiQC)
*   [Fastp](https://github.com/OpenGene/fastp) 
*   [Miniasm](https://github.com/lh3/miniasm)
*   [Megahit](https://github.com/voutcn/megahit) 
*   [Medaka](https://github.com/nanoporetech/medaka)
*   [Flye](https://github.com/mikolmogorov/Flye)
*   [Pilon](https://github.com/broadinstitute/pilon)
*   [SPAdes](https://github.com/ablab/spades)
*   [samtools](https://github.com/samtools/samtools)  
*   [FHIR validator](https://github.com/hapifhir/org.hl7.fhir.validator-wrapper)

## Setup
1.  Clone the repository for local installation:
    ```bash
    git clone https://github.com/robind1/KPmutationpipeline.git
    cd KPmutationpipeline
    ```
2.  Installl Docker:
    ```bash
    curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
    ```
3.  Installl Nextflow:
    ```bash
    curl -s https://get.nextflow.io | bash
    ```
4.  Testing the Nextflow install:
    ```bash
    nextflow -v
    ```