---
name: nmr-chemical-shift-prediction
description: Use when you need predicted 1H NMR shifts across a compound structure
  library (e.g., SMILES/SDF from the Natural Products Atlas) to assess dereplication
  coverage or classify shielded vs. deshielded resonances.
when_to_use_negative:
- When accurate structure dereplication against reference spectra is required — the
  authors state 'lightweight' NMR predictors do not have sufficient accuracy for that
  task
- When compounds belong to specialist classes (sterols, low MW isoprenoids, lipids,
  volatile terpenes) whose exclusively shielded 1H signals fall outside the analysis
  scope
tools:
- name: NMRShiftDB2
  role: Source of the stereo-aware HOSE code model used to predict 1H chemical shifts
    for every structure in the compound set
- name: MADByTE
  role: Downstream NMR mixture-analysis platform for which the predicted-signal statistics
    define the dereplicatable compound scope
  repo: https://github.com/liningtonlab/MADByTE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:0df60d8b280ea9bf54c0993a7474eff2dbc26dde4be062fef2787d9a750aef9a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - MADByTE
  - NMRShiftDB2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# nmr-chemical-shift-prediction

## Summary

Predict in silico 1H NMR chemical shifts for a large natural product structure set using the stereo-aware HOSE code model from NMRShiftDB2, enabling shielded/deshielded signal statistics (e.g., 94.9% of 27,844 Natural Products Atlas microbial NPs possess at least 10% deshielded signals; 0.4% exclusively shielded).

## When to use

Use when you need predicted 1H NMR shifts across a compound structure library (e.g., SMILES/SDF from the Natural Products Atlas) to assess dereplication coverage or classify shielded vs. deshielded resonances.

## When NOT to use

- When accurate structure dereplication against reference spectra is required — the authors state 'lightweight' NMR predictors do not have sufficient accuracy for that task
- When compounds belong to specialist classes (sterols, low MW isoprenoids, lipids, volatile terpenes) whose exclusively shielded 1H signals fall outside the analysis scope

## Inputs

- Compound structure set (27,844 microbial natural products from the Natural Products Atlas database)
- Stereo-aware HOSE code 1H NMR prediction model from NMRShiftDB2

## Outputs

- Per-molecule predicted 1H chemical shift lists
- Per-molecule fraction of deshielded signals with shielded/deshielded classification
- Summary statistics CSV reporting percentage with exclusively shielded signals (0.4%) and percentage with at least 10% deshielded signals (94.9%)

## How to apply

Load the compound structure set (e.g., the 27,844 microbial natural products from the Natural Products Atlas database) and apply the stereo-aware HOSE code model available from NMRShiftDB2 to predict 1H chemical shifts for every structure. For each molecule, classify predicted 1H signals as shielded or deshielded and compute the fraction of deshielded signals per compound. Tally the compounds with at least 10% deshielded signals and those with exclusively shielded signals, then write the two percentages to a summary statistics CSV. Rationale: this lightweight predictor is fast enough for library-scale annotation (as done in the MADByTE workflow), but the authors caution it lacks the accuracy needed for full spectral dereplication against reference libraries.

## Related tools

- **NMRShiftDB2** (Source of the stereo-aware HOSE code model used to predict 1H chemical shifts for every structure in the compound set)
- **MADByTE** (Downstream NMR mixture-analysis platform for which the predicted-signal statistics define the dereplicatable compound scope) — https://github.com/liningtonlab/MADByTE

## Evaluation signals

- Percentage of molecules with at least 10% deshielded 1H signals must reproduce the reported value of 94.9%
- Percentage of molecules with exclusively shielded 1H signals must reproduce the reported value of 0.4%
- Total molecule count processed must equal the input set size (27,844 microbial natural products)
- Every structure must yield a complete predicted 1H shift list before shielded/deshielded classification is tallied
- Summary CSV must contain exactly the two percentage metrics (deshielded ≥10% and exclusively shielded)

## Limitations

- Current 'lightweight' NMR predictors do not have sufficient accuracy to support direct dereplication against large-scale NMR repositories
- Compound classes with exclusively shielded 1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes) are effectively precluded from downstream MADByTE analysis and require user-supplied reference spectra

## Evidence
