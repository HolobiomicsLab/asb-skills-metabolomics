---
name: spreadsheet-data-scrubbing
description: Use when raw adduct-level CCS spreadsheets ([M-H]-, [M+H]+, [M+Na]+)
  from the McLean Unified CCS Compendium must be cleaned into training sets for linear
  support vector regression CCS prediction.
when_to_use_negative:
- The input is already a cleaned, model-ready training CSV with post-scrubbing counts
  (567/518/461) matching the published tables
- The target adducts are dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source
  fragments, which the CCSP 2.0 training sets do not support
- The goal is CCS-based filtering of annotation candidates (ΔCCS threshold of 2.8%,
  10 ppm mass error) rather than training-set curation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CCSP 2.0
  role: Downstream consumer of the scrubbed [M-H]-, [M+H]+ and [M+Na]+ training tables;
    its supported mass ranges and SVR models define one exclusion criterion
  repo: https://github.com/facundof2016/CCSP2.0
- name: Pandas
  role: Used to replace duplicate CCS entries with a single entry averaging the provided
    CCS values during scrubbing
- name: Python
  role: Implementation language for the scrubbing script and the CCSP 2.0 Jupyter
    notebook
  repo: https://github.com/facundof2016/CCSP2.0
- name: DeepCCS
  role: Its ML mass-range compatibility is one exclusion criterion applied to compendium
    entries
- name: AllCCS
  role: Its ML mass-range compatibility is one exclusion criterion applied to compendium
    entries
- name: CCSbase
  role: Its ML mass-range compatibility is one exclusion criterion applied to compendium
    entries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:2427854f2b88b6213f1bcc234eba306da846b74a3bad33f2e0266ce92e0e105f
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# spreadsheet-data-scrubbing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curation procedure that scrubs McLean Unified CCS Compendium spreadsheets before CCS model training: averaging duplicate CCS entries and excluding entries with m/z deviations >1 mDa, rare chemical elements, or incompatibility with DeepCCS/AllCCS/CCSbase mass ranges.

## When to use

Use when raw adduct-level CCS spreadsheets ([M-H]-, [M+H]+, [M+Na]+) from the McLean Unified CCS Compendium must be cleaned into training sets for linear support vector regression CCS prediction.

## When NOT to use

- The input is already a cleaned, model-ready training CSV with post-scrubbing counts (567/518/461) matching the published tables
- The target adducts are dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments, which the CCSP 2.0 training sets do not support
- The goal is CCS-based filtering of annotation candidates (ΔCCS threshold of 2.8%, 10 ppm mass error) rather than training-set curation

## Inputs

- McLean Unified CCS Compendium adduct spreadsheets for [M-H]-, [M+H]+ and [M+Na]+ (CCS values, theoretical m/z, InChI strings)
- Adduct species specification per sheet (ionic species used to recompute m/z from InChI)

## Outputs

- Three cleaned CSV files: 567 [M-H]-, 518 [M+H]+ and 461 [M+Na]+ ions
- Deduplicated and filtered per-adduct tables ready as CCSP 2.0 training sets

## How to apply

Download the complete [M-H]-, [M+H]+ and [M+Na]+ data sets from the McLean Unified CCS Compendium as separate spreadsheets, then use pandas to replace duplicate entries with a single entry averaging the provided CCS values. Exclude entries whose m/z computed from the provided InChI and ionic species deviates from the listed theoretical m/z by more than 1 mDa (typically isotopically-labelled compound misrepresentation). Also drop entries containing any chemical element that appears in fewer than five database entries, and entries not compatible with the mass ranges set in the ML algorithms used by DeepCCS, AllCCS or CCSbase. The rationale is to remove duplicate, mislabelled, and poorly represented chemistry that would degrade the linear SVR models in CCSP 2.0. Validation: confirm the cleaned tables match the reported post-scrubbing counts of 567 singly deprotonated, 518 singly protonated and 461 singly sodiated ions, and write the three cleaned CSVs.

## Related tools

- **CCSP 2.0** (Downstream consumer of the scrubbed [M-H]-, [M+H]+ and [M+Na]+ training tables; its supported mass ranges and SVR models define one exclusion criterion) — https://github.com/facundof2016/CCSP2.0
- **Pandas** (Used to replace duplicate CCS entries with a single entry averaging the provided CCS values during scrubbing)
- **Python** (Implementation language for the scrubbing script and the CCSP 2.0 Jupyter notebook) — https://github.com/facundof2016/CCSP2.0
- **DeepCCS** (Its ML mass-range compatibility is one exclusion criterion applied to compendium entries)
- **AllCCS** (Its ML mass-range compatibility is one exclusion criterion applied to compendium entries)
- **CCSbase** (Its ML mass-range compatibility is one exclusion criterion applied to compendium entries)

## Examples

```
import pandas as pd; dfs = {a: pd.read_excel(f'mclean_{a}.xlsx') for a in ['M-H','M+H','M+Na']}; [dedupe_and_filter(d) for d in dfs]; [df.to_csv(f'{a}_cleaned.csv') for a, df in dfs.items()]
```

## Evaluation signals

- Cleaned [M-H]- table contains exactly 567 singly deprotonated ions
- Cleaned [M+H]+ table contains exactly 518 singly protonated ions
- Cleaned [M+Na]+ table contains exactly 461 singly sodiated ions
- No remaining entry has an InChI-derived m/z deviating from the listed theoretical m/z by more than 1 mDa
- Every remaining entry's chemical elements each appear in at least five database entries and all masses fall within DeepCCS/AllCCS/CCSbase-compatible ranges

## Limitations

- Entries removed include isotopically-labelled compound misrepresentations, which are detected only via the 1 mDa m/z deviation test
- Elements appearing in fewer than five database entries are excluded, so rare chemistry is not trainable
- Scrubbed sets cover only singly charged [M-H]-, [M+H]+ and [M+Na]+ adducts; dimers such as [2M+H]+ or [2M-H]-, other less common adducts or in-source fragments are not supported
- The 1 mDa, five-entry element-frequency and mass-range thresholds are tuned to CCSP 2.0/DeepCCS/AllCCS/CCSbase and may not transfer to other models

## Evidence

- [other] replace duplicate CCS entries with their average, and excludes entries with m/z deviating more than 1 mDa from theory: [evidence span withheld — anchor status: not_found]
- [other] reported post-scrubbing counts of 567 singly deprotonated, 518 singly protonated and 461 singly sodiated ions: [evidence span withheld — anchor status: not_found]
- [readme] open source Python notebook intended to help ion-mobility scientists predict collision cross sections with user-curated training sets: [evidence span withheld — anchor status: external_doc]
