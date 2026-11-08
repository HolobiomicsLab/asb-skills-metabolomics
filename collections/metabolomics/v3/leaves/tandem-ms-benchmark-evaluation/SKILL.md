---
name: tandem-ms-benchmark-evaluation
description: Use when a trained formula-identification model must be evaluated on
  held-out MS/MS spectra to quantify top-5 accuracy, compare against SIRIUS/BUDDY
  baselines, or reveal performance stratification across molecular properties.
when_to_use_negative:
- Spectra are chimeric (co-eluting) mixtures — FIDDLE's performance decreases as spectral
  mixing increases, so benchmark numbers will not be comparable
- The goal is de novo structural elucidation rather than formula identification benchmarking
  — top-5 formula ranking alone does not resolve structures
- Compounds contain rare elements outside the retained atom set (C, H, O, N, F, S,
  Cl, P, B, I, Br, Na, K) or exceed 1500 m/z — outside the benchmark's explainable
  space
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FIDDLE
  role: Deep learning model whose TCN and rescore checkpoints are evaluated on the
    NIST23 test set to compute top-1/top-5 accuracy
  repo: https://github.com/JosieHong/FIDDLE
- name: SIRIUS
  role: Top-down baseline method compared against FIDDLE on the evaluation set
  repo: https://github.com/boecker-lab/sirius
- name: BUDDY
  role: Bottom-up baseline (msbuddy) whose candidate formula space is compared against
    FIDDLE
  repo: https://github.com/Philipbear/msbuddy
- name: Q-TOF
  role: Instrument type whose held-out test spectra form one evaluation split
- name: Orbitrap
  role: Instrument type whose held-out test spectra form the second evaluation split
    (5 ppm precursor tolerance)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:6004e5e826c733c51472cb91f2a365d9d288bf84dcd593e0b5c6e06184789ec5
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
  - BUDDY
  - FIDDLE
  - Orbitrap
  - Q-TOF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# tandem-ms-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark-evaluates deep learning molecular formula identification (FIDDLE) on held-out tandem MS/MS test spectra (NIST23), reporting top-1/top-5 accuracy on Q-TOF and Orbitrap instruments and stratifying performance by molecular properties such as LogP polarity, precursor mass, and ClassyFire superclass.

## When to use

Use when a trained formula-identification model must be evaluated on held-out MS/MS spectra to quantify top-5 accuracy, compare against SIRIUS/BUDDY baselines, or reveal performance stratification across molecular properties.

## When NOT to use

- Spectra are chimeric (co-eluting) mixtures — FIDDLE's performance decreases as spectral mixing increases, so benchmark numbers will not be comparable
- The goal is de novo structural elucidation rather than formula identification benchmarking — top-5 formula ranking alone does not resolve structures
- Compounds contain rare elements outside the retained atom set (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K) or exceed 1500 m/z — outside the benchmark's explainable space

## Inputs

- Held-out MS/MS test spectra from NIST23 (Q-TOF and Orbitrap instrument types)
- Ground-truth molecular formulas per spectrum
- Pre-trained FIDDLE TCN and rescore checkpoints (Orbitrap and Q-TOF variants)
- Canonical SMILES / molecular annotations (LogP, ClassyFire superclass) for test compounds

## Outputs

- Per-spectrum prediction CSV with ranked candidate formulas
- Top-1 and top-5 accuracy tables overall and per stratum (LogP group, mass bin, superclass)
- Fig. 3c/d/f-style summary panels flagging underperforming superclasses

## How to apply

Run the released FIDDLE TCN and rescore checkpoints (separate Q-TOF and Orbitrap variants) over the held-out test spectra via the repository's NIST23 evaluation script (experiments_test_nist23.sh), writing per-spectrum predictions to a results CSV. Parse the CSV, join each spectrum's ground-truth formula with molecular annotations (canonical SMILES-derived LogP, ClassyFire superclass), and bin spectra by polarity group, precursor mass (e.g., <200 Da vs. larger bins), and superclass. Compute top-5 accuracy per stratum per instrument type; apply the paper's data filters when preparing inputs (m/z range (0, 1500], >=5 peaks, peaks >=50 m/z, molecules composed of frequent atoms with <=300 atoms, common precursor types). Judge correctness by reproducing headline metrics (88.3% top-1, 93.6% top-5 on the evaluation set, >10% over SIRIUS and BUDDY) and stratified values such as 97.3% (Q-TOF) / 95.0% (Orbitrap) top-5 for <200 Da and the polarity trend polar 95.0% > moderate 94.7% > nonpolar 88.8%.

## Related tools

- **FIDDLE** (Deep learning model whose TCN and rescore checkpoints are evaluated on the NIST23 test set to compute top-1/top-5 accuracy) — https://github.com/JosieHong/FIDDLE
- **SIRIUS** (Top-down baseline method compared against FIDDLE on the evaluation set) — https://github.com/boecker-lab/sirius
- **BUDDY** (Bottom-up baseline (msbuddy) whose candidate formula space is compared against FIDDLE) — https://github.com/Philipbear/msbuddy
- **Q-TOF** (Instrument type whose held-out test spectra form one evaluation split)
- **Orbitrap** (Instrument type whose held-out test spectra form the second evaluation split (5 ppm precursor tolerance))

## Examples

```
bash experiments_test_nist23.sh --instrument qtof --check_point ~/.msfiddle/check_point --output results_nist23_qtof.csv
```

## Evaluation signals

- Reproduce headline metrics: top-1 accuracy 88.3% and top-5 accuracy 93.6% on the evaluation set, outperforming SIRIUS and BUDDY by over 10%
- Stratified table reproduces <200 Da top-5 accuracy of 97.3% (Q-TOF) and 95.0% (Orbitrap)
- LogP stratification shows the inverse trend: polar 95.0% > moderate 94.7% > nonpolar 88.8% top-5 accuracy, consistent across Q-TOF and Orbitrap
- Prediction CSV contains per-spectrum top-5 candidates for every non-filtered test spectrum (>=5 peaks, m/z in (0, 1500], frequent atoms, <=300 atoms)
- External metabolomics spot-check: top-5 accuracy ~75.1% (positive) / 66.2% (negative) mode, rising to 80.0% / 73.8% with SIRIUS+BUDDY candidates

## Limitations

- Accuracy degrades on chimeric (co-eluting) spectra as spectral mixing increases
- Experimental condition effects are not fully eliminated; accuracy varies across conditions, especially those with less training data
- FDR control via confidence score thresholds is proposed but not implemented, so benchmark accuracies are uncalibrated per-spectrum
- Adduct type prediction is not part of the current workflow, limiting end-to-end application
- No data augmentation was applied for Orbitrap training (Gaussian noise, mean 0 / sd 0.1, was used only for Q-TOF)

## Evidence

- [results] Top-5 accuracy inversely correlated with LogP: [evidence span withheld — anchor status: not_found]
