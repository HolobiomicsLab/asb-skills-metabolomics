---
name: insilico-formula-annotation-sirius
description: Use when MS/MS peak lists (e.g., from a microbeMASST batch search) lack
  library matches and you need tentative molecular formula and compound-class assignments;
  applied to spectra with parent mass < 850 Da.
when_to_use_negative:
- Spectra with parent mass >= 850 Da, for which SIRIUS/CANOPUS class prediction was
  not applied in the reference workflow
- Spectra already annotated at higher confidence (e.g., MSI level 1) by direct GNPS
  library matching — this skill only yields tentative level-2 annotations
- Raw LC-MS/MS data that still requires peak picking/feature detection (first process
  with MZmine to generate the .mgf)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: Predicts candidate molecular formulas and fragment trees for each filtered
    MS/MS spectrum
- name: CANOPUS
  role: Assigns putative compound classes from the SIRIUS fragment spectra
- name: microbeMASST
  role: Batch-searches the spectra against microbial reference data; supplies the
    spectra, matches, and _library.tsv GNPS library hits used for the level-2 MSI
    join
  repo: https://github.com/robinschmid/microbe_masst
- name: Fast Search API
  role: Backend used by jobs.py to batch-search multiple MS/MS spectra against indexed
    GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data
  repo: https://fasst.gnps2.org/fastsearch/
- name: MZmine
  role: Generates the single .mgf peak-list file used as input to the batch search
  repo: https://github.com/mzmine/mzmine
- name: Python 3.10
  role: Required runtime for the microbeMASST batch jobs.py and for building the annotation/class
    TSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:7dd261d22d521cfe7345fd8c7d5eafa20652f94d7f2a03ab2a7e1a90c77b29da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - CANOPUS
  - Fast Search API
  - MZmine
  - Python 3.10
  - SIRIUS
  - microbeMASST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# insilico-formula-annotation-sirius

## Summary

In-silico annotation of microbial MS/MS spectra by predicting molecular formulas and fragment trees with SIRIUS and compound classes with CANOPUS, then joining predictions with GNPS library hits to assign level-2 Metabolomics Standards Initiative (MSI) annotations.

## When to use

Use when MS/MS peak lists (e.g., from a microbeMASST batch search) lack library matches and you need tentative molecular formula and compound-class assignments; applied to spectra with parent mass < 850 Da.

## When NOT to use

- Spectra with parent mass >= 850 Da, for which SIRIUS/CANOPUS class prediction was not applied in the reference workflow
- Spectra already annotated at higher confidence (e.g., MSI level 1) by direct GNPS library matching — this skill only yields tentative level-2 annotations
- Raw LC-MS/MS data that still requires peak picking/feature detection (first process with MZmine to generate the .mgf)

## Inputs

- MS/MS spectrum peak lists (e.g., .mgf file generated via MZmine, or USI list as .csv/.tsv) from a microbeMASST search
- _library.tsv output from the microbeMASST batch search (GNPS library matches)
- Parent mass per spectrum (filter: < 850 Da)

## Outputs

- Per-spectrum annotation/class TSV with columns: parent mass, predicted formula, CANOPUS class, MSI annotation level
- Level-2 MSI annotation rate among the antibiotic-sensitive spectra (23% of 512 in the reference study)

## How to apply

Load the peak lists of the putative microbial spectra (e.g., the 2,425 spectra from the microbeMASST search output) and filter to entries with parent mass < 850 Da, since CANOPUS class prediction is restricted to that mass range. Run SIRIUS on each filtered spectrum to predict candidate molecular formulas and fragment trees, then run CANOPUS on those results to assign compound classes from the fragment spectra. Join the SIRIUS/CANOPUS class predictions with the GNPS library hits from the _library.tsv output of the batch MASST search to assign level-2 (2007 Metabolomics Standards Initiative) putative annotations. Aggregate per-spectrum annotation status and compute the level-2 MSI annotation rate among the target set (here, the 512 antibiotic-sensitive microbial spectra). Write the annotation/class table as a TSV with one row per spectrum, with columns for parent mass, predicted formula, CANOPUS class, and MSI annotation level; use Python 3.10 as required by the microbeMASST batch tooling.

## Related tools

- **SIRIUS** (Predicts candidate molecular formulas and fragment trees for each filtered MS/MS spectrum)
- **CANOPUS** (Assigns putative compound classes from the SIRIUS fragment spectra)
- **microbeMASST** (Batch-searches the spectra against microbial reference data; supplies the spectra, matches, and _library.tsv GNPS library hits used for the level-2 MSI join) — https://github.com/robinschmid/microbe_masst
- **Fast Search API** (Backend used by jobs.py to batch-search multiple MS/MS spectra against indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data) — https://fasst.gnps2.org/fastsearch/
- **MZmine** (Generates the single .mgf peak-list file used as input to the batch search) — https://github.com/mzmine/mzmine
- **Python 3.10** (Required runtime for the microbeMASST batch jobs.py and for building the annotation/class TSV)

## Evaluation signals

- Every spectrum with parent mass < 850 Da carries a CANOPUS class assignment; spectra >= 850 Da should be excluded from class prediction
- Reproduced level-2 MSI annotation rate is in the ballpark of the reference result (23% of the 512 antibiotic-sensitive microbial spectra annotated via GNPS library matching)
- Output TSV has one row per spectrum with non-empty parent mass, formula prediction, CANOPUS class, and MSI annotation level columns
- Annotated classes match expected biology: amidated microbial bile acids, free and keto bile acids, N-acyl-lipids, di- and tripeptides, soyasapogenol; terpenoid-classified spectra observed across phyla while amino acid/peptide spectra are more phylum-specific
- _library.tsv was regenerated by re-running jobs.py with skip_existing=True until no new output appeared, ensuring the library join used complete match data

## Limitations

- CANOPUS class prediction is limited to spectra with parent mass < 850 Da in this workflow
- Annotations are tentative (level-2 MSI), based on library matching and in-silico prediction rather than reference-standard confirmation
- The Fast Search API can fail on some entries during batch runs; jobs.py must be re-run a couple of times with skip_existing=True until no new output is generated
- Python 3.10 is required; other versions may break the batch tooling

## Evidence

- [other] level-2 MSI via GNPS library: [evidence span withheld — anchor status: not_found]
- [other] SIRIUS step: [evidence span withheld — anchor status: not_found]
- [readme] library.tsv enables MSI level 2: "This enables a Level 2 annotation according the Metabolomics Standards Initiative."
- [methods] batch search scope: "execute a batch search of multiple MS/MS spectra against the current indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN"
- [readme] python version requirement: "Please make user to use **_Python 3.10_**"
