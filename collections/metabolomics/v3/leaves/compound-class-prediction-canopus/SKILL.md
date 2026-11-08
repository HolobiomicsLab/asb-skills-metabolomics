---
name: compound-class-prediction-canopus
description: Use when MS/MS spectra (e.g. peak lists from a microbeMASST search) lack
  confident library matches but need putative class-level annotation, and spectra
  have parent mass < 850 Da.
when_to_use_negative:
- Spectra already have confident GNPS library matches at MSI level 1 or already carry
  validated structural annotations
- Spectra have parent mass >= 850 Da, outside the mass range used for SIRIUS/CANOPUS
  prediction in this workflow
- The goal is taxonomic attribution of spectra rather than chemical-class annotation
  — use the microbeMASST _matches.tsv / _count_domain.tsv outputs instead
edam_operation: http://edamontology.org/operation_0394
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: SIRIUS
  role: Predicts candidate molecular formulas and fragment trees for each filtered
    MS/MS spectrum
- name: CANOPUS
  role: Assigns compound classes from fragment spectra using the SIRIUS prediction
    results
- name: microbeMASST
  role: Supplies the putative microbial spectra peak lists and the GNPS-library _library.tsv
    matches that are joined to CANOPUS class predictions for level-2 MSI annotation
  repo: https://masst.gnps2.org/microbemasst/
- name: Python 3.10
  role: Required runtime for running the jobs.py batch search that generates the annotation
    inputs
- name: MZmine
  role: Generates the single .mgf peak-list file used as batch-search input
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:5f14fe97a243f2b1f8441099831f6ed83d3287757d3afb9907c03871f965a94e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CANOPUS
  - MZmine
  - Python 3.10
  - SIRIUS
  - microbeMASST
  edam_operation: http://edamontology.org/operation_0394
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# compound-class-prediction-canopus

## Summary

Predicts putative chemical classes for MS/MS spectra by running SIRIUS for formula/fragment-tree prediction followed by CANOPUS compound-class prediction from fragment spectra, then joining class predictions with GNPS library hits to assign level-2 Metabolomics Standards Initiative (MSI) annotations.

## When to use

Use when MS/MS spectra (e.g. peak lists from a microbeMASST search) lack confident library matches but need putative class-level annotation, and spectra have parent mass < 850 Da.

## When NOT to use

- Spectra already have confident GNPS library matches at MSI level 1 or already carry validated structural annotations
- Spectra have parent mass >= 850 Da, outside the mass range used for SIRIUS/CANOPUS prediction in this workflow
- The goal is taxonomic attribution of spectra rather than chemical-class annotation — use the microbeMASST _matches.tsv / _count_domain.tsv outputs instead

## Inputs

- MS/MS spectra peak lists from the microbeMASST search output (single .mgf file generated via MZmine, or a .csv/.tsv list of USIs)
- _library.tsv output from the batch MASST search containing GNPS library spectra matches
- Parent-mass filter threshold (< 850 Da)

## Outputs

- Per-spectrum TSV with one row per spectrum: parent mass, predicted formula, CANOPUS class, and MSI annotation level
- Level-2 MSI annotation rate (fraction of spectra annotated, e.g. 23% of 512 antibiotic-sensitive spectra)
- Aggregated class assignments distinguishing phylum-specific vs phylum-wide classes (terpenoid vs amino acid/peptide spectra)

## How to apply

Load the MS/MS peak lists (e.g. the 2,425 putative microbial spectra from the microbeMASST search output) and filter to entries with parent mass < 850 Da. Run SIRIUS on each filtered spectrum to predict candidate molecular formulas and fragment trees, then run CANOPUS on the SIRIUS results to assign compound classes directly from fragment spectra. Join the SIRIUS/CANOPUS class predictions with the GNPS library hits found in the _library.tsv output of the batch MASST search to assign level-2 MSI annotations (e.g. amidated microbial bile acids, free and keto bile acids, N-acyl-lipids, di- and tripeptides, soyasapogenol). Write a per-spectrum TSV with columns for parent mass, formula prediction, CANOPUS class, and MSI annotation level, then aggregate annotation status to compute the level-2 annotation rate among spectra of interest (e.g. the 512 antibiotic-sensitive microbial spectra). Run the underlying batch search under Python 3.10 via jobs.py against the Fast Search API.

## Related tools

- **SIRIUS** (Predicts candidate molecular formulas and fragment trees for each filtered MS/MS spectrum)
- **CANOPUS** (Assigns compound classes from fragment spectra using the SIRIUS prediction results)
- **microbeMASST** (Supplies the putative microbial spectra peak lists and the GNPS-library _library.tsv matches that are joined to CANOPUS class predictions for level-2 MSI annotation) — https://masst.gnps2.org/microbemasst/
- **Python 3.10** (Required runtime for running the jobs.py batch search that generates the annotation inputs)
- **MZmine** (Generates the single .mgf peak-list file used as batch-search input) — https://github.com/mzmine/mzmine

## Evaluation signals

- Every spectrum in the output TSV has one row with non-empty parent mass, formula prediction, CANOPUS class, and MSI annotation level columns
- All spectra in the annotation table pass the parent mass < 850 Da filter applied before SIRIUS/CANOPUS
- Level-2 MSI annotations are only assigned where a GNPS library hit exists in _library.tsv; the computed annotation rate (e.g. 23% of 512 antibiotic-sensitive spectra) is reproducible on re-run
- Batch search was re-run with skip_existing=True until no new output was generated, so no spectra are missing from the annotation table due to transient Fast Search API failures

## Limitations

- SIRIUS/CANOPUS predictions in this workflow are restricted to spectra with parent mass < 850 Da, limiting coverage to smaller metabolites
- Annotations are putative only (MSI level 2 via GNPS library matches), not validated structures
- The Fast Search API can silently fail on some entries, requiring repeated jobs.py runs with skip_existing=True until no new output appears

## Evidence

- [methods] Batch search scope: "batch search of multiple MS/MS spectra against the current indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN"
- [readme] Library output enables MSI level 2: [evidence span withheld — anchor status: not_found]
- [readme] Python version requirement: "Please make user to use **_Python 3.10_**"
- [readme] Input format: "You can run either a single .mgf file generated via [MZmine](https://github.com/mzmine/mzmine), from the molecular networking in GNPS workflow, or a list of [USIs]"
