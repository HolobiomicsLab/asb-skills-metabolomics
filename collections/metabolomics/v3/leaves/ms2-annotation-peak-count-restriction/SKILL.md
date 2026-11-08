---
name: ms2-annotation-peak-count-restriction
description: Use when you must quantify the marginal contribution of MS2 peak information
  to de novo formula annotation accuracy — e.g., reproducing an Np peak-ablation sweep
  or benchmarking MS1-only versus MS2-conditioned formula prediction on spectra with
  subformula annotations.
when_to_use_negative:
- Input data is negative-mode MS/MS — MIST-CF models were trained only for positive-mode
  data
- The goal is fingerprint or structure prediction (original MIST / CSI:FingerID) rather
  than de novo chemical formula annotation accuracy
- Spectra lack per-peak subformula annotations, since the subsampling step selects
  intensity-ranked subformula-annotated peaks
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MIST-CF
  role: Energy-based neural network model whose NPLIB1 top-1 formula accuracies are
    recomputed under each Np peak restriction
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: SIRIUS decomp supplies the algorithmic decomposition of exact MS1 masses into
    formula candidates that MIST-CF then scores under each peak restriction
  repo: https://github.com/boecker-lab/sirius
- name: NPLIB1
  role: Public natural-products test dataset from GNPS on which the per-Np accuracies
    are computed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:810983ce789f3209c018a79e855637511bd8e2bd10152444d4ad5660244b79cb
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - MIST-CF
  - NPLIB1
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# MS2 annotation peak-count restriction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A sensitivity-analysis skill for MIST-CF chemical formula annotation on NPLIB1 in which each MS/MS spectrum is subsampled to its Np highest-intensity, subformula-annotated MS2 peaks before scoring, quantifying how much MS2 peak information drives top-1 formula accuracy (Np = 0,1,3,5,10,20,50; Np=0 equivalently reduces to the MS1-only model).

## When to use

Use when you must quantify the marginal contribution of MS2 peak information to de novo formula annotation accuracy — e.g., reproducing an Np peak-ablation sweep or benchmarking MS1-only versus MS2-conditioned formula prediction on spectra with subformula annotations.

## When NOT to use

- Input data is negative-mode MS/MS — MIST-CF models were trained only for positive-mode data
- The goal is fingerprint or structure prediction (original MIST / CSI:FingerID) rather than de novo chemical formula annotation accuracy
- Spectra lack per-peak subformula annotations, since the subsampling step selects intensity-ranked subformula-annotated peaks

## Inputs

- MIST-CF predictions on the NPLIB1 test set (precomputed model outputs from run_scripts/max_subpeak/eval_models.py)
- NPLIB1 MS/MS spectra with intensity-ranked, subformula-annotated MS2 peaks (dataset extracted from the GNPS database)
- List of Np restriction values: 0, 1, 3, 5, 10, 20, 50

## Outputs

- subpeak_eval_results.csv with one row per Np setting and the corresponding top-1 formula accuracy
- Accuracy table matching Figure 4 / Table 1: 0.623 (MS1-only/Np=0), 0.673, 0.719, 0.729, 0.754, 0.756, 0.774 for increasing Np

## How to apply

Load the precomputed MIST-CF predictions on the NPLIB1 test set (produced via run_scripts/max_subpeak/eval_models.py in the mist-cf repository). For each Np restriction value (0, 1, 3, 5, 10, 20, 50), subsample each spectrum to its Np highest-intensity MS2 peaks that carry subformula annotations; Np=0 removes all MS2 peaks and therefore serves as the MS1-only equivalence baseline, since MIST-CF otherwise scores agreement between a precursor formula candidate and the MS/MS spectrum. Recompute top-1 formula accuracy on the restricted spectra for every Np setting. The expected accuracy trajectory rises monotonically from 0.623 (Np=0/MS1-only) through 0.673 (Np=1), 0.719 (Np=3), 0.729 (Np=5), 0.754 (Np=10), 0.756 (Np=20) to 0.774 (Np=50). Tabulate one row per Np into subpeak_eval_results.csv and check the computed values against the reported ones within numerical tolerance.

## Related tools

- **MIST-CF** (Energy-based neural network model whose NPLIB1 top-1 formula accuracies are recomputed under each Np peak restriction) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (SIRIUS decomp supplies the algorithmic decomposition of exact MS1 masses into formula candidates that MIST-CF then scores under each peak restriction) — https://github.com/boecker-lab/sirius
- **NPLIB1** (Public natural-products test dataset from GNPS on which the per-Np accuracies are computed)

## Examples

```
python run_scripts/max_subpeak/eval_models.py
```

## Evaluation signals

- Np=0 accuracy must equal the MS1-only baseline value of 0.623 on the NPLIB1 test set
- Computed top-1 accuracies match the reported series 0.673 (Np=1), 0.719 (Np=3), 0.729 (Np=5), 0.754 (Np=10), 0.756 (Np=20), 0.774 (Np=50) within reasonable numerical tolerance
- Accuracy is monotonically non-decreasing as Np increases (each additional peak budget cannot lower the intensity-ranked subsample)
- subpeak_eval_results.csv contains exactly one row per Np restriction setting (7 rows for the Np = 0,1,3,5,10,20,50 sweep)

## Limitations

- MIST-CF models were trained only for positive-mode data, so the sweep cannot be extended to negative-ion spectra
- Candidate formulae still rely on SIRIUS's algorithmic decomposition of exact masses due to its fast implementation
- The quickstart model (public NPLIB1/CANOPS-trained) may be less performant than NIST20-trained models, which are only available upon request to users with a NIST license

## Evidence
