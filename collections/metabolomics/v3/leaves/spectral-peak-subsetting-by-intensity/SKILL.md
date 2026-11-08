---
name: spectral-peak-subsetting-by-intensity
description: Use when reproducing the MIST-CF Np peak-ablation sweep (Figure 4 / Table
  1) or when testing how restricting a formula-prediction model to a maximum number
  of intensity-ranked MS2 peaks changes top-1 formula accuracy.
when_to_use_negative:
- 'Input spectra are negative-mode or NIST20-licensed data: the released MIST-CF models
  were trained only for positive mode, and NIST20 models require a NIST license'
- The goal is SIRIUS-style fragmentation-tree-based annotation rather than the MIST-CF
  subformula-assignment pipeline this ablation operates on
- No per-peak intensity metadata or subformula annotations are available, so the Np
  highest-intensity restriction cannot be defined
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MIST-CF
  role: Chemical formula prediction model whose MS2 peak visibility is restricted
    to Np intensity-ranked, subformula-annotated peaks during the ablation
  repo: https://github.com/samgoldman97/mist-cf
- name: NPLIB1
  role: Public natural-products test dataset (extracted from GNPS) on which top-1
    accuracy is recomputed for each Np setting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:1c727b2367650578118a547a53bc017fe51c8bcbfcd3a8bc6aad566471c74fbe
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - MIST-CF
  - NPLIB1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spectral-peak-subsetting-by-intensity

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Subsamples each MS/MS spectrum to its Np highest-intensity, subformula-annotated peaks (plus an Np=0 MS1-only equivalent) to quantify how peak visibility controls MIST-CF top-1 chemical formula accuracy on the NPLIB1 test set.

## When to use

Use when reproducing the MIST-CF Np peak-ablation sweep (Figure 4 / Table 1) or when testing how restricting a formula-prediction model to a maximum number of intensity-ranked MS2 peaks changes top-1 formula accuracy.

## When NOT to use

- Input spectra are negative-mode or NIST20-licensed data: the released MIST-CF models were trained only for positive mode, and NIST20 models require a NIST license
- The goal is SIRIUS-style fragmentation-tree-based annotation rather than the MIST-CF subformula-assignment pipeline this ablation operates on
- No per-peak intensity metadata or subformula annotations are available, so the Np highest-intensity restriction cannot be defined

## Inputs

- MIST-CF predictions on the NPLIB1 test set (from the public GNPS-derived NPLIB1 dataset)
- MS2 spectra with intensity-ranked, subformula-annotated peaks
- Np restriction values: 0, 1, 3, 5, 10, 20, 50

## Outputs

- subpeak_eval_results.csv with one row per Np setting
- Per-Np top-1 formula accuracies (0.623, 0.673, 0.719, 0.729, 0.754, 0.756, 0.774) validated against Figure 4 / Table 1

## How to apply

Load precomputed MIST-CF predictions on the NPLIB1 test set via run_scripts/max_subpeak/eval_models.py. For each Np in {0, 1, 3, 5, 10, 20, 50}, subsample every spectrum to its Np highest-intensity, subformula-annotated MS2 peaks; Np=0 corresponds to the MS1-only equivalence baseline (no MS2 peaks visible to the model). Recompute top-1 formula accuracy per Np setting and tabulate the values in subpeak_eval_results.csv. The expected trajectory is monotonic: 0.623 (Np=0, MS1-only), 0.673 (1), 0.719 (3), 0.729 (5), 0.754 (10), 0.756 (20), 0.774 (50). Peak ordering by intensity matters because MIST-CF consumes subformula-annotated fragments, so a random subsample is not an equivalent ablation.

## Related tools

- **MIST-CF** (Chemical formula prediction model whose MS2 peak visibility is restricted to Np intensity-ranked, subformula-annotated peaks during the ablation) — https://github.com/samgoldman97/mist-cf
- **NPLIB1** (Public natural-products test dataset (extracted from GNPS) on which top-1 accuracy is recomputed for each Np setting)

## Examples

```
python run_scripts/max_subpeak/eval_models.py  # restrict each NPLIB1 test spectrum to Np in {0,1,3,5,10,20,50} intensity-ranked peaks and write subpeak_eval_results.csv
```

## Evaluation signals

- subpeak_eval_results.csv contains exactly one row per Np setting (0, 1, 3, 5, 10, 20, 50)
- Np=0 accuracy equals the MS1-only baseline value 0.623 within numerical tolerance
- Computed accuracies match the reported values 0.673 (Np=1), 0.719 (Np=3), 0.729 (Np=5), 0.754 (Np=10), 0.756 (Np=20), 0.774 (Np=50) within reasonable tolerance
- Accuracy is monotonically non-decreasing as Np increases
- Each subsampled spectrum contains at most Np peaks, and Np=0 spectra contain no MS2 peaks (MS1-only equivalence)

## Limitations

- MIST-CF models were trained only for positive-mode data; negative-mode spectra are not supported
- MIST-CF still relies on SIRIUS's algorithmic decomposition of exact masses into formula candidates, so the ablation requires SIRIUS decomp for candidate enumeration
- The released quickstart model trained on public NPLIB1/CANOPUS data may underperform the NIST20-trained model, particularly for Orbitrap or higher-resolution data

## Evidence
