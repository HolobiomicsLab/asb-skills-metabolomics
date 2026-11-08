---
name: natural-products-structure-dataset-processing
description: Use when you need predicted 1H chemical shifts for a batch of known natural
  product structures (e.g. Natural Products Atlas SMILES/MOL files) to compute shielded/deshielded
  signal statistics or build an in silico reference spectral library.
when_to_use_negative:
- When only experimental NMR spectra (raw FID/processed 1D or 2D data) are available
  and no structures need in silico prediction
- When the goal is dereplicating compound classes with exclusively shielded 1H signals
  (sterols, low MW isoprenoids, lipids, volatile terpenes), which are precluded from
  analysis by design
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3314
tools:
- name: NMRShiftDB2
  role: Provides the stereo-aware HOSE code model used to predict 1H chemical shifts
    for each natural product structure
- name: MADByTE
  role: Downstream platform whose dereplication scope motivates the shielded/deshielded
    coverage statistics
  repo: https://github.com/liningtonlab/MADByTE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:7534d8d1274c79d32bfb8084854141c3609e1ebc99f7c5e63d5fd9f41d894bcb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - MADByTE
  - NMRShiftDB2
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# natural-products-structure-dataset-processing

## Summary

Predicts in silico 1H NMR spectra for a large natural product structure set (e.g. 27,844 microbial NPAtlas compounds) using the stereo-aware HOSE code model from NMRShiftDB2, then tallies shielded vs deshielded signal fractions to characterize the chemical-shift coverage of a dereplication library.

## When to use

Use when you need predicted 1H chemical shifts for a batch of known natural product structures (e.g. Natural Products Atlas SMILES/MOL files) to compute shielded/deshielded signal statistics or build an in silico reference spectral library.

## When NOT to use

- When only experimental NMR spectra (raw FID/processed 1D or 2D data) are available and no structures need in silico prediction
- When the goal is dereplicating compound classes with exclusively shielded 1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes), which are precluded from analysis by design

## Inputs

- Structure set of 27,844 microbial natural products from the Natural Products Atlas database
- Stereo-aware HOSE code predictor model available from NMRShiftDB2

## Outputs

- Predicted 1H chemical shifts for every structure in the set
- Per-molecule fraction of deshielded 1H signals
- Summary statistics CSV with the proportion of molecules exclusively shielded and the proportion with >=10% deshielded signals (expected 0.4% and 94.9%)

## How to apply

Load the compound structure set (the article used 27,844 microbial natural products from the Natural Products Atlas database), then apply the stereo-aware HOSE code model available from NMRShiftDB2 to predict 1H chemical shifts for every structure. For each molecule, classify its predicted 1H signals as shielded or deshielded and compute the fraction of deshielded signals. Tally molecules with at least 10% deshielded signals separately from molecules containing exclusively shielded signals, and write both percentages to a summary CSV. The rationale is that a dereplication platform like MADByTE excludes the shielded quadrant of the TOCSY spectrum (<2.5 ppm), so quantifying deshielded-signal coverage establishes which compound classes the platform can and cannot dereplicate.

## Related tools

- **NMRShiftDB2** (Provides the stereo-aware HOSE code model used to predict 1H chemical shifts for each natural product structure)
- **MADByTE** (Downstream platform whose dereplication scope motivates the shielded/deshielded coverage statistics) — https://github.com/liningtonlab/MADByTE

## Examples

```
python predict_nmr_stats.py --structures npatlas_microbial_27844.sdf --predictor nmsshiftdb2_hose --min_deshielded_frac 0.10 --out summary_stats.csv
```

## Evaluation signals

- Computed proportion of molecules with exclusively shielded 1H signals must equal 0.4%
- Computed proportion of molecules possessing at least 10% deshielded signals must equal 94.9%
- Denominator of both tallies must be the full 27,844-compound Natural Products Atlas set
- Per-molecule deshielded fractions across all molecules sum into the two reported categories

## Limitations

- Specialist compound classes with exclusively shielded 1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes) are precluded from downstream MADByTE dereplication
- Current 'lightweight' NMR predictors do not have sufficient accuracy to substitute for large-scale NMR repositories in dereplication

## Evidence
