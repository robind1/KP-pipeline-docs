# Output Files
The primary output is the **HL7 FHIR** bundle and per-sample genomic reports.

## FHIR Genomic Bundle

### Observation Resources
Each detected resistance feature generates an Observation resource containing:

*   **Resistance Genes** (`code: 69548-6`: Genetic variant assessment)
    *   **Gene ID**: Standard gene symbol (e.g., *blaCTX-M-15*), encoded using LOINC `48018-6`.
    *   **Drug Class**: SNOMED CT codes for affected drug classes, encoded using LOINC `51961-1`.
    *   **Context**: Distinguishes between acquired resistance genes and intrinsic chromosomal variants (e.g., *blaSHV-1*). Narrow-spectrum SHV variants are excluded from resistance classification.
    *   **Amino Acid Change (Point Mutations)**: pHGVS notation (e.g., `p.(Ser83Leu)`) encoded using LOINC `48005-3`.

*   **Strain Typing**
    *   **MLST** (`code: 612-2`): Sequence Type (ST) (e.g., ST258), referenced against `http://pubmlst.org/klebsiella`.
    *   **cgMLST** (`code: SP000682`): Core genome MLST profile including called loci count, total loci, call rate %, and per-locus allele assignments referenced against `https://www.cgmlst.org/ncs`.

*   **Capsule & Serotyping**
    *   **K-Type** (`code: SP000678`): Capsule type (e.g., K64), referenced against `http://kaptive.holtlab.net/capsule`.

*   **Virulence**
    *   **Virulence Score** (`code: SP000680`): Quantitative score (0–5) based on the presence of key virulence factors.

*   **Predicted Susceptibility Panel** (`code: 29576-6` — Bacterial susceptibility panel)
    *   Genotypic prediction for 12 antimicrobials: Ampicillin, Amoxicillin-Clavulanate, Piperacillin-Tazobactam, Cefotaxime, Ceftazidime, Cefepime, Meropenem, Ciprofloxacin, Gentamicin, Amikacin, Trimethoprim-Sulfamethoxazole, Colistin.
    *   Each drug result is encoded with a LOINC code and LOINC interpretation (`LA6676-6` = Resistant, `LA24225-7` = Susceptible).
    *   Resistance is inferred from: carbapenemase genes, ESBL genes, aminoglycoside gene mapping (granular per-drug), OmpK35/36 porin mutations, fluoroquinolone target mutations (GyrA, ParC), colistin regulatory mutations (MgrB, PhoP/Q, PmrA/B, CrrA/B), and mobile colistin resistance genes (*mcr*).

### DiagnosticReport Resource (`code: 81247-9`)
*   **Conclusion**: A human-readable summary including Resistance Classification, MLST Sequence Type, Clonal Complex, Capsule Type, and Virulence Score.
*   **ConclusionCode**: Structured SNOMED CT or local terminology coding for the resistance classification.
*   **Presentation**: Contains a Base64-encoded HTML representation of the full report including detected genes, MLST allele profile, and susceptibility data.

## Drug Resistance Classification

| Classification | Definition | Logic | ConclusionCode |
| :--- | :--- | :--- | :--- |
| **Susceptible** | No significant resistance | No acquired resistance genes detected; intrinsic ampicillin resistance only | `KP-SO` |
| **Resistant** | Low-level resistance | Resistance genes detected for <3 classes, no ESBL/AmpC/carbapenemase | `KP-RL` |
| **ESBL** | Extended-Spectrum Beta-Lactamase | Presence of *blaCTX-M*, or ESBL-type *blaSHV*/*blaTEM* variants; <3 drug classes affected | SNOMED `409801009` |
| **Multidrug-Resistant ESBL** | ESBL + additional resistance | ESBL genes + ≥3 antimicrobial classes affected | `SP000687` |
| **AmpC** | AmpC Beta-Lactamase | Presence of plasmid-mediated AmpC genes (*dha*, *cmy*, *mox*, *fox*, etc.); <3 drug classes | SNOMED `1098101000112102` |
| **Multidrug-Resistant AmpC** | AmpC + additional resistance | AmpC genes + ≥3 antimicrobial classes affected | `SP000683` |
| **MDR** | Multidrug-Resistant | Resistance genes detected for ≥3 antimicrobial classes (no ESBL/AmpC/carbapenemase) | SNOMED `714315002` |
| **CRE** | Carbapenem-Resistant Enterobacteriaceae | Presence of carbapenemase genes (e.g., *blaKPC*, *blaNDM*, *blaVIM*, *blaIMP*, *blaOXA-48/181/232*) | SNOMED `1098201000112108` |
| **XDR** | Extensively Drug-Resistant | CRE + colistin resistance (chromosomal mutation or *mcr* gene) | `SP000681` |

> **Note**: Narrow-spectrum SHV variants (e.g., *blaSHV-1*, *blaSHV-11*) are classified as intrinsic and are **excluded** from resistance classification. Chromosomal gene context is detected from observation narrative text and skipped unless the gene confers a higher-tier resistance class.
