---
name: ms-instrument-type-comparison
description: Use when comparing a predictor's per-spectrum accuracy across MS/MS instrument
  types (Q-TOF vs. Orbitrap) or strata such as LogP polarity, precursor mass bins,
  or chemical superclass, to locate where a model underperforms.
when_to_use_negative:
- The input is raw vendor MS/MS data before any model inference has produced per-spectrum
  candidate formulas — run the FIDDLE evaluation pipeline first
- The goal is benchmarking a model against baselines (SIRIUS, BUDDY) on overall accuracy
  rather than dissecting accuracy across molecular strata and instruments
- Spectra come from instrument types or precursor types outside the evaluated scope
  (e.g., non-Q-TOF/Orbitrap data or rare precursor types excluded by the training
  filters)
tools:
- name: FIDDLE
  role: Deep learning model (TCN + rescore checkpoints, Q-TOF and Orbitrap variants)
    whose per-spectrum top-5 predictions are stratified and compared
  repo: https://github.com/JosieHong/FIDDLE
- name: msbuddy (BUDDY)
  role: Optional candidate-formula generator whose outputs can be incorporated in
    file-based comparison workflows
  repo: https://github.com/Philipbear/msbuddy
- name: Q-TOF
  role: Instrument type whose spectra form one stratification axis (10 ppm precursor
    tolerance setting)
- name: Orbitrap
  role: Instrument type whose spectra form the comparison axis (5 ppm precursor tolerance
    setting)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:cb698ff7dac188bea872a277268cdec6a9d31d5243755def7a9ff5a7991b6af2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - FIDDLE
  - Orbitrap
  - Q-TOF
  - msbuddy (BUDDY)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ms-instrument-type-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratified evaluation of a formula-identification model's accuracy by instrument type (Q-TOF vs. Orbitrap) and by molecular strata (LogP polarity bins, precursor mass bins, ClassyFire superclass), as used to characterize FIDDLE's top-5 accuracy trends on the NIST23 test set.

## When to use

Use when comparing a predictor's per-spectrum accuracy across MS/MS instrument types (Q-TOF vs. Orbitrap) or strata such as LogP polarity, precursor mass bins, or chemical superclass, to locate where a model underperforms.

## When NOT to use

- The input is raw vendor MS/MS data before any model inference has produced per-spectrum candidate formulas — run the FIDDLE evaluation pipeline first
- The goal is benchmarking a model against baselines (SIRIUS, BUDDY) on overall accuracy rather than dissecting accuracy across molecular strata and instruments
- Spectra come from instrument types or precursor types outside the evaluated scope (e.g., non-Q-TOF/Orbitrap data or rare precursor types excluded by the training filters)

## Inputs

- Per-spectrum model prediction CSV with top-5 formula candidates (from NIST23 held-out test set)
- Ground-truth molecular formulas for test spectra
- Compound annotations: canonical SMILES (for LogP) and ClassyFire superclass
- Instrument-type labels (Q-TOF vs. Orbitrap) per spectrum

## Outputs

- Stratified top-5 accuracy table (per LogP bin, mass bin, superclass, and instrument type)
- Fig. 3c/d/f-style summary panels showing accuracy trends across strata
- List of underperforming chemical superclasses

## How to apply

Run the trained model (e.g., FIDDLE TCN + rescore checkpoints with Q-TOF and Orbitrap variants) over the held-out test spectra using the repository's evaluation script (experiments_test_nist23.sh), writing per-spectrum predictions to a CSV. Join each prediction with ground-truth formulas and molecular annotations derived from canonical SMILES (LogP) and ClassyFire superclass. Bin spectra by polarity group (polar vs. nonpolar LogP), by precursor mass (e.g., <200 Da vs. larger bins), and by superclass, then compute top-5 accuracy per stratum and per instrument type, keeping instrument-specific parameters in mind (10 ppm precursor tolerance for Q-TOF, 5 ppm for Orbitrap). Visualize as Fig. 3c/d/f-style panels and flag underperforming superclasses such as organohalogens, polymers, organosulfur, and unknown. Validate by reproducing the reported stratum values, e.g., <200 Da top-5 accuracy of 97.3% (Q-TOF) and 95.0% (Orbitrap), and the polar/moderate/nonpolar trend 95.0%/94.7%/88.8%.

## Related tools

- **FIDDLE** (Deep learning model (TCN + rescore checkpoints, Q-TOF and Orbitrap variants) whose per-spectrum top-5 predictions are stratified and compared) — https://github.com/JosieHong/FIDDLE
- **msbuddy (BUDDY)** (Optional candidate-formula generator whose outputs can be incorporated in file-based comparison workflows) — https://github.com/Philipbear/msbuddy
- **Q-TOF** (Instrument type whose spectra form one stratification axis (10 ppm precursor tolerance setting))
- **Orbitrap** (Instrument type whose spectra form the comparison axis (5 ppm precursor tolerance setting))

## Examples

```
bash experiments_test_nist23.sh  # run FIDDLE TCN + rescore checkpoints (Q-TOF/Orbitrap) over the NIST23 test spectra, writing per-spectrum predictions to a results CSV
```

## Evaluation signals

- Stratified table reproduces <200 Da top-5 accuracy of 97.3% (Q-TOF) and 95.0% (Orbitrap)
- LogP strata reproduce the reported monotonic trend: polar 95.0%, moderate 94.7%, nonpolar 88.8% top-5 accuracy (a 6.2% gap consistent across Q-TOF and Orbitrap)
- Overall test-set top-5 accuracy matches the reported 93.6% (top-1 88.3%)
- Underperforming superclass list matches the paper's (organohalogens, polymers, organosulfur, unknown)
- Every spectrum appears in exactly one bin per stratification axis; per-bin counts sum to the full test set

## Limitations

- Q-TOF and Orbitrap training sets differ: data augmentation (Gaussian noise, mean 0, sd 0.1) was applied only to Q-TOF spectra, so cross-instrument gaps may partly reflect training differences
- Experimental condition effects are not fully eliminated; accuracy varies across conditions, especially those with less training data
- Predictor scope is limited to small molecules: spectra with fewer than five peaks, m/z above 1500, non-frequent atoms, or total atom counts above 300 were excluded from the dataset

## Evidence

- [results] FIDDLE's top-5 accuracy is inversely correlated with LogP: polar compounds achieve the highest accuracy (95.0%): [evidence span withheld — anchor status: not_found]
