# Known limitations

Most of these are kept from 1.4 on purpose: 2.0 changes the structure of the pipeline, not the
science.

## No species identification

Every sample is assumed to be *K. pneumoniae*. Kleborate runs without its species module, so the
output has no species call. Confirm the species separately, for example with Kleborate's
`enterobacterales__species` module or another species check, to catch other *Klebsiella* species
or contamination.

## Genotypic prediction only

Kleborate is the only resistance caller. The susceptibility panel and the resistance
classification come from a fixed rule set (`bin/kpfhir_resistance.py`), not from a phenotype. A
**Susceptible** drug means only that no known resistance gene or mutation for it was found;
mechanisms that Kleborate does not report, such as changes in efflux or gene expression, are
missed.

## Colistin resistance from mutations

Kleborate reports truncations and frameshifts in *MgrB*, *PmrB* and *PhoPQ* as colistin
resistance mutations and counts them in its own resistance score. KPtoFHIR does the same, so a
carbapenemase producer with such a call is classified **XDR**.

- Example: the HS11286 test genome. Kleborate reports `pmrB p.Gly20fs` and scores it 3
  (carbapenemase with colistin resistance); KPtoFHIR reports XDR.
- Frameshifts can be assembly or sequencing errors, especially at low coverage or in older,
  homopolymer-prone data. Check the mutation, and the colistin phenotype, before acting on an XDR
  call.

## Plasmid placement from short reads is a prediction

Short-read assemblies break plasmids at repeats, so many genes are `unresolved`, and MOB-suite's
assignment of small contigs is a prediction. Nanopore assemblies place genes more reliably. The
confidence of each plasmid result is recorded in the plasmid Observations.

## cgMLST schemas change during runs

chewBBACA adds newly inferred alleles to the schema during allele calling. Keep one copy of the
schema per analysis series, so that allele numbers stay comparable within the series.

## Susceptibility panel codes

Ceftazidime and Cefepime share LOINC `18879-7` in the panel, as in 1.4. A consumer that reads only
the code cannot tell the two components apart; the component text names the drug.

## Drug classes without a code

Genes from Kleborate's `Phe` (phenicol), `Fcyn` (fosfomycin), `Gly` (glycopeptide) and `Tgc`
(tigecycline) columns are reported without the *effect on drug efficacy* component, because those
class names have no SNOMED CT mapping.

## Point-mutation notation

Kleborate's porin and truncation notations (e.g. `OmpK36GD`, `MgrB-62%`) are not HGVS. The amino
acid change component is filled only when the notation looks like a substitution, as in 1.4.

## Codings kept from 1.4 in the satusehat profile

To reproduce the 1.4 output, `--fhir_profile satusehat` keeps two codings that the generic profile
replaces with text:
- genes coded under `http://www.genenames.org/geneId`, which is meant for human genes;
- point mutations coded under the LOINC system, with the mutation name as the code.

## Layout kept from 1.4

Every resistance Observation repeats the sample's virulence loci as components.

## Sequencing QC for Nanopore only

The sequencing QC Observation and its coverage warning are made for ONT samples only. For Illumina
samples, check the FastQC and fastp reports.

## Hand-written tables

- The clonal-complex table covers about 20 STs; other STs report `Unknown`.
- The `high_risk_clone` list (ST11, ST258, ST147, ST15) and the subspecies ST ranges in
  `lineage.json` are incomplete. They are used only in the text report.

## medaka model

The default `r941_min_sup_g507` suits R9.4.1 flow cells. Set `--medaka_model` for R10 data.

## SATUSEHAT upload not yet verified

Upload has not yet been tested in 2.0 against the SATUSEHAT sandbox or production platform. Test
against the sandbox first.
