---
name: dna-adduct-identifier-harmonization
description: Use when compiling or merging DNA adduct lists from different publications
  or databases and adduct names, formulas, or structures are inconsistent, duplicated,
  or ambiguous.
when_to_use_negative:
- Input is already a harmonized set of adduct identifiers and the goal is downstream
  MS/MS spectral matching rather than curation
- The entries are 'suspected' adducts (e.g. hypothetical genotoxicant–nucleobase combinations)
  that lack literature references and confirmed identification
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: ChemDraw
  role: Draw and standardize adduct structures and generate .cdxml/.mol files plus
    systematic names
- name: rstudio/DT
  role: R package used to present the curated/harmonized adduct database as interactive
    web datatables
  repo: https://github.com/rstudio/DT
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:f7134fc98085300df9ebabebdc279b4113e2c71e0e04ee0726d4995a2a0e2f16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - ChemDraw
  - rstudio/DT
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# dna-adduct-identifier-harmonization

## Summary

Harmonizes the structures, molecular formulas, and systematic names of literature-reported DNA adducts so entries from different sources can be compared, matched, and compiled into a single curated database.

## When to use

Use when compiling or merging DNA adduct lists from different publications or databases and adduct names, formulas, or structures are inconsistent, duplicated, or ambiguous.

## When NOT to use

- Input is already a harmonized set of adduct identifiers and the goal is downstream MS/MS spectral matching rather than curation
- The entries are 'suspected' adducts (e.g. hypothetical genotoxicant–nucleobase combinations) that lack literature references and confirmed identification

## Inputs

- Literature-reported DNA adduct entries with names, genotoxicants, study designs, and identification techniques
- Structural files (.cdxml, .mol) and structural identifiers (SMILES, InChI, InChIKey)
- Prior database entry lists (e.g. Hemeryck et al. 2015, Carrà et al. 2019a, Guo et al. 2017) for overlap reconciliation

## Outputs

- Harmonized adduct table with canonical names, formulas, monoisotopic masses, and structural identifiers (e.g. .xlsx / curated_core_adducts.csv)
- Cross-database overlap table with per-adduct membership, matches, and exclusion rationale
- Flagged discrepancies for mass mismatches > 0.01 Da or unresolved name ambiguity

## How to apply

Start from the literature search results that retain only adducts that were unequivocally identified or tentatively identified with strong supporting evidence (e.g. by HRMS/MS, reference standards, or NMR). For each adduct, standardize the structure using drawing tools such as ChemDraw and record the monoisotopic mass, chemical formula, SMILES, InChI, InChIKey, and IUPAC name, plus .cdxml and .mol files. Apply systematic naming rules so alternative designations of the same adduct map to one canonical identifier (e.g. Cro-1I-dG = 1,N2-(Me-OH-P)-dG); ensure positional isomers remain distinct entries. Harmonize names, formulas, and masses before computing overlaps against prior databases, matching entries by harmonized name, molecular formula, and monoisotopic mass within the 220–640 Da range. Validate by recomputing exact masses from .mol structures with RDKit and requiring agreement with deposited [M+H]+ or [M]+ values within 0.01 Da, and by confirming that the harmonized core reproduces the reported totals (279 literature adducts from 16 genotoxicant classes and 9 sources).

## Related tools

- **ChemDraw** (Draw and standardize adduct structures and generate .cdxml/.mol files plus systematic names)
- **rstudio/DT** (R package used to present the curated/harmonized adduct database as interactive web datatables) — https://github.com/rstudio/DT

## Examples

```
df = pd.read_excel('dna_adduct_database.xlsx', engine='openpyxl'); core = df[df['reference'].notna()]  # isolate 279 literature-derived entries
```

## Evaluation signals

- Recomputed monoisotopic mass from .mol/SDF structure matches the deposited [M+H]+ or [M]+ value within 0.01 Da
- Core literature-derived subset contains exactly 279 entries, each with non-null name, formula, mass, source, study design, and identification technique
- Cross-database overlap reproduces reported counts (121/122/102 prior entries, 92 additional, 54 excluded) within ±1 adduct
- Alternative names map to a single canonical identifier (e.g. Cro-1I-dG = 1,N2-(Me-OH-P)-dG) with no duplicate rows for the same structure
- Positional-isomer entries remain distinct rather than collapsed by shared formula/mass

## Limitations

- Excluded adducts were removed because of insufficient identification confidence, so overlaps against less-stringently curated databases will under-represent those entries
- Mass and formula matching alone cannot resolve positional isomers, which require the harmonized names and structures
- Only adducts within the 220–640 Da monoisotopic-mass range were targeted, so adducts outside this window are out of scope

## Evidence

- [abstract] The second step consisted in harmonizing structures, molecular formulas, and names: "The second step consisted in harmonizing structures, molecular formulas, and names"
- [abstract] Literature-selection criterion for the harmonized core: "The first step included a literature search, selecting only DNA adducts that were unequivocally identified"
- [abstract] Repository hosting the harmonized database: "The database, including 582 entries, is publicly available"
- [abstract] Structural identifiers recorded per adduct: "monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature"
- [abstract] Curated core of literature-reported adducts: "for building a systematic database of 279 DNA adducts"
