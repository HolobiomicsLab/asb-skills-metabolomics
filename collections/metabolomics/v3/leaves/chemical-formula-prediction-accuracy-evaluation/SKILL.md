---
name: chemical-formula-prediction-accuracy-evaluation
description: Use when benchmarking a de novo chemical formula predictor on MS/MS spectra
  (e.g., NPLIB1 test set) and you need reproducible top-1 accuracy values, especially
  as a function of how many MS2 peaks are visible to the model.
when_to_use_negative:
- Input spectra are negative-mode, since MIST-CF models were trained only for positive-mode
  data
- The goal is prospective benchmarking on CASMI2022 challenge spectra rather than
  NPLIB1 top-1 accuracy sweeps
- Spectra are database-searched assignments rather than de novo formula prediction
  candidates (MIST-CF does not reference spectrum databases)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0157
tools:
- name: MIST-CF
  role: Energy-based neural network model whose top-1 formula accuracy on NPLIB1 is
    measured across Np peak-restriction settings
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Provides the SIRIUS decomp algorithmic decomposition of exact MS1 masses into
    the formula candidate lists that MIST-CF then scores
  repo: https://github.com/boecker-lab/sirius/releases/download/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:656e9d48a01f77a506c1b51435e0723b9b7afd6271a1e23d1623d3e07fc67270
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - MIST-CF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chemical-formula-prediction-accuracy-evaluation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Evaluates top-1 chemical formula accuracy of MIST-CF on NPLIB1 test spectra, including the Np peak-ablation sweep that restricts each spectrum to its Np highest-intensity subformula-annotated MS2 peaks (Np = 0,1,3,5,10,20,50), with Np=0 serving as the MS1-only baseline.

## When to use

Use when benchmarking a de novo chemical formula predictor on MS/MS spectra (e.g., NPLIB1 test set) and you need reproducible top-1 accuracy values, especially as a function of how many MS2 peaks are visible to the model.

## When NOT to use

- Input spectra are negative-mode, since MIST-CF models were trained only for positive-mode data
- The goal is prospective benchmarking on CASMI2022 challenge spectra rather than NPLIB1 top-1 accuracy sweeps
- Spectra are database-searched assignments rather than de novo formula prediction candidates (MIST-CF does not reference spectrum databases)

## Inputs

- MIST-CF prediction outputs on the NPLIB1 test set (spectra with MS1 precursor mass and subformula-annotated MS2 peaks)
- NPLIB1 test set spectra (public natural products dataset from GNPS, as preprocessed in the mist-cf repo)
- List of Np restriction values: 0, 1, 3, 5, 10, 20, 50

## Outputs

- subpeak_eval_results.csv with one row per Np setting and its top-1 formula accuracy
- Validation report confirming Np=0 matches the MS1-only baseline (0.623) and monotonic accuracy increase up to 0.774 at Np=50

## How to apply

Load MIST-CF predictions on the NPLIB1 test set produced via run_scripts/max_subpeak/eval_models.py in the mist-cf repository. For each Np restriction value (0, 1, 3, 5, 10, 20, 50), subsample each spectrum to its Np highest-intensity MS2 peaks that carry subformula annotations, then recompute top-1 formula accuracy on the restricted spectra; Np=0 makes the model effectively MS1-only. The key decision point is that the reported accuracies rise monotonically — 0.673 (Np=1), 0.719 (Np=3), 0.729 (Np=5), 0.754 (Np=10), 0.756 (Np=20), 0.774 (Np=50) — while Np=0 must collapse to the MS1-only baseline of 0.623. Tabulate the per-Np accuracies into subpeak_eval_results.csv with one row per Np setting and compare each value against the reported numbers within reasonable numerical tolerance.

## Related tools

- **MIST-CF** (Energy-based neural network model whose top-1 formula accuracy on NPLIB1 is measured across Np peak-restriction settings) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (Provides the SIRIUS decomp algorithmic decomposition of exact MS1 masses into the formula candidate lists that MIST-CF then scores) — https://github.com/boecker-lab/sirius/releases/download/

## Examples

```
python run_scripts/max_subpeak/eval_models.py  # recomputes per-Np top-1 accuracy and writes subpeak_eval_results.csv
```

## Evaluation signals

- subpeak_eval_results.csv contains exactly one row per Np setting (0, 1, 3, 5, 10, 20, 50)
- Np=0 accuracy equals the MS1-only baseline of 0.623 within numerical tolerance
- Accuracy is monotonically non-decreasing with Np, matching 0.673, 0.719, 0.729, 0.754, 0.756, 0.774 within reasonable tolerance
- The Np=50 accuracy (0.774) recovers the full-spectrum MIST-CF performance reported for NPLIB1

## Limitations

- Models were trained only for positive-mode data, so this evaluation cannot be extended to negative-mode spectra
- MIST-CF scores are not integrated with MS1 isotopic scores, so the Np=0 case reflects precursor-mass-only conditioning without isotope-pattern scoring
- Adduct switching in MS2 subformula assignment is not considered, which bounds achievable top-1 accuracy

## Evidence

- [results] Restricting MIST-CF to Np intensity-ranked peaks shows accuracy rising from 0.673 (1 peak) to 0.719 (3 peaks): [evidence span withheld — anchor status: not_found]
- [results] Np=0 matching the MS1-only baseline at 0.623: [evidence span withheld — anchor status: not_found]
