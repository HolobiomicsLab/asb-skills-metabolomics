---
name: compendium-duplicate-averaging
description: Use when preparing an ion-mobility CCS training set from McLean Unified
  CCS Compendium adduct sheets where duplicate entries with differing measured CCS
  values exist and must be collapsed before further scrubbing or model fitting.
when_to_use_negative:
- Entries are already unique (no duplicated compound/adduct pairs report multiple
  CCS values) — averaging would silently drop real replicate values without a duplicate-detection
  need.
- The goal is benchmarking prediction accuracy on the 463-measurement McLean test
  set rather than building a training set; test-set composition should not be altered
  by duplicate averaging.
- Training on user-supplied custom CCS data that is not drawn from the McLean Compendium;
  the duplicate structure and averaging rationale are specific to compendium entry
  provenance.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: pandas
  role: Used to load the McLean Compendium adduct spreadsheets and average CCS values
    across duplicate entries before deduplication-aware scrubbing steps
- name: CCSP 2.0
  role: The downstream Jupyter-notebook pipeline into which the cleaned per-adduct
    CSVs feed; this skill reproduces its documented McLean-data scrubbing step
  repo: https://github.com/facundof2016/CCSP2.0
- name: Python
  role: Execution environment (Python 3) in which the pandas-based deduplication/averaging
    is performed as part of the CCSP 2.0 notebook workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:a1eb3e791b47ecab25b7706810b5b79df3890b35ea129ca52a73378249fdf917
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
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# compendium-duplicate-averaging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A data-cleaning step for CCS model training sets that collapses duplicate ion entries in McLean Unified CCS Compendium spreadsheets by averaging their measured CCS values, using pandas, as part of building the CCSP 2.0 [M-H]-, [M+H]+ and [M+Na]+ training sets.

## When to use

Use when preparing an ion-mobility CCS training set from McLean Unified CCS Compendium adduct sheets where duplicate entries with differing measured CCS values exist and must be collapsed before further scrubbing or model fitting.

## When NOT to use

- Entries are already unique (no duplicated compound/adduct pairs report multiple CCS values) — averaging would silently drop real replicate values without a duplicate-detection need.
- The goal is benchmarking prediction accuracy on the 463-measurement McLean test set rather than building a training set; test-set composition should not be altered by duplicate averaging.
- Training on user-supplied custom CCS data that is not drawn from the McLean Compendium; the duplicate structure and averaging rationale are specific to compendium entry provenance.

## Inputs

- McLean Unified CCS Compendium spreadsheets for [M-H]-, [M+H]+ and [M+Na]+ adducts, containing compound identifiers, measured CCS values, m/z and adduct species per entry

## Outputs

- Cleaned per-adduct CSV tables with one row per unique compound/adduct pair, duplicate CCS entries replaced by their averaged CCS value

## How to apply

Download the complete [M-H]-, [M+H]+ and [M+Na]+ data sets from the McLean Unified CCS Compendium as separate spreadsheets, one per adduct type. Load each adduct sheet into pandas and identify entries that duplicate the same compound/adduct pair but report different measured CCS values. Replace each duplicate group with a single entry whose CCS value is the average of the provided CCS values; this preserves the consensus of independent drift tube measurements while removing rows that would double-weight the same ion during linear support vector regression training. Keep the averaged entries together with their m/z and adduct fields so subsequent scrubbing steps (e.g. 1 mDa theoretical-vs-listed m/z check) can operate on the merged rows. Write the cleaned adduct tables to CSV for the remaining scrubbing and validation steps.

## Related tools

- **pandas** (Used to load the McLean Compendium adduct spreadsheets and average CCS values across duplicate entries before deduplication-aware scrubbing steps)
- **CCSP 2.0** (The downstream Jupyter-notebook pipeline into which the cleaned per-adduct CSVs feed; this skill reproduces its documented McLean-data scrubbing step) — https://github.com/facundof2016/CCSP2.0
- **Python** (Execution environment (Python 3) in which the pandas-based deduplication/averaging is performed as part of the CCSP 2.0 notebook workflow)

## Examples

```
df.groupby(['compound_id','adduct'])['ccs'].mean().reset_index()
```

## Evaluation signals

- Averaged CCS value equals the arithmetic mean of the original duplicate group's CCS values; spot-check a few duplicate groups against the raw compendium sheet.
- No compound/adduct pair appears more than once in the cleaned per-adduct table.
- Downstream scrubbing still yields the paper's reported counts (567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions) after m/z and element-frequency filters are applied.
- Cleaned tables retain m/z and adduct columns so the subsequent 1 mDa m/z deviation filter can run without re-joining data.

## Limitations

- The article does not state whether averaging should use the mean vs. median across duplicate CCS values; arithmetic mean is the implied convention, and edge cases with >2 duplicates are not discussed.
- Duplicate averaging must precede the 1 mDa theoretical-vs-listed m/z filter; applying it after other scrubbing steps could silently drop entries from the paper's reported final counts.
- The scrubbing workflow is specific to the McLean Unified CCS Compendium structure; other CCS databases may define duplicates differently.
- Count validation against the paper's 567/518/461 post-scrubbing totals depends on all subsequent scrubbing steps (element-frequency, mass-range) being reproduced identically.

## Evidence

- [full_text] Duplicate averaging drives final training-set size: [evidence span withheld — anchor status: not_found]
- [full_text] Duplicate collapse defined precisely: [evidence span withheld — anchor status: not_found]
- [full_text] Counts only reproducible after averaging: [evidence span withheld — anchor status: not_found]
- [full_text] m/z filter applied after averaging: [evidence span withheld — anchor status: not_found]
- [readme] Tooling requirement for the notebook: [evidence span withheld — anchor status: external_doc]
