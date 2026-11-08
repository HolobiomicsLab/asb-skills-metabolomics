---
name: chemical-element-frequency-filtering
description: Use when curating McLean CCS Compendium training data for [M-H]-, [M+H]+
  or [M+Na]+ adducts and entries containing rare chemical elements (fewer than five
  occurrences in the database) must be excluded before model training.
when_to_use_negative:
- When the goal is filtering metabolite annotation candidates by predicted CCS (ΔCCS
  threshold of 2.8% and 10 ppm mass error) — this skill is a training-set curation
  step, not an annotation filter
- When duplicates and m/z outliers have not yet been removed — the element-frequency
  filter is designed to run after duplicate averaging and the 1 mDa m/z deviation
  check
- When the target adduct or molecule class is not represented in the McLean Compendium
  sheets this filter was designed for (e.g. dimers such as [2M+H]+/[2M-H]-, other
  less common adducts, or in-source fragments, which the current CCSP 2.0 training
  sets do not support)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: CCSP 2.0
  role: Open-source Jupyter notebook pipeline in which the element-frequency filter
    is one of the McLean Compendium scrubbing steps applied before SVR training
  repo: https://github.com/facundof2016/CCSP2.0
- name: Pandas
  role: Used to load each adduct spreadsheet, average duplicate entries, and apply
    the element-frequency exclusion rule to the DataFrame
- name: Python
  role: Implementation language for the scrubbing and filtering steps in the CCSP
    2.0 notebook
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:a4d992a378c4efdb8b6e103212e84538ae5dcc7457e3691ba62abdfe64e6092d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - AllCCS
  - CCSP 2.0
  - CCSbase
  - DeepCCS
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# chemical-element-frequency-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A data-curation filter from the CCSP 2.0 pipeline that removes McLean Unified CCS Compendium entries containing any chemical element occurring in fewer than five database entries, so that linear support vector regression models are not trained on elements with too few examples.

## When to use

Use when curating McLean CCS Compendium training data for [M-H]-, [M+H]+ or [M+Na]+ adducts and entries containing rare chemical elements (fewer than five occurrences in the database) must be excluded before model training.

## When NOT to use

- When the goal is filtering metabolite annotation candidates by predicted CCS (ΔCCS threshold of 2.8% and 10 ppm mass error) — this skill is a training-set curation step, not an annotation filter
- When duplicates and m/z outliers have not yet been removed — the element-frequency filter is designed to run after duplicate averaging and the 1 mDa m/z deviation check
- When the target adduct or molecule class is not represented in the McLean Compendium sheets this filter was designed for (e.g. dimers such as [2M+H]+/[2M-H]-, other less common adducts, or in-source fragments, which the current CCSP 2.0 training sets do not support)

## Inputs

- McLean Unified CCS Compendium adduct spreadsheets ([M-H]-, [M+H]+ and [M+Na]+ sheets, downloaded separately)
- Per-entry chemical formula or neutral InChI string from which element symbols are parsed
- Partially cleaned pandas DataFrame (duplicates already averaged, m/z outliers already removed)

## Outputs

- Adduct-specific cleaned entry table in which every remaining chemical element appears in at least five database entries
- Final cleaned CSV files yielding 567 singly deprotonated, 518 singly protonated and 461 singly sodiated ions

## How to apply

Download the complete [M-H]-, [M+H]+ and [M+Na]+ data sets from the McLean Unified CCS Compendium as separate spreadsheets, and apply this filter to each adduct sheet independently after replacing duplicate CCS entries with their average and removing entries whose m/z (computed from the provided InChI and ionic species) deviates more than 1 mDa from the theoretical value. Parse the chemical elements present in each entry's formula/InChI, count how many database entries contain each element within that adduct sheet, and drop any entry containing an element with a count below five. The rationale is that elements appearing in fewer than five entries provide insufficient training examples for reliable descriptor-based CCS prediction with linear support vector regression. After this filter, further exclude entries incompatible with the mass ranges used by DeepCCS, AllCCS, or CCSBase, and write the cleaned tables as CSVs.

## Related tools

- **CCSP 2.0** (Open-source Jupyter notebook pipeline in which the element-frequency filter is one of the McLean Compendium scrubbing steps applied before SVR training) — https://github.com/facundof2016/CCSP2.0
- **Pandas** (Used to load each adduct spreadsheet, average duplicate entries, and apply the element-frequency exclusion rule to the DataFrame)
- **Python** (Implementation language for the scrubbing and filtering steps in the CCSP 2.0 notebook)

## Examples

```
el_counts = df['formula'].str.findall(r'[A-Z][a-z]?').explode().value_counts(); df = df[df['formula'].str.findall(r'[A-Z][a-z]?').apply(lambda els: all(el_counts[e] >= 5 for e in els))]
```

## Evaluation signals

- After all scrubbing steps (duplicate averaging, 1 mDa m/z filter, element-frequency filter, mass-range compatibility check), cleaned tables must match the reported counts of 567 [M-H]-, 518 [M+H]+ and 461 [M+Na]+ ions
- No chemical element remaining in any cleaned adduct sheet occurs in fewer than five database entries (verify via a value_counts audit over parsed element symbols)
- The filter is applied per adduct sheet ([M-H]-, [M+H]+, [M+Na]+ processed separately), not on the merged table
- Entries removed are traceable: each dropped row should contain at least one element with count < 5 in that sheet
- The three cleaned CSVs load successfully into the CCSP 2.0 notebook for subsequent RDKit/Mordred descriptor calculation

## Limitations

- The five-entry cutoff is an empirical choice tied to the McLean Compendium corpus size; re-applying it to other CCS databases without recalibration may over- or under-filter
- Removing rare-element entries restricts the final model to chemistries with sufficient representation, so the trained CCSP 2.0 models will not predict CCS for compounds containing filtered-out elements
- This filter alone does not guarantee a clean training set — it must be combined with the 1 mDa m/z deviation check (which typically catches isotopically-labelled compound misrepresentation) and the DeepCCS/AllCCS/CCSBase mass-range compatibility check to reach the reported 567/518/461 counts

## Evidence

- [full_text] Element-frequency exclusion rule as defined in the CCSP 2.0 scrubbing workflow: [evidence span withheld — anchor status: not_found]
- [full_text] Per-adduct source data for the filter: [evidence span withheld — anchor status: not_found]
- [full_text] Filter is one step among ordered scrubbing steps including the 1 mDa m/z check: [evidence span withheld — anchor status: not_found]
- [readme] Tool in which the filtering skill is packaged: [evidence span withheld — anchor status: external_doc]
