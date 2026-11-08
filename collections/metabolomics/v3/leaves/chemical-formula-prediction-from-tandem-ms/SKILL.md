---
name: chemical-formula-prediction-from-tandem-ms
description: Use when you have MS/MS spectra (MGF with title, precursor_mz, precursor_type,
  collision_energy) from Q-TOF or Orbitrap instruments and need candidate molecular
  formulas ranked by confidence.
when_to_use_negative:
- Input is MS1-only data without tandem MS/MS fragmentation spectra — BUDDY can handle
  m/z-only queries but this FIDDLE skill requires MS/MS
- Spectra are from instruments other than Q-TOF or Orbitrap, or molecules outside
  the 13 frequent atoms / >1500 m/z / >300 atoms — these are excluded by the paper's
  data filters
- Goal is chimeric (co-eluting) spectrum deconvolution — FIDDLE's performance decreases
  as spectral mixing increases
edam_operation: http://edamontology.org/operation_0395
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
tools:
- name: FIDDLE
  role: Primary deep learning model predicting and ranking molecular formulas from
    MS/MS spectra (TCN encoder + BFS candidate refinement + rescore model)
  repo: https://github.com/JosieHong/FIDDLE
- name: BUDDY
  role: Bottom-up baseline that filters candidate formulas using a reference library
    of known MS/MS-explainable formulas for comparison
  repo: https://github.com/Philipbear/msbuddy
- name: SIRIUS
  role: Top-down baseline computing fragmentation trees and isotope pattern scores
    for formula identification comparison
  repo: https://github.com/sirius-ms/sirius
- name: msfiddle
  role: PyPI package providing FIDDLE CLI and Python API for MGF-based predictions
    and batched inference
  repo: https://github.com/josiehong/msfiddle
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:9e2d69f8c591c95588c1d05edffa40721a1eca82d4161d9aa285dbb7be6fc109
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
  edam_operation: http://edamontology.org/operation_0395
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# chemical-formula-prediction-from-tandem-ms

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts molecular formulas from MS/MS tandem mass spectra using FIDDLE, a deep learning model trained on Q-TOF and Orbitrap spectra, and benchmarks it against SIRIUS (top-down) and BUDDY (bottom-up) by top-1/top-5 formula accuracy. Formulas are the first constraint for structural elucidation of 'dark matter' molecules lacking reference spectra.

## When to use

Use when you have MS/MS spectra (MGF with title, precursor_mz, precursor_type, collision_energy) from Q-TOF or Orbitrap instruments and need candidate molecular formulas ranked by confidence.

## When NOT to use

- Input is MS1-only data without tandem MS/MS fragmentation spectra — BUDDY can handle m/z-only queries but this FIDDLE skill requires MS/MS
- Spectra are from instruments other than Q-TOF or Orbitrap, or molecules outside the 13 frequent atoms / >1500 m/z / >300 atoms — these are excluded by the paper's data filters
- Goal is chimeric (co-eluting) spectrum deconvolution — FIDDLE's performance decreases as spectral mixing increases

## Inputs

- MS/MS spectra in MGF format with title, precursor_mz, precursor_type, and collision_energy fields
- Test or query spectra from Q-TOF or Orbitrap instruments (e.g., NIST23, MoNA, GNPS, Agilent PCDL compiled benchmark)
- Pre-trained FIDDLE checkpoints (TCN + rescore weights per instrument)

## Outputs

- Ranked candidate molecular formulas per spectrum with predicted confidence scores
- Top-1 and top-5 formula accuracy metrics per tool per instrument (e.g., noised_accuracy_results.csv)
- Comparative accuracy vs SIRIUS 6 and BUDDY on identical spectra

## How to apply

Prepare spectra as MGF files with precursor m/z, precursor type, and collision energy, retaining molecules with the 13 frequent atoms (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K), m/z in (0, 1500], at least 5 peaks, peaks >= 50 m/z, common precursor adducts like [M + H]+, and <= 300 atoms total. Run FIDDLE with instrument-appropriate pre-trained weights (fiddle_tcn_qtof.pt + fiddle_rescore_qtof.pt for Q-TOF; fiddle_tcn_orbitrap.pt + fiddle_rescore_orbitrap.pt for Orbitrap) and the matching config, applying precursor mass tolerances of 10 ppm (Q-TOF) and 5 ppm (Orbitrap). FIDDLE first predicts a formula with stacked dilated convolutions, then refines candidates via breadth-first search aligned to the precursor mass under SENIOR rules, and ranks them with a secondary rescore model. For robustness testing, add Gaussian noise (mean 0, sd 0.1) to spectrum intensities — a convention used for Q-TOF data augmentation but notably NOT applied to Orbitrap training. Compare against SIRIUS 6 and BUDDY (msbuddy) on identical inputs, evaluating top-1 and top-5 formula accuracy.

## Related tools

- **FIDDLE** (Primary deep learning model predicting and ranking molecular formulas from MS/MS spectra (TCN encoder + BFS candidate refinement + rescore model)) — https://github.com/JosieHong/FIDDLE
- **BUDDY** (Bottom-up baseline that filters candidate formulas using a reference library of known MS/MS-explainable formulas for comparison) — https://github.com/Philipbear/msbuddy
- **SIRIUS** (Top-down baseline computing fragmentation trees and isotope pattern scores for formula identification comparison) — https://github.com/sirius-ms/sirius
- **msfiddle** (PyPI package providing FIDDLE CLI and Python API for MGF-based predictions and batched inference) — https://github.com/josiehong/msfiddle

## Examples

```
bash running_scripts/experiments_test_noised.sh
```

## Evaluation signals

- On the benchmark test set, FIDDLE should reach 88.3% top-1 and 93.6% top-5 accuracy, beating SIRIUS and BUDDY by over 10%
- At the highest Gaussian noise level (mean 0, sd 0.1 convention), top-5 accuracy should approximate Fig. 2g values: Q-TOF FIDDLE 95.0% vs BUDDY 93.5% vs SIRIUS 52.5%; Orbitrap FIDDLE 93.3% vs BUDDY 75.4% vs SIRIUS 63.1%
- All predicted formulas must satisfy SENIOR rules and match the precursor mass within 10 ppm (Q-TOF) or 5 ppm (Orbitrap)
- Output candidate lists must be restricted to the 13 frequent atoms with total atom count <= 300 and precursor m/z <= 1500
- Results table (e.g., noised_accuracy_results.csv) must contain per-tool, per-noise-level, per-instrument top-5 accuracies

## Limitations

- Accuracy varies across experimental conditions, especially those with less training data
- FIDDLE's performance decreases on chimeric spectra as spectral mixing increases
- No data augmentation (Gaussian noise) was applied for Orbitrap training because noise addition lowered spectral similarity; FDR control via confidence thresholds and adduct type prediction remain future work
- BUDDY's scope is restricted by its reference library coverage and cannot propose entirely uncharacterized formulas; SIRIUS's reliance on neutral loss fragments limits performance on multiply-charged spectra

## Evidence
