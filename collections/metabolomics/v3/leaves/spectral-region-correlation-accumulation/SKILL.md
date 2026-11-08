---
name: spectral-region-correlation-accumulation
description: Use when a CRNN (or similar) reconstruction of a plasma NMR spectrum
  is misaligned relative to the target spectrum and metabolite candidates must be
  globally shifted before local optimization.
when_to_use_negative:
- The reconstruction and target spectrum are already aligned, so a global shift search
  would be redundant
- Local, per-metabolite fine alignment is needed rather than a single bulk shift applied
  to all candidates
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3272
tools:
- name: NMRQNet
  role: Deep-learning pipeline whose CRNN initial reconstruction is globally shifted
    using the region-correlation argmax step before local optimization
  repo: github.com/LiuzLab/NMRQNet
- name: NMRQNet EDTA plasma analysis (NMRQNet tutorial)
  role: Jupyter notebook (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb) demonstrating
    the EDTA plasma workflow in which this global correction step is applied
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:935f369b42b460021b0a62e792e4d6e2b1f24e33a14a2dbb768da55ce76eca14
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
  - NMRQNet EDTA plasma analysis (NMRQNet tutorial)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spectral-region-correlation-accumulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Corrects the global positional offset between NMRQNet's CRNN initial reconstruction and the target plasma 1H NMR spectrum by sliding the reconstruction over shifts of -150 to +150 steps and selecting the shift whose 30-region summed correlation with the target is maximal.

## When to use

Use when a CRNN (or similar) reconstruction of a plasma NMR spectrum is misaligned relative to the target spectrum and metabolite candidates must be globally shifted before local optimization.

## When NOT to use

- The reconstruction and target spectrum are already aligned, so a global shift search would be redundant
- Local, per-metabolite fine alignment is needed rather than a single bulk shift applied to all candidates

## Inputs

- Plasma spectra csv in NMRQNet format: first column chemical shift values, one column per plasma sample (e.g. control_human_plasma.csv)
- CRNN initial reconstruction of each plasma spectrum

## Outputs

- global_corrected_spectra.csv with chemical shift values in the first column and one globally shifted, corrected column per plasma sample
- Per-sample argmax global shift within -150 to +150 steps

## How to apply

Load the plasma spectra csv (first column chemical shift values, one column per sample) with pandas per the NMRQNet input format. For each sample, take the CRNN initial reconstruction and slide it over shifts from -150 to +150 steps relative to the target spectrum. At each candidate shift, divide the spectrum into 30 evenly spaced spectral regions and accumulate the correlation between reconstruction and target across those regions. Select the argmax shift as the global shift, apply it to align all metabolite candidates, and save the corrected spectra (chemical shift values plus one corrected column per sample) to global_corrected_spectra.csv. The rationale is that a single best global shift maximizing regional correlation removes bulk positional variation before the per-metabolite local optimization stage.

## Related tools

- **NMRQNet** (Deep-learning pipeline whose CRNN initial reconstruction is globally shifted using the region-correlation argmax step before local optimization) — github.com/LiuzLab/NMRQNet
- **NMRQNet EDTA plasma analysis (NMRQNet tutorial)** (Jupyter notebook (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb) demonstrating the EDTA plasma workflow in which this global correction step is applied) — github.com/LiuzLab/NMRQNet

## Examples

```
df = pd.read_csv('control_human_plasma.csv'); shifts = range(-150, 151); best = max(shifts, key=lambda s: sum(region_corrs(recon.shift(s), target, n_regions=30)))
```

## Evaluation signals

- Output file global_corrected_spectra.csv exists and has the same sample columns and chemical shift axis as the input csv
- Each corrected spectrum corresponds to the shift in [-150, +150] steps achieving maximum summed correlation over the 30 evenly spaced spectral regions
- Correlation between each corrected reconstruction and its target spectrum is at least as high as at any other candidate shift
- Chemical shift values in the first column of the output are unchanged relative to the input csv

## Limitations

- Only a single global shift per sample is estimated; residual local positional or pattern variations are left to downstream local optimization
- The search range is bounded at +/-150 steps, so offsets larger than this will not be corrected
- NMRQNet regularizes less on false-positive metabolite predictions, requiring a final manual visualization check after quantification

## Evidence

- [readme] Input format details: [evidence span withheld — anchor status: not_found]
