---
name: set-operation-venn-quantification
description: Use when comparing a newly curated chemical database against one or more
  published predecessors and you need to report exact counts of additions, exclusions,
  and multi-way overlaps.
when_to_use_negative:
- When the input databases have not yet been harmonized to shared naming rules — raw
  alternate names (e.g. Cro-1I-dG vs 1,N2-(Me-OH-P)-dG) will produce inflated non-overlap
  counts
- When the goal is to annotate unknown HRMS peaks against the 582-entry database rather
  than to quantify database overlap — use the spectral-library/annotation workflow
  instead
- When chemical structures lack matching molecular formula and monoisotopic mass within
  the 220–640 Da window — identity matching becomes unreliable and set membership
  is not defensible
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: ChemDraw
  role: Source of the structural identifiers (.cdxml, .mol, SMILES, InChI, InChIKey,
    IUPAC names, monoisotopic masses, molecular formulas) used to harmonize and match
    adduct entries across databases
- name: liquid chromatography (LC)
  role: Experimental context for the DNA adductomics measurements that motivated building
    the curated database; not directly used in the set-operation step
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:d75acbef01bdc2093b22e520e16371606cbde007c5127e693f19d954e98e1995
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDraw
  - liquid chromatography (LC)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Set-operation-venn-quantification

## Summary

Quantify overlap and extension between a curated DNA adduct database and prior published adduct databases by harmonizing entry identities (names, molecular formulas, monoisotopic masses) and applying set operations to reproduce the Venn structure of database membership.

## When to use

Use when comparing a newly curated chemical database against one or more published predecessors and you need to report exact counts of additions, exclusions, and multi-way overlaps.

## When NOT to use

- When the input databases have not yet been harmonized to shared naming rules — raw alternate names (e.g. Cro-1I-dG vs 1,N2-(Me-OH-P)-dG) will produce inflated non-overlap counts
- When the goal is to annotate unknown HRMS peaks against the 582-entry database rather than to quantify database overlap — use the spectral-library/annotation workflow instead
- When chemical structures lack matching molecular formula and monoisotopic mass within the 220–640 Da window — identity matching becomes unreliable and set membership is not defensible

## Inputs

- Curated DNA adduct database (.xlsx/.docx, 279 literature-reported entries) from https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
- Three prior published databases: Hemeryck et al. 2015 (121 diet-related adducts), Carrà et al. 2019a (122 endogenous adducts), Guo et al. 2017 (102 bulky hydrophobic adducts)
- Systematic naming rules and structural identifiers (SMILES, InChI, InChIKey, IUPAC names, molecular formulas, monoisotopic masses) for each entry

## Outputs

- Pairwise and three-way overlap counts across the four databases (Venn structure of Figure 3A)
- Per-adduct membership table with name, molecular formula, monoisotopic mass, database flags, and inclusion/exclusion rationale (mirroring Supplementary Table S4)
- Verified count summary: 92 additional adducts and 54 excluded adducts relative to prior databases

## How to apply

Load the curated DNA adduct database (279 literature-reported entries as .xlsx/.docx from the GitLab repository) alongside each prior database (Hemeryck et al. 2015 with 121 diet-related adducts; Carrà et al. 2019a with 122 endogenous adducts; Guo et al. 2017 with 102 bulky hydrophobic adducts). Harmonize entry identities using the paper's systematic naming rules so alternate names resolve to a single identifier — e.g. map Cro-1I-dG to 1,N2-(Me-OH-P)-dG — and cross-check matches on molecular formula and monoisotopic mass within the 220–640 Da range. Run pairwise and three-way set operations on the harmonized identifiers to reconstruct the Venn structure shown in Figure 3A. Verify the reported counts explicitly: 279 current entries, 121/122/102 prior entries, 92 additional adducts retrieved from the literature beyond the three prior databases, and 54 adducts present in prior databases excluded for insufficient identification confidence. Tabulate per-adduct membership with name, formula, mass, database flags, and inclusion/exclusion rationale (mirroring Supplementary Table S4), and flag any count discrepancy beyond a single-adduct difference for expert review.

## Related tools

- **ChemDraw** (Source of the structural identifiers (.cdxml, .mol, SMILES, InChI, InChIKey, IUPAC names, monoisotopic masses, molecular formulas) used to harmonize and match adduct entries across databases)
- **liquid chromatography (LC)** (Experimental context for the DNA adductomics measurements that motivated building the curated database; not directly used in the set-operation step)

## Examples

```
# R: reproduce the 4-database Venn counts
current <- readxl::read_excel('dna_adductomics_database.xlsx'); prior <- list(hemeryck_2015, carra_2019a, guo_2017); n_added <- length(setdiff(ids(current), Reduce(union, prior)))
```

## Evaluation signals

- Harmonized-identifier set cardinalities equal the reported totals: 279 (current), 121, 122, and 102 (prior databases)
- Three-way overlap reproduces Figure 3A with exactly 92 additional adducts retrieved from the literature beyond the three prior databases
- Exactly 54 prior-database adducts are excluded, each with a recorded rationale (not found or not identified at sufficient confidence)
- Every matched pair agrees on both molecular formula and monoisotopic mass within the 220–640 Da range, with alternate names resolved via the paper's naming rules
- Discrepancies of more than a single adduct in any count are escalated to expert review rather than silently accepted

## Limitations

- Exclusions depend on the identification-confidence threshold applied during curation: 54 prior-database adducts were dropped for insufficient confidence, so borderline entries may flip between inclusion and exclusion
- Alternate-name mapping (e.g. Cro-1I-dG = 1,N2-(Me-OH-P)-dG) is error-prone without the paper's systematic naming rules; missed synonyms deflate overlap counts
- Mass/formula matching is restricted to the 220–640 Da monoisotopic-mass window; adducts outside this range cannot be matched by this route

## Evidence

- [results] Compared to the three prior databases, the current database includes 92 additional DNA adducts retrieved from the literature: [evidence span withheld — anchor status: not_found]
- [results] 54 adducts present in the previous databases were excluded due to insufficient identification confidence: [evidence span withheld — anchor status: not_found]
- [abstract] Harmonization of structures, molecular formulas, and names: "The second step consisted in harmonizing structures, molecular formulas, and names"
- [abstract] monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature: "monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature"
- [abstract] The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database ).: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
