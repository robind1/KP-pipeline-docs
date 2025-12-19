# Installation

## Prerequisites
To run this pipeline, you need the following installed on your system:

*   [Nextflow](https://www.nextflow.io/)
*   [Python](https://www.python.org/)
*   [FastQC](https://github.com/s-andrews/FastQC)
*   [MultiQC](https://github.com/MultiQC/MultiQC)
*   [Fastp](https://github.com/OpenGene/fastp)
*   [chopper](https://github.com/wdecoster/chopper) 
*   [Megahit](https://github.com/voutcn/megahit)
*   [SPAdes](https://github.com/ablab/spades)
*   [Medaka](https://github.com/nanoporetech/medaka)
*   [Flye](https://github.com/mikolmogorov/Flye)
*   [Pilon](https://github.com/broadinstitute/pilon)
*   [chewBBACA](https://github.com/B-UMMI/chewBBACA)
*   [samtools](https://github.com/samtools/samtools)  
*   [FHIR validator](https://github.com/hapifhir/org.hl7.fhir.validator-wrapper)

## Setup
1.  Clone the repository for local installation:
    ```bash
    git clone https://github.com/robind1/kp-mutation-pipeline.git
    cd kpmutationpipeline
    ```
2.  Install Nextflow:
    ```bash
    curl -s https://get.nextflow.io | bash
    ```
3.  Testing the Nextflow install:
    ```bash
    nextflow -v
    ```

