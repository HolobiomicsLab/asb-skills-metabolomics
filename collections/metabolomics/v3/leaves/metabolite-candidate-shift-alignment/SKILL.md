---
name: metabolite-candidate-shift-alignment
description: Use when a CRNN-reconstructed NMR plasma spectrum is misaligned relative
  to its target spectrum and all metabolite candidates need one global shift before
  local optimization.
when_to_use_negative:
- When the offset is small and local metabolite-level fitting already corrects it
  — the global step targets whole-spectrum positional offsets, not residual peak-level
  misfit
- When the input is raw unprocessed vendor data rather than a CRNN reconstruction
  paired with a target spectrum — the correlation search needs both series
- When spectra are from non-plasma biofluids — the NMRQNet tutorials and preprocessing
  described here cover EDTA and heparin/citrate plasma analysis
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: NMRQNet
  role: Provides the CRNN initial reconstruction and the global shift-correlation
    correction step over metabolite candidates
  repo: github.com/LiuzLab/NMRQNet
- name: EDTA plasma analysis (NMRQNet tutorial)
  role: Reference workflow notebook showing how to run the plasma spectra correction
    (tutorial_for_EDTA_plasma.ipynb)
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:9087f773314827fa45f24368ac2382e72f74f03b36e03ea9a724fc1dd188ec29
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  tools:
  - EDTA plasma analysis (NMRQNet tutorial)
  - NMRQNet
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metabolite-candidate-shift-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Corrects the global positional offset between a CRNN initial reconstruction and the target 1H NMR plasma spectrum by choosing the shift step that maximizes summed regional correlation, then applying that shift to all metabolite candidates.

## When to use

Use when a CRNN-reconstructed NMR plasma spectrum is misaligned relative to its target spectrum and all metabolite candidates need one global shift before local optimization.

## When NOT to use

- When the offset is small and local metabolite-level fitting already corrects it — the global step targets whole-spectrum positional offsets, not residual peak-level misfit
- When the input is raw unprocessed vendor data rather than a CRNN reconstruction paired with a target spectrum — the correlation search needs both series
- When spectra are from non-plasma biofluids — the NMRQNet tutorials and preprocessing described here cover EDTA and heparin/citrate plasma analysis

## Inputs

- CSV of 1H NMR plasma spectra with first column chemical shift values and one column per sample (e.g. NMRQNet/data/tutorial_data/control_human_plasma.csv)
- CRNN initial reconstruction of each spectrum (per-sample spectral estimate)

## Outputs

- Globally corrected spectra stored as CSV with chemical shift values in the first column and one corrected column per plasma sample (e.g. global_corrected_spectra.csv)

## How to apply

Load the plasma csv (first column chemical shift values, one column per sample) with pandas, following the NMRQNet data format. Slide each CRNN initial reconstruction over shifts from -150 to +150 steps relative to the target spectrum; at each candidate shift, partition the spectrum into 30 evenly spaced spectral regions and accumulate the correlation between reconstruction and target across those regions. Select the argmax shift per sample as the global shift and apply it to align all metabolite candidates. The rationale is that a single dominant offset must be removed before local fitting, otherwise residual positional variation dominates the reconstruction error. Save the corrected spectra (chemical shift axis plus one corrected column per sample) for downstream local optimization.

## Related tools

- **NMRQNet** (Provides the CRNN initial reconstruction and the global shift-correlation correction step over metabolite candidates) — github.com/LiuzLab/NMRQNet
- **EDTA plasma analysis (NMRQNet tutorial)** (Reference workflow notebook showing how to run the plasma spectra correction (tutorial_for_EDTA_plasma.ipynb)) — github.com/LiuzLab/NMRQNet

## Examples

```
python tutorial_for_EDTA_plasma.ipynb  # load plasma spectra csv, run global shift search, save global_corrected_spectra.csv
```

## Evaluation signals

- Output CSV retains the same sample columns and chemical shift axis as the input csv
- Each corrected spectrum corresponds to the shift achieving maximum summed correlation over the 30 evenly spaced regions
- Search window stays within the -150 to +150 step range relative to the target spectrum
- Corrected spectra show improved reconstruction fit (lower residual, higher Pearson correlation) versus the un-shifted CRNN reconstruction

## Limitations

- NMRQNet regularizes less on false-positive metabolite predictions, so a final visualization check is still required after alignment
- The global shift applies one offset per sample; peak-specific positional variation is handled only by the subsequent local optimization

## Evidence

- [full_text] NMRQNet's global correction selects the shift step whose reconstructed spectrum yields the highest correlation across regions, and applies that shift to all metabolite candidates.: [evidence span withheld — anchor status: not_found]
