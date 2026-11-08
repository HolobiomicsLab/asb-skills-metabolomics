---
name: molecular-formula-mass-matching
description: Use when merging or comparing DNA adduct databases whose entries use
  inconsistent nomenclature — harmonize identities by chemical formula and monoisotopic
  mass before computing overlaps or inclusion/exclusion decisions.
when_to_use_negative:
- Input entries already share canonical identifiers (e.g. all carry InChIKey), so
  formula/mass matching adds no value
- Analyzing spectra directly — this skill is a database curation step, not an MS/MS
  spectral annotation task
- Adduct masses fall outside the 220–640 Da window covered by the database
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: ChemDraw
  role: Generate/verify canonical structural identifiers and molecular formulas for
    harmonized adduct entries
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:fe51c7edc7bb21227a026775d04d95b0c6b7e5cf991f30f810f81547f4774060
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDraw
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# molecular-formula-mass-matching

## Summary

Harmonize DNA adduct entries across databases by matching molecular formulas and monoisotopic masses within the 220–640 Da adductomics window, so that alternate names (e.g. Cro-1I-dG = 1,N2-(Me-OH-P)-dG) resolve to the same chemical entity.

## When to use

Use when merging or comparing DNA adduct databases whose entries use inconsistent nomenclature — harmonize identities by chemical formula and monoisotopic mass before computing overlaps or inclusion/exclusion decisions.

## When NOT to use

- Input entries already share canonical identifiers (e.g. all carry InChIKey), so formula/mass matching adds no value
- Analyzing spectra directly — this skill is a database curation step, not an MS/MS spectral annotation task
- Adduct masses fall outside the 220–640 Da window covered by the database

## Inputs

- Curated DNA adduct database (.xlsx/.docx, 279 entries) from the GitLab repository
- Prior DNA adduct databases (Hemeryck et al. 2015, Carrà et al. 2019a, Guo et al. 2017)
- Structural identifier files (.cdxml, .mol) or SMILES/InChI/InChIKey strings per entry

## Outputs

- Harmonized adduct table with name, formula, monoisotopic mass, database membership, and inclusion/exclusion rationale
- Overlap counts (pairwise/three-way set intersections) reproducing the Venn structure of Figure 3A
- List of 92 additional adducts and 54 excluded adducts

## How to apply

Load the curated database (.xlsx/.docx from the GitLab repository, 279 literature-reported entries) alongside prior databases and map alternate adduct names using the paper's systematic naming rules. Match entries on molecular formula and monoisotopic mass within the 220–640 Da range that the database covers, as reported in the harmonization step. Compute set operations on the harmonized identifiers to quantify overlap (e.g. 92 additional adducts beyond three prior databases, 54 excluded for insufficient identification confidence). Use ChemDraw to generate/verify canonical structural identifiers (.cdxml, .mol, SMILES, InChI, InChIKey, IUPAC names) so each matched entry carries unambiguous chemical metadata. Validation: the harmonized counts must reproduce the paper's reported totals; flag discrepancies beyond a single-adduct difference for expert review.

## Related tools

- **ChemDraw** (Generate/verify canonical structural identifiers and molecular formulas for harmonized adduct entries)

## Evaluation signals

- Computed totals match the paper's reported values: 279 current entries, 121/122/102 prior entries, 92 additional, 54 excluded
- Every matched pair agrees on molecular formula AND monoisotopic mass within the 220–640 Da window
- Alternate-name mappings (e.g. Cro-1I-dG ↔ 1,N2-(Me-OH-P)-dG) resolve to the same formula/mass
- Inclusion/exclusion rationale recorded for every adduct not present in all databases
- No discrepancy beyond a single adduct between computed and published overlap counts

## Limitations

- Matching relies on formula + monoisotopic mass, so isobaric adducts (same formula, different structure) can be conflated without structural identifier checks
- The 220–640 Da range excludes smaller/larger adducts from the harmonization scope
- Prior databases (Hemeryck, Carrà, Guo) use naming conventions that must be mapped manually before set operations are meaningful

## Evidence

- [abstract] harmonized structures, molecular formulas, and names: "The second step consisted in harmonizing structures, molecular formulas, and names"
- [abstract] database scope and identifiers: "monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature"
- [abstract] entry counts: "for building a systematic database of 279 DNA adducts"
- [supplementary] mass window from source card: [evidence span withheld — anchor status: not_found]
