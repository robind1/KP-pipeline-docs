# Known limitations

## No species identification

Every sample is assumed to be *K. pneumoniae*. Kleborate runs without its species module, so the
output has no species call. Confirm the species separately, for example with Kleborate's
`enterobacterales__species` module or another species check, to catch other *Klebsiella* species
or contamination.

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

Ceftazidime and Cefepime share LOINC `18879-7` in the panel.

## Drug classes without a code

Genes from Kleborate's `Phe` (phenicol), `Fcyn` (fosfomycin), `Gly` (glycopeptide) and `Tgc`
(tigecycline) columns are reported without the *effect on drug efficacy* component, because those
class names have no SNOMED CT mapping.

## Point-mutation notation

Kleborate's porin and truncation notations (e.g. `OmpK36GD`, `MgrB-62%`) are not HGVS. The amino
acid change component is filled only when the notation looks like a substitution.
