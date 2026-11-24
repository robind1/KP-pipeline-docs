# Pipeline architecture

Every sample follows the branch for its platform to an assembly, then joins the shared steps. The
plasmid, cgMLST, clinical, link and upload steps run only when their parameters are given (the
plasmid step runs by default).

```mermaid
flowchart TD
    IN[/"Samplesheet or reads folder"/]
    IN --> ILL["Illumina<br/>FastQC · fastp · SPAdes · bwa · Pilon"]
    IN --> ONT["ONT<br/>chopper · NanoPlot · Flye · medaka"]
    IN --> FA["FASTA<br/>used as is"]
    ILL & ONT & FA --> ASM[("Assembly")]
    ONT --> QC["Coverage and polishing QC"]
    ASM --> TYP["Kleborate<br/>MLST · K/O loci · virulence · AMR"]
    ASM --> PLA["MOB-suite · minimap2<br/>plasmids, gene location"]
    ASM --> CG["chewBBACA cgMLST<br/>(with --cgmlst_schema)"]
    TYP & PLA & CG & QC --> GEN["FHIR genomic bundle<br/>(no patient data)"]
    TYP & PLA --> REP["Sample report"]
    META[/"Clinical metadata<br/>(optional)"/] --> CLIN["FHIR clinical resources"]
    GEN & CLIN --> LINK["Linked transaction bundle"]
    GEN & LINK --> VAL["FHIR validation<br/>IG 3.0.0"]
    VAL --> UP["Upload<br/>(optional)"]
```
