---
name: instrument-specific-model-checkpoint-selection
description: Use when running FIDDLE formula prediction or reproducing benchmark accuracies
  on MS/MS spectra whose instrument type (Orbitrap vs Q-TOF) is known, so the matching
  pre-trained weights are loaded.
when_to_use_negative:
- Spectra from instruments other than Orbitrap or Q-TOF, for which no FIDDLE checkpoints
  were trained
- Spectra with m/z above 1500 or molecules outside the frequent-atom set, which fall
  outside FIDDLE's training filter scope
- Use cases requiring the redesigned v2.0.0 rescore model — v1.0.0 checkpoints are
  incompatible with published paper numbers
edam_operation: http://edamontology.org/operation_3947
tools:
- name: FIDDLE
  role: Deep learning model whose instrument-specific TCN and rescore checkpoints
    are selected and loaded
  repo: https://github.com/JosieHong/FIDDLE
- name: msfiddle
  role: PyPI package/CLI that downloads and loads the instrument-matched pre-trained
    FIDDLE models for prediction
  repo: https://github.com/josiehong/msfiddle
- name: BUDDY
  role: Optional candidate-generator whose output is passed via --buddy_path in the
    integrated variant
  repo: https://github.com/Philipbear/msbuddy
- name: SIRIUS
  role: Optional candidate-generator whose output is passed via --sirius_path in the
    integrated variant
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:7d5c380180f8c437e9803bd13cf21f5020a61c55a91109d04f766008ce45e95c
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
  edam_operation: http://edamontology.org/operation_3947
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Instrument-specific model checkpoint selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects and applies the correct FIDDLE pre-trained checkpoint pair (TCN encoder + rescore model) matched to the mass spectrometer type, since separate models were trained for Orbitrap and Q-TOF spectra and their pre-processing tolerances differ.

## When to use

Use when running FIDDLE formula prediction or reproducing benchmark accuracies on MS/MS spectra whose instrument type (Orbitrap vs Q-TOF) is known, so the matching pre-trained weights are loaded.

## When NOT to use

- Spectra from instruments other than Orbitrap or Q-TOF, for which no FIDDLE checkpoints were trained
- Spectra with m/z above 1500 or molecules outside the frequent-atom set, which fall outside FIDDLE's training filter scope
- Use cases requiring the redesigned v2.0.0 rescore model — v1.0.0 checkpoints are incompatible with published paper numbers

## Inputs

- MGF-format MS/MS spectra with title, precursor_mz, precursor_type, and collision_energy fields
- Instrument-type annotation (Orbitrap or Q-TOF) for each spectrum or batch
- Pre-trained checkpoints fiddle_tcn_orbitrap.pt, fiddle_rescore_orbitrap.pt, fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt
- Optional SIRIUS/BUDDY candidate outputs (sirius_output.csv, buddy_output.csv) for the integrated variant

## Outputs

- Ranked candidate molecular formulas with confidence scores per input spectrum
- Top-1, top-5 (and higher-K) accuracy metrics per instrument type

## How to apply

Download the four pre-trained weights (fiddle_tcn_orbitrap.pt, fiddle_rescore_orbitrap.pt, fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt) from the FIDDLE v1.0.0 release page into ./check_point/. Determine each spectrum's instrument type and invoke run_fiddle.py with the config and checkpoint pair for that instrument; never mix a Q-TOF checkpoint with Orbitrap spectra or vice versa because the models encode different training conditions (Q-TOF training used Gaussian noise augmentation with sd 0.1, Orbitrap did not) and different precursor mass tolerances (10 ppm Q-TOF, 5 ppm Orbitrap). For the integrated variant, pass --buddy_path and --sirius_path alongside the instrument-matched checkpoints. Validate the selection by confirming top-1/top-5 accuracies near the reported 88.3%/93.6% on the NIST23 evaluation set. Note that v2.0.0 redesigned the rescore model (Siamese architecture), so v1.0.0 checkpoints are required to reproduce the published numbers.

## Related tools

- **FIDDLE** (Deep learning model whose instrument-specific TCN and rescore checkpoints are selected and loaded) — https://github.com/JosieHong/FIDDLE
- **msfiddle** (PyPI package/CLI that downloads and loads the instrument-matched pre-trained FIDDLE models for prediction) — https://github.com/josiehong/msfiddle
- **BUDDY** (Optional candidate-generator whose output is passed via --buddy_path in the integrated variant) — https://github.com/Philipbear/msbuddy
- **SIRIUS** (Optional candidate-generator whose output is passed via --sirius_path in the integrated variant) — https://github.com/sirius-ms/sirius

## Examples

```
bash running_scripts/experiments_test_nist23.sh  # invokes run_fiddle.py with the matching Orbitrap or Q-TOF config and checkpoints
```

## Evaluation signals

- Reproduced top-1 accuracy ≈88.3% and top-5 ≈93.6% on the NIST23 evaluation set when the correct instrument checkpoints are used
- FIDDLE outperforms SIRIUS and BUDDY by over 10% on the same spectra
- Q-TOF runs use 10 ppm precursor tolerance and Orbitrap runs use 5 ppm, matching the training-time settings
- Large-compound stratum (molecular weight > 800 Da) shows the expected accuracy degradation relative to small precursors (e.g. 0.642 top-1 for Q-TOF, 0.813 for Orbitrap)
- Checkpoint filenames in ./check_point/ match the instrument type of the config passed to run_fiddle.py

## Limitations

- Accuracy varies across experimental conditions, especially those with less training data
- No data augmentation was applied for Orbitrap training, so the Orbitrap model may behave differently from Q-TOF under spectral noise
- v2.0.0 redesigned the rescore model (Siamese architecture), breaking compatibility with v1.0.0 checkpoints and paper reproduction

## Evidence
