---
name: metabolite-annotation-table-generation
description: Use when MS/MS spectra (e.g., .mgf peak lists or USI lists from a microbeMASST
  search) need putative metabolite-class annotations and MSI level-2 assignments compiled
  into a single tabular file.
when_to_use_negative:
- Spectra with parent mass >= 850 Da — CANOPUS class prediction was applied only to
  spectra with parent mass < 850 Da
- When a level-1 (standardized reference standard) MSI annotation is required — this
  workflow yields only level-2 putative annotations via library matching
- Input is already a curated feature table with annotations; this skill regenerates
  annotations from raw search outputs
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: microbeMASST
  role: Batch-searches MS/MS spectra against indexed microbial reference data and
    produces _library.tsv with GNPS library matches used for level-2 MSI annotation
  repo: https://github.com/robinschmid/microbe_masst
- name: Fast Search API
  role: Backend API invoked by jobs.py to execute the batch spectrum search against
    GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN
- name: SIRIUS
  role: Predicts candidate molecular formulas and fragment trees for each spectrum
    with parent mass < 850 Da
- name: CANOPUS
  role: Assigns putative compound classes from SIRIUS fragment-tree predictions
- name: Python 3.10
  role: Runtime required for the batch jobs.py workflow and downstream table assembly
- name: MZmine
  role: Generates the single .mgf peak-list file that serves as search input
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:af2e62bda1896fc81b4f91b932406a06897ec704f15d9684957dc6f1055d6ced
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
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metabolite-annotation-table-generation

## Summary

Generates a per-spectrum metabolite annotation table by combining GNPS library matches (level 2 MSI) with SIRIUS formula predictions and CANOPUS compound-class assignments from microbeMASST search outputs.

## When to use

Use when MS/MS spectra (e.g., .mgf peak lists or USI lists from a microbeMASST search) need putative metabolite-class annotations and MSI level-2 assignments compiled into a single tabular file.

## When NOT to use

- Spectra with parent mass >= 850 Da — CANOPUS class prediction was applied only to spectra with parent mass < 850 Da
- When a level-1 (standardized reference standard) MSI annotation is required — this workflow yields only level-2 putative annotations via library matching
- Input is already a curated feature table with annotations; this skill regenerates annotations from raw search outputs

## Inputs

- _library.tsv output from microbeMASST batch search (GNPS library matches)
- _matches.tsv / _count_domain.tsv outputs from microbeMASST batch search
- .mgf peak-list file generated via MZmine, or a .csv/.tsv list of USIs
- 2,425 putative microbial MS/MS spectra (peak lists from microbeMASST search output)

## Outputs

- Per-spectrum annotation/class TSV with columns: parent mass, formula prediction, CANOPUS class, MSI annotation level
- Aggregated level-2 MSI annotation rate over the target spectrum set (e.g., 23% of 512 spectra)

## How to apply

Load the microbeMASST search outputs (_library.tsv from the batch MASST run) and filter spectra to parent mass < 850 Da, since SIRIUS/CANOPUS class prediction is restricted to that range. Run SIRIUS on each filtered spectrum to predict candidate molecular formulas and fragment trees, then run CANOPUS to assign compound classes from the fragment spectra. Join the SIRIUS/CANOPUS class predictions with GNPS library hits from _library.tsv to assign level-2 annotations under the 2007 Metabolomics Standards Initiative. Write a TSV with one row per spectrum and columns for parent mass, formula prediction, CANOPUS class, and MSI annotation level; in the source study this yielded a 23% level-2 annotation rate among 512 antibiotic-sensitive microbial spectra, covering amidated microbial bile acids, free and keto bile acids, N-acyl-lipids, di- and tripeptides, and soyasapogenol.

## Related tools

- **microbeMASST** (Batch-searches MS/MS spectra against indexed microbial reference data and produces _library.tsv with GNPS library matches used for level-2 MSI annotation) — https://github.com/robinschmid/microbe_masst
- **Fast Search API** (Backend API invoked by jobs.py to execute the batch spectrum search against GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN)
- **SIRIUS** (Predicts candidate molecular formulas and fragment trees for each spectrum with parent mass < 850 Da)
- **CANOPUS** (Assigns putative compound classes from SIRIUS fragment-tree predictions)
- **Python 3.10** (Runtime required for the batch jobs.py workflow and downstream table assembly)
- **MZmine** (Generates the single .mgf peak-list file that serves as search input) — https://github.com/mzmine/mzmine

## Examples

```
python code/jobs.py  # batch microbeMASST search; re-run with skip_existing=True until no new output, then join _library.tsv with SIRIUS/CANOPUS results
```

## Evaluation signals

- Annotation table has exactly one row per input spectrum and columns for parent mass, formula prediction, CANOPUS class, and MSI annotation level
- All SIRIUS/CANOPUS-annotated rows have parent mass < 850 Da (the applied filter)
- Re-running jobs.py with skip_existing=True produces no new outputs (all API entries completed)
- Level-2 MSI annotation rate is reproducible; in the reference study ~23% of 512 antibiotic-sensitive spectra received level-2 annotations from GNPS library matches
- Annotated classes fall within reported families (amidated microbial bile acids, free/keto bile acids, N-acyl-lipids, di-/tripeptides, soyasapogenol)

## Limitations

- Some Fast Search API entries fail per run; jobs.py must be re-run repeatedly until no new output is generated
- Only putative level-2 (not confirmed) MSI annotations are achievable via GNPS library matching
- CANOPUS/SIRIUS class prediction limited to spectra with parent mass < 850 Da

## Evidence

- [readme] A _library.tsv file will be generated. This contains a list of spectra from the [GNPS libraries] found to match your spectrum of interest. This enables a Level 2 annotation according the Metabolomics Standards Initiative.: [evidence span withheld — anchor status: not_found]
- [results] Among the 512 antibiotic-sensitive microbial MS/MS spectra, 23% received a level 2 putative annotation under the 2007 Metabolomics Standards Initiative by matching the GNPS reference library: [evidence span withheld — anchor status: not_found]
- [methods] Run SIRIUS on each filtered spectrum to predict candidate molecular formulas and fragment trees.: [evidence span withheld — anchor status: not_found]
- [readme] Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`"
- [methods] Run CANOPUS on the SIRIUS results to assign compound classes from fragment spectra.: [evidence span withheld — anchor status: not_found]
