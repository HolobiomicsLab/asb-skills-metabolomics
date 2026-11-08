---
name: cross-database-overlap-analysis
description: Use when comparing adduct or molecule inventories across multiple published
  databases to quantify overlap, novel additions, and exclusions via set operations
  on harmonized names, formulas, and monoisotopic masses.
when_to_use_negative:
- The task is annotating new HRMS/MS spectra against the adduct database rather than
  comparing database contents — use the spectral-library/annotation workflow instead
- The databases use incompatible identifier schemes with no available molecular formulas
  or monoisotopic masses for harmonization, making set operations unreliable
tools:
- name: ChemDraw
  role: Used to compute/verify monoisotopic masses, chemical formulas, and structural
    identifiers (.cdxml, .mol, SMILES, InChI, InChIKey, IUPAC names) when harmonizing
    adduct identities across databases
- name: rstudio/DT
  role: R DataTables interface used to render the resulting per-adduct overlap/exclusion
    tables as interactive web datatables
  repo: https://github.com/rstudio/DT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:28d2a1323d4c3005c42318af004e34a200841b9389a3b8a3143c6983e3cd2a6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - ChemDraw
  - rstudio/DT
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cross-database-overlap-analysis

## Summary

Quantifies overlap and extension between the curated 279-entry literature DNA adduct database and prior published adduct databases (Hemeryck et al. 2015, Carrà et al. 2019a, Guo et al. 2017) using set operations on systematically harmonized identifiers, yielding counts of novel additions and confidence-based exclusions (Figure 3A Venn, Supplementary Table S4).

## When to use

Use when comparing adduct or molecule inventories across multiple published databases to quantify overlap, novel additions, and exclusions via set operations on harmonized names, formulas, and monoisotopic masses.

## When NOT to use

- The task is annotating new HRMS/MS spectra against the adduct database rather than comparing database contents — use the spectral-library/annotation workflow instead
- The databases use incompatible identifier schemes with no available molecular formulas or monoisotopic masses for harmonization, making set operations unreliable

## Inputs

- Curated DNA adduct database (.xlsx/.docx with 279 literature-reported entries) from https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
- Prior database: Hemeryck et al. 2015 (121 diet-related DNA adducts)
- Prior database: Carrà et al. 2019a (122 endogenous DNA adducts)
- Prior database: Guo et al. 2017 (102 bulky hydrophobic DNA adducts)
- Systematic naming/alias rules (e.g., Cro-1I-dG = 1,N2-(Me-OH-P)-dG) with molecular formulas and monoisotopic masses in the 220–640 Da range

## Outputs

- Pairwise and three-way overlap (Venn) counts across the four databases (reproducing Figure 3A)
- Count of 92 additional adducts retrieved from literature beyond the prior databases
- Count of 54 adducts excluded due to insufficient identification confidence
- Per-adduct table with name, formula, mass, database membership, and inclusion/exclusion rationale (mirroring Supplementary Table S4)

## How to apply

Load the curated DNA adduct database (.xlsx/.docx, 279 literature-reported entries) from the GitLab repository (https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database) together with the three prior databases: Hemeryck et al. 2015 (121 diet-related adducts), Carrà et al. 2019a (122 endogenous adducts), and Guo et al. 2017 (102 bulky hydrophobic adducts). Harmonize entry identities using the paper's systematic naming rules, mapping alternate names (e.g., Cro-1I-dG = 1,N2-(Me-OH-P)-dG) and matching on molecular formula and monoisotopic mass, restricted to the 220–640 Da range. Compute pairwise and three-way overlaps of the four databases with set operations on the harmonized identifiers to reproduce the Venn structure of Figure 3A. Verify the reported counts: 92 additional adducts beyond the three prior databases and 54 adducts excluded for insufficient identification confidence. Tabulate per-adduct matches and exclusions with name, formula, mass, database membership, and inclusion/exclusion rationale, mirroring Supplementary Table S4; flag any discrepancy beyond a single-adduct difference for expert review.

## Related tools

- **ChemDraw** (Used to compute/verify monoisotopic masses, chemical formulas, and structural identifiers (.cdxml, .mol, SMILES, InChI, InChIKey, IUPAC names) when harmonizing adduct identities across databases)
- **rstudio/DT** (R DataTables interface used to render the resulting per-adduct overlap/exclusion tables as interactive web datatables) — https://github.com/rstudio/DT

## Examples

```
DT::datatable(overlap_table, rownames = FALSE)
```

## Evaluation signals

- Computed totals match the paper's explicit values: 279 current literature entries, 121/122/102 prior entries, 92 additional adducts, 54 excluded adducts
- Harmonized matches agree on both molecular formula and monoisotopic mass, with all entries restricted to the 220–640 Da range
- Alternate-name aliases (e.g., Cro-1I-dG vs 1,N2-(Me-OH-P)-dG) resolve to a single identifier rather than being double-counted
- Every excluded adduct carries a documented inclusion/exclusion rationale, and the tabulated output mirrors Supplementary Table S4
- Any discrepancy beyond a single-adduct difference between computed and reported overlap counts is flagged for expert review

## Limitations

- Exclusion of the 54 prior-database adducts rests on identification-confidence judgments; borderline cases may need expert review
- The comparison covers adducts within the 220–640 Da mass range; entries outside this range fall outside the analysis scope
- The full database contains 582 entries, but only the 279 literature-reported entries participate in this overlap analysis; the 303 new potential adducts are not part of the comparison

## Evidence

- [abstract] 279 literature entries: "for building a systematic database of 279 DNA adducts"
- [abstract] public database source: "The database, including 582 entries, is publicly available"
- [abstract] harmonization step: "The second step consisted in harmonizing structures, molecular formulas, and names"
- [readme] DT rendering: [evidence span withheld — anchor status: external_doc]
