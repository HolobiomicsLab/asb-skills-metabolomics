---
name: nmr-spectral-global-shift-correction
description: Use when a CRNN reconstruction of a 1H NMR plasma spectrum is misaligned
  relative to the target spectrum and metabolite candidates must be aligned before
  local quantification optimization.
when_to_use_negative:
- When the misalignment is local/peak-specific rather than a global positional offset
  — the article notes NMRQNet regularizes less on false-positive predictions and requires
  a final visualization check, not a further shift search.
- When working with non-plasma biofluid samples — NMRQNet's biofluid-specific analysis
  pipeline is not yet developed per the discussion.
- When spectra are already aligned or when preprocessing has already removed positional
  variation.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3488
tools:
- name: NMRQNet
  role: Provides the CRNN initial reconstruction and the EDTA plasma analysis tutorial
    that defines the shift-search procedure over -150 to +150 steps with 30 regions.
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:5d9f02690541a82dee8967a23cc3aff3f1025faa0a4e3cc8388462e860583637
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  tools:
  - NMRQNet
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3488
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# nmr-spectral-global-shift-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Corrects the global positional offset between a CRNN-initialized NMR spectrum reconstruction and the target plasma spectrum by selecting the shift whose regional correlations across 30 spectral regions are maximal, then applying that shift to all metabolite candidates.

## When to use

Use when a CRNN reconstruction of a 1H NMR plasma spectrum is misaligned relative to the target spectrum and metabolite candidates must be aligned before local quantification optimization.

## When NOT to use

- When the misalignment is local/peak-specific rather than a global positional offset — the article notes NMRQNet regularizes less on false-positive predictions and requires a final visualization check, not a further shift search.
- When working with non-plasma biofluid samples — NMRQNet's biofluid-specific analysis pipeline is not yet developed per the discussion.
- When spectra are already aligned or when preprocessing has already removed positional variation.

## Inputs

- plasma spectra csv with chemical shift values in the first column and one column per sample (e.g. control_human_plasma.csv format)
- CRNN initial reconstruction of each plasma spectrum

## Outputs

- global_corrected_spectra.csv containing chemical shift values and one corrected column per plasma sample
- per-sample selected global shift (argmax of summed regional correlation)

## How to apply

Load the plasma spectra csv (first column chemical shift values, one column per sample) with pandas, following the NMRQNet data format. Take the CRNN initial reconstruction of each spectrum and slide it over shifts from -150 to +150 steps relative to the target spectrum. At each candidate shift, divide the spectrum into 30 evenly spaced spectral regions and accumulate the correlation between reconstruction and target across those regions. Select the shift with the maximum accumulated correlation (argmax) as the global shift for each sample, and apply it to all metabolite candidates in that sample's reconstruction before local optimization.

## Related tools

- **NMRQNet** (Provides the CRNN initial reconstruction and the EDTA plasma analysis tutorial that defines the shift-search procedure over -150 to +150 steps with 30 regions.) — github.com/LiuzLab/NMRQNet

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('control_human_plasma.csv'); df.to_csv('global_corrected_spectra.csv')"  # after shift search per NMRQNet EDTA plasma tutorial
```

## Evaluation signals

- Output file global_corrected_spectra.csv exists and retains the same sample columns and chemical shift axis as the input csv.
- Each corrected spectrum corresponds to the shift achieving maximum summed correlation over the 30 regions.
- Selected shift lies within the -150 to +150 step search window.
- Reconstructed-vs-target correlation computed across 30 evenly spaced regions is maximal at the chosen shift.
- Corrected spectra feed the subsequent local optimization without further global alignment residual.

## Limitations

- The correction is a global shift only; it cannot fix local peak-pattern variations brought by environmental and internal factors, which is why NMRQNet also requires local optimization.
- NMRQNet regularizes less on false-positive predictions, so a final visualization check is still required after correction and quantification.
- The shift range (-150 to +150 steps) and 30-region segmentation are fixed choices from the tutorial and may not generalize to spectra with larger positional offsets.

## Evidence
