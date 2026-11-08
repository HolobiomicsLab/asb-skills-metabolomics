---
name: top-k-accuracy-benchmark-evaluation
description: Use when benchmarking chemical formula identification from tandem MS/MS
  spectra against published baselines (e.g., reproducing FIDDLE's 88.3% top-1 / 93.6%
  top-5 accuracy on NIST23 against SIRIUS and BUDDY).
when_to_use_negative:
- Spectra contain chimeric (co-eluting) compounds — FIDDLE's performance decreases
  as spectral mixing increases, so benchmark numbers will not match clean-spectra
  expectations
- Precursor m/z values exceed 1500 — the m/z range is confined to (0, 1500] due to
  the rarity of spectra above that value
- Molecules contain rare atoms outside C, H, O, N, F, S, Cl, P, B, I, Br, Na, K, or
  exceed 300 total atoms — they are outside the training/model scope
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0121
tools:
- name: FIDDLE
  role: Deep learning method under evaluation; predicts molecular formulas from MS/MS
    spectra via pre-trained TCN and rescore checkpoints
  repo: https://github.com/JosieHong/FIDDLE
- name: SIRIUS
  role: Top-down baseline whose formula predictions are benchmarked against FIDDLE
    on the same test spectra
  repo: https://github.com/sirius-ms/sirius
- name: BUDDY
  role: Bottom-up baseline (msbuddy) whose formula predictions are benchmarked against
    FIDDLE on the same test spectra
  repo: https://github.com/Philipbear/msbuddy
- name: msfiddle
  role: PyPI CLI/API packaging of FIDDLE for formula prediction from MGF files, supporting
    file-based BUDDY/SIRIUS candidate integration
  repo: https://github.com/josiehong/msfiddle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:9d73f8c86f77c15df468419df74a5be28c5ab537009e488e1bdcb8754ff7b487
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
  - Q-TOF
  - SIRIUS
  - msfiddle
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# Top-K accuracy benchmark evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates molecular formula prediction methods (FIDDLE, SIRIUS, BUDDY) on held-out NIST23-only test spectra by computing top-1 and top-5 (and higher-K) accuracies per instrument type (Q-TOF vs. Orbitrap), with optional stratification by precursor mass (e.g., compounds >800 Da).

## When to use

Use when benchmarking chemical formula identification from tandem MS/MS spectra against published baselines (e.g., reproducing FIDDLE's 88.3% top-1 / 93.6% top-5 accuracy on NIST23 against SIRIUS and BUDDY).

## When NOT to use

- Spectra contain chimeric (co-eluting) compounds — FIDDLE's performance decreases as spectral mixing increases, so benchmark numbers will not match clean-spectra expectations
- Precursor m/z values exceed 1500 — the m/z range is confined to (0, 1500] due to the rarity of spectra above that value
- Molecules contain rare atoms outside C, H, O, N, F, S, Cl, P, B, I, Br, Na, K, or exceed 300 total atoms — they are outside the training/model scope

## Inputs

- NIST23-only test spectra in MGF format with title, precursor_mz, precursor_type, and collision_energy fields
- FIDDLE pre-trained checkpoints (fiddle_tcn_orbitrap.pt, fiddle_rescore_orbitrap.pt, fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt)
- SIRIUS predictions exported as sirius_output.csv
- BUDDY predictions exported as buddy_output.csv

## Outputs

- Top-1, top-5, and higher-K accuracy metrics per instrument type (Q-TOF, Orbitrap)
- Mass-stratified accuracy breakdown for compounds above 800 Da (as in Fig. 2)

## How to apply

Prepare the NIST23-only test spectra in MGF format with title, precursor_mz, precursor_type, and collision_energy fields, and run SIRIUS (v6.0.1) and BUDDY (v0.3.6) on the same spectra, exporting their predictions as sirius_output.csv and buddy_output.csv. Download the FIDDLE v1.0.0 pre-trained checkpoints (fiddle_tcn_orbitrap.pt, fiddle_rescore_orbitrap.pt, fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt) into ./check_point/, matching the model to the instrument type. Execute running_scripts/experiments_test_nist23.sh, which invokes run_fiddle.py with the matching Orbitrap or Q-TOF config and checkpoints, adding --buddy_path and --sirius_path for the integrated variant. Compute top-1, top-5, and higher-K accuracies per instrument type, and stratify accuracy by precursor mass with a breakdown for compounds above 800 Da (FIDDLE reported 0.642 top-1 for Q-TOF and 0.813 for Orbitrap in that stratum). The rationale is that accuracy decreases with increasing precursor m/z due to the exponentially growing candidate formula space, so mass-stratified reporting reveals large-compound degradation.

## Related tools

- **FIDDLE** (Deep learning method under evaluation; predicts molecular formulas from MS/MS spectra via pre-trained TCN and rescore checkpoints) — https://github.com/JosieHong/FIDDLE
- **SIRIUS** (Top-down baseline whose formula predictions are benchmarked against FIDDLE on the same test spectra) — https://github.com/sirius-ms/sirius
- **BUDDY** (Bottom-up baseline (msbuddy) whose formula predictions are benchmarked against FIDDLE on the same test spectra) — https://github.com/Philipbear/msbuddy
- **msfiddle** (PyPI CLI/API packaging of FIDDLE for formula prediction from MGF files, supporting file-based BUDDY/SIRIUS candidate integration) — https://github.com/josiehong/msfiddle

## Examples

```
bash running_scripts/experiments_test_nist23.sh
```

## Evaluation signals

- FIDDLE reproduces the reported ~88.3% top-1 and ~93.6% top-5 accuracy on the evaluation set
- FIDDLE outperforms SIRIUS and BUDDY by over 10% on the same test spectra
- The >800 Da precursor mass stratum shows degraded accuracy relative to small precursors (Q-TOF top-1 ~0.642, Orbitrap top-1 ~0.813)
- Metrics are reported separately per instrument type (Q-TOF vs. Orbitrap) with matching checkpoint/config assignment
- MGF input parses with the required fields: title, precursor_mz, precursor_type, collision_energy

## Limitations

- Accuracy varies across experimental conditions, especially those with less training data
- Performance degrades on chimeric (co-eluting) spectra as spectral mixing increases
- FDR control via confidence score thresholds is proposed but not implemented; adduct type prediction is not part of the workflow
- No data augmentation was applied for Orbitrap training (Gaussian noise addition lowered similarity for Orbitrap spectra)

## Evidence
