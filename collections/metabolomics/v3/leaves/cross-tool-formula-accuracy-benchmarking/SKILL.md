---
name: cross-tool-formula-accuracy-benchmarking
description: Use when you have a labeled MS/MS challenge dataset (e.g., CASMI 2022,
  304 positive-mode spectra) and need to compare formula-annotation accuracy of two
  or more tools under identical inputs.
when_to_use_negative:
- Evaluating negative-mode spectra — the released models were trained only for positive-mode
  data
- Input spectra lack ground-truth formula/adduct assignments, so no accuracy comparison
  is possible
- The goal is de novo prediction on new data rather than scoring tool performance
  against a labeled benchmark
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MIST-CF
  role: Pretrained formula/adduct ranking model scored on the challenge spectra without
    manual curation
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Out-of-the-box baseline (Submission and CSI:FingerID outputs) plus SIRIUS
    decomp used to generate formula candidates for MIST-CF
  repo: https://github.com/boecker-lab/sirius
- name: CSI:FingerID
  role: SIRIUS-integrated database-search predictor whose top-1 formula accuracy is
    compared against MIST-CF
  repo: https://github.com/sirius-ms/sirius
- name: MZmine
  role: Feature detection and export of the 304 positive-mode MS/MS spectra from challenge
    mzML files
- name: evaluate_pred.py
  role: Analysis script scoring both systems' top-1 predictions against CASMI 2022
    ground truth
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:ec2bce03d240ee6f114927b8306b5d7eea1c35f18e0a7ec8d5a5530ccb390db7
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - CSI:FingerID
  - MIST-CF
  - MZmine
  - SIRIUS
  - analysis/evaluate_pred.py
  - evaluate_pred.py
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Cross-tool formula accuracy benchmarking

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Head-to-head evaluation of chemical formula prediction tools (e.g., MIST-CF vs SIRIUS/CSI:FingerID) on a shared, ground-truthed spectrum set such as the CASMI2022 positive-mode challenge, scoring joint formula+adduct and formula-only top-1 accuracy against Table 2 baselines.

## When to use

Use when you have a labeled MS/MS challenge dataset (e.g., CASMI 2022, 304 positive-mode spectra) and need to compare formula-annotation accuracy of two or more tools under identical inputs.

## When NOT to use

- Evaluating negative-mode spectra — the released models were trained only for positive-mode data
- Input spectra lack ground-truth formula/adduct assignments, so no accuracy comparison is possible
- The goal is de novo prediction on new data rather than scoring tool performance against a labeled benchmark

## Inputs

- casmi22.zip preprocessed challenge data (Zenodo record 8151490), extracted under ./data/casmi22
- CASMI 2022 challenge mzML files with ground-truth formula/adduct assignments
- 304 positive-mode exported MS/MS spectra (MGF, via MZmine 3)
- SIRIUS formula candidate decompositions (SIRIUS decomp) of observed MS1 masses

## Outputs

- Per-spectrum top-1 formula (and formula+adduct) predictions for each tool
- Joint formula-and-adduct accuracy and formula-only top-1 accuracy per tool
- results_table.tsv with per-spectrum predictions and the Table 2 comparison

## How to apply

Extract the challenge spectra (304 positive-mode MS/MS from the CASMI 2022 mzML files via MZmine 3 feature detection/export) and run each tool on the identical exported spectra. Run the pretrained MIST-CF model (src/mist_cf/mist_cf_score/predict_mgf.py) with SIRIUS decomp-generated formula candidates, and run out-of-the-box SIRIUS/CSI:FingerID per run_scripts/casmi22_eval/run_sirius.sh, wrangling its output with run_scripts/casmi22_eval/wrangle_sirius.py. Score top-1 predictions against CASMI ground truth using analysis/evaluate_pred.py, reporting both joint formula-and-adduct accuracy (MIST-CF 0.862 vs SIRIUS submission ~equivalent) and formula-only accuracy (MIST-CF 0.842 vs SIRIUS CSI:FingerID 0.516), reproducing Table 2 (86.8% vs 67.8%). Write per-spectrum predictions and the comparison table to results_table.tsv so per-spectrum disagreements can be audited.

## Related tools

- **MIST-CF** (Pretrained formula/adduct ranking model scored on the challenge spectra without manual curation) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (Out-of-the-box baseline (Submission and CSI:FingerID outputs) plus SIRIUS decomp used to generate formula candidates for MIST-CF) — https://github.com/boecker-lab/sirius
- **CSI:FingerID** (SIRIUS-integrated database-search predictor whose top-1 formula accuracy is compared against MIST-CF) — https://github.com/sirius-ms/sirius
- **MZmine** (Feature detection and export of the 304 positive-mode MS/MS spectra from challenge mzML files)
- **evaluate_pred.py** (Analysis script scoring both systems' top-1 predictions against CASMI 2022 ground truth)

## Examples

```
. run_scripts/casmi22_eval/run_sirius.sh && python run_scripts/casmi22_eval/wrangle_sirius.py && python analysis/evaluate_pred.py --out results_table.tsv
```

## Evaluation signals

- results_table.tsv contains one scored row per spectrum (304 positive-mode entries) for each tool
- MIST-CF joint formula+adduct accuracy lands at ~0.862, nearly equal to SIRIUS (Submission), and formula-only top-1 accuracy at ~0.842 vs 0.516 for SIRIUS (CSI:FingerID)
- The comparison reproduces Table 2 (86.8% vs 67.8% and 0.842 vs 0.516) with no manual curation or postprocessing applied to MIST-CF predictions
- Both tools consumed the identical exported MGF spectra, so accuracy deltas are attributable to the models, not preprocessing differences
- MIST-CF predictions are made without MS1 isotopic information, matching the paper's evaluation setup

## Limitations

- Models were trained only for positive-mode data; the benchmark cannot cover negative-mode spectra
- MIST-CF still relies on SIRIUS's algorithmic decomposition of exact masses into formula candidates, so the comparison is partially coupled to SIRIUS
- The quickstart MIST-CF model (public NPLIB1/CANOPUS training) may underperform the NIST20-trained model, particularly on Orbitrap or higher-resolution data

## Evidence
