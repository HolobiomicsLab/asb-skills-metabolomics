---
name: instrument-specific-model-inference
description: Use when MS/MS spectra (MGF with title, precursor_mz, precursor_type,
  collision_energy) originate from a known instrument type (Q-TOF or Orbitrap) and
  formula predictions must use the matching pre-trained FIDDLE weights and mass tolerance.
when_to_use_negative:
- Instrument origin of the spectra is unknown or mixed without labels, since selecting
  the wrong checkpoint/tolerance invalidates the mass-refinement step
- Spectra fall outside the paper's preprocessing scope (e.g., fewer than five peaks,
  m/z above 1500, peaks below 50 m/z, non-standard precursor types)
- Molecules contain atoms outside C, H, O, N, F, S, Cl, P, B, I, Br, Na, K or exceed
  the 300-atom cap, as FIDDLE's candidate space was built for frequent atoms and small
  molecules
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: FIDDLE
  role: Deep-learning formula predictor whose instrument-specific TCN and rescore
    checkpoints are selected per instrument class
  repo: https://github.com/JosieHong/FIDDLE
- name: msfiddle
  role: PyPI CLI/API for loading instrument-appropriate FIDDLE checkpoints and predicting
    formulas from MGF files
  repo: https://github.com/josiehong/msfiddle
- name: BUDDY
  role: Bottom-up comparison predictor run on the same spectra with instrument-specific
    tolerances
  repo: https://github.com/Philipbear/msbuddy
- name: SIRIUS
  role: Top-down comparison predictor (SIRIUS 6) run on the identical spectra for
    benchmarking
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:622a6e6d183f9198edf86a52228c88a6859dfe5e2be8669aeb58ff15e331faa1
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# instrument-specific-model-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run FIDDLE formula prediction with the instrument-appropriate pre-trained TCN and rescore weights plus the matching precursor mass tolerance, so model assumptions align with how the spectra were acquired (Q-TOF vs Orbitrap).

## When to use

Use when MS/MS spectra (MGF with title, precursor_mz, precursor_type, collision_energy) originate from a known instrument type (Q-TOF or Orbitrap) and formula predictions must use the matching pre-trained FIDDLE weights and mass tolerance.

## When NOT to use

- Instrument origin of the spectra is unknown or mixed without labels, since selecting the wrong checkpoint/tolerance invalidates the mass-refinement step
- Spectra fall outside the paper's preprocessing scope (e.g., fewer than five peaks, m/z above 1500, peaks below 50 m/z, non-standard precursor types)
- Molecules contain atoms outside C, H, O, N, F, S, Cl, P, B, I, Br, Na, K or exceed the 300-atom cap, as FIDDLE's candidate space was built for frequent atoms and small molecules

## Inputs

- MGF-format MS/MS test spectra with title, precursor_mz, precursor_type, and collision_energy fields
- Instrument-type label per spectrum or batch (Q-TOF or Orbitrap)
- Pre-trained FIDDLE checkpoints (fiddle_tcn_qtof.pt / fiddle_rescore_qtof.pt or fiddle_tcn_orbitrap.pt / fiddle_rescore_orbitrap.pt)
- Instrument-matched FIDDLE config yml

## Outputs

- Per-spectrum ranked candidate formula lists with predicted confidence scores
- Top-1 and top-5 formula accuracy metrics per instrument type (e.g., noised_accuracy_results.csv)

## How to apply

First determine the instrument class of the input spectra, because FIDDLE ships separate pre-trained checkpoints: fiddle_tcn_qtof.pt with fiddle_rescore_qtof.pt for Q-TOF, and fiddle_tcn_orbitrap.pt with fiddle_rescore_orbitrap.pt for Orbitrap, each paired with its corresponding config yml. Apply the instrument-specific precursor mass tolerance used by the paper: 10 ppm for Q-TOF and 5 ppm for Orbitrap, which constrains the breadth-first candidate-formula refinement step. Feed MGF-format spectra (requiring title, precursor_mz, precursor_type, and collision_energy fields) into running_scripts/experiments_test_noised.sh or the equivalent FIDDLE entry point; the TCN encodes peaks with dilated convolutions, candidate formulas are generated under SENIOR rules, and the Siamese rescore model ranks candidates. Note that only Q-TOF training used Gaussian-noise augmentation (mean 0, sd 0.1); Orbitrap models were trained without it, so noised-input behavior differs by instrument. Validate by comparing top-1/top-5 accuracies against reported baselines (88.3% top-1, 93.6% top-5 overall) and per-instrument figures such as 95.0% top-5 on Q-TOF and 93.3% on Orbitrap under high noise.

## Related tools

- **FIDDLE** (Deep-learning formula predictor whose instrument-specific TCN and rescore checkpoints are selected per instrument class) — https://github.com/JosieHong/FIDDLE
- **msfiddle** (PyPI CLI/API for loading instrument-appropriate FIDDLE checkpoints and predicting formulas from MGF files) — https://github.com/josiehong/msfiddle
- **BUDDY** (Bottom-up comparison predictor run on the same spectra with instrument-specific tolerances) — https://github.com/Philipbear/msbuddy
- **SIRIUS** (Top-down comparison predictor (SIRIUS 6) run on the identical spectra for benchmarking) — https://github.com/sirius-ms/sirius

## Examples

```
bash running_scripts/experiments_test_noised.sh --instrument qtof --tcn fiddle_tcn_qtof.pt --rescore fiddle_rescore_qtof.pt --ppm 10 --input test_noised.mgf
```

## Evaluation signals

- Predictions were produced with the checkpoint matching the instrument (qtof weights for Q-TOF, orbitrap weights for Orbitrap), not a swapped pair
- Precursor mass tolerance matches the paper: 10 ppm for Q-TOF runs and 5 ppm for Orbitrap runs
- Top-5 accuracy at the highest noise level reproduces Fig. 2g values: Q-TOF FIDDLE 95.0% (BUDDY 93.5%, SIRIUS 52.5%); Orbitrap FIDDLE 93.3% (BUDDY 75.4%, SIRIUS 63.1%)
- All candidate formulas satisfy SENIOR rules and align with the precursor mass within tolerance
- Results are written to noised_accuracy_results.csv with per-tool, per-noise-level, per-instrument top-5 accuracies

## Limitations

- No data augmentation was applied for Orbitrap training (adding Gaussian noise lowered similarity for Orbitrap spectra), so Orbitrap models may respond differently to noised inputs
- Accuracy varies across experimental conditions, especially those with less training data
- FIDDLE's performance decreases as spectral mixing (chimeric/co-eluting spectra) increases
- Precursor types are simplified by absorbing neutral molecules (H2O, NH3, CO2, CH2O2, CH4O2) into the formula representation; adduct type prediction is not part of the workflow

## Evidence

- [results] Q-TOF vs Orbitrap weight selection: [evidence span withheld — anchor status: not_found]
