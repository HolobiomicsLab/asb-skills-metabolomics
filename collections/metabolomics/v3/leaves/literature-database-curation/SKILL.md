---
name: literature-database-curation
description: Use when compiling or updating a small-molecule/bioadduct annotation
  database from published sources, or when reproducing an overlap comparison between
  a new curated database and prior databases to report added entries and excluded
  entries with rationale.
when_to_use_negative:
- The goal is annotating raw LC-HRMS/MS spectra against the database (spectral matching
  is a downstream step, not the curation overlap analysis)
- The input is a set of predicted/in silico adducts rather than literature-reported
  entries whose identification confidence can be judged
- Prior databases cannot be harmonized because systematic names, formulas, or monoisotopic
  masses are unavailable for entries
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0091
tools:
- name: ChemDraw
  role: Used to generate and harmonize structural identifiers (.cdxml, .mol, SMILES,
    InChI, InChIKey, IUPAC names) and verify molecular formulas and monoisotopic masses
    during entry matching
- name: rstudio/DT
  role: R DataTables interface used to render the curated adduct entries and MS/MS
    fragment data as interactive web datatables for review
  repo: https://github.com/rstudio/DT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:489891f97ed90d1bd40f6734ebe3e6435905affc7a337f752143e6e2ddb2cb68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - ChemDraw
  - rstudio/DT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# literature-database-curation

## Summary

Curate a literature-derived chemical annotation database by searching prior published compilations (Hemeryck 2015, Carrà 2019a, Guo 2017), harmonizing entry identities via systematic naming plus molecular formula and monoisotopic mass matching, and computing set-overlaps against each prior database to quantify extensions and justified exclusions.

## When to use

Use when compiling or updating a small-molecule/bioadduct annotation database from published sources, or when reproducing an overlap comparison between a new curated database and prior databases to report added entries and excluded entries with rationale.

## When NOT to use

- The goal is annotating raw LC-HRMS/MS spectra against the database (spectral matching is a downstream step, not the curation overlap analysis)
- The input is a set of predicted/in silico adducts rather than literature-reported entries whose identification confidence can be judged
- Prior databases cannot be harmonized because systematic names, formulas, or monoisotopic masses are unavailable for entries

## Inputs

- Curated database file (.xlsx/.docx) with 279 literature-reported DNA adduct entries including names, molecular formulas, monoisotopic masses, SMILES/InChI/InChIKey
- Three prior published DNA adduct databases: Hemeryck et al. 2015 (121 diet-related adducts), Carrà et al. 2019a (122 endogenous adducts), Guo et al. 2017 (102 bulky hydrophobic adducts)
- Paper's systematic naming rules and alias mapping (e.g. Cro-1I-dG = 1,N2-(Me-OH-P)-dG)

## Outputs

- Pairwise and three-way overlap counts reproducing the Figure 3A Venn structure
- Verified totals: 92 additional adducts, 54 excluded prior-database adducts
- Per-adduct match/exclusion table with name, formula, mass, database membership, and rationale (mirroring Supplementary Table S4)

## How to apply

Load the curated database (e.g. the 279 literature-reported DNA adduct entries in the GitLab-hosted .xlsx/.docx) and each prior database (Hemeryck et al. 2015: 121 diet-related adducts; Carrà et al. 2019a: 122 endogenous adducts; Guo et al. 2017: 102 bulky hydrophobic adducts). Harmonize entry identities using systematic naming rules, mapping alternate names (e.g. Cro-1I-dG = 1,N2-(Me-OH-P)-dG), and confirm matches by molecular formula plus monoisotopic mass within the 220–640 Da range. Compute pairwise and three-way overlaps of the databases with set operations on the harmonized identifiers to reconstruct the Venn structure. Verify against the reported counts: 92 additional adducts retrieved from the literature beyond the three prior databases, and 54 prior-database adducts excluded for insufficient identification confidence. Tabulate per-adduct matches and exclusions (name, formula, mass, database membership, inclusion/exclusion rationale) mirroring Supplementary Table S4.

## Related tools

- **ChemDraw** (Used to generate and harmonize structural identifiers (.cdxml, .mol, SMILES, InChI, InChIKey, IUPAC names) and verify molecular formulas and monoisotopic masses during entry matching)
- **rstudio/DT** (R DataTables interface used to render the curated adduct entries and MS/MS fragment data as interactive web datatables for review) — https://github.com/rstudio/DT

## Examples

```
import pandas as pd; cur = pd.read_excel('dna_adducts_database.xlsx'); prev = {n: pd.read_excel(f'{n}.xlsx') for n in ['hemeryck2015','carra2019a','guo2017']}
```

## Evaluation signals

- Computed overlap counts match the paper exactly: 279 current entries, 121/122/102 prior-database entries, 92 additional retrieved adducts, 54 excluded adducts
- Any discrepancy beyond a single-adduct difference is flagged for expert review
- Every matched pair agrees on both molecular formula and monoisotopic mass within the 220–640 Da range
- Alternate-name aliases (e.g. Cro-1I-dG vs 1,N2-(Me-OH-P)-dG) resolve to the same harmonized identifier rather than being double-counted
- Each excluded entry carries an explicit rationale (not found, or not identified at sufficient confidence), mirroring Supplementary Table S4

## Limitations

- Overlap resolution depends on identification-confidence judgments from the source papers; 54 prior adducts were dropped solely for insufficient confidence, which is subjective at boundaries
- Harmonization is only as good as the naming aliases and mass/formula matching — entries outside the 220–640 Da mass range or lacking formula/mass data cannot be reliably matched
- Counts refer specifically to the 279 literature-reported adducts; the full 582-entry database additionally contains 303 potential adducts not literature-reported, which should not enter the overlap comparison

## Evidence

- [abstract] The database, including 582 entries, is publicly available: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
- [abstract] Harmonization step: "The second step consisted in harmonizing structures, molecular formulas, and names"
- [abstract] Structural identifier formats: "monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature"
- [abstract] Literature-reported subset size: "for building a systematic database of 279 DNA adducts"
- [readme] Datatables rendering: [evidence span withheld — anchor status: external_doc]
