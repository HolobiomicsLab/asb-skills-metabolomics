---
name: cross-correlation-spectral-alignment
description: Use when a CRNN (or similar) initial NMR reconstruction exhibits a global
  positional offset relative to the target plasma spectrum and metabolite candidates
  must be aligned before local optimization.
when_to_use_negative:
- The offset is a local or per-metabolite misalignment rather than a global positional
  offset of the whole spectrum — this step applies one shift to all metabolite candidates
  and defers finer alignment to NMRQNet's local optimization.
- Input spectra are already positionally aligned to the target, so a -150 to +150
  step shift search would be redundant.
- Data are not 1H NMR plasma spectra in the NMRQNet csv format (e.g. raw FID files
  or feature tables).
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: NMRQNet
  role: Pipeline whose global positional correction step this skill reproduces; consumes
    the plasma csv and performs the shift-search alignment of the CRNN reconstruction
    before local optimization.
  repo: github.com/LiuzLab/NMRQNet
- name: NMRQNet EDTA plasma analysis (NMRQNet tutorial)
  role: Reference notebook (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb) demonstrating
    the EDTA plasma analysis workflow including this global correction.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:e6ef3da15ab12b8aa49bf1547b1b5f3d1b42bbff78638007e2deb1976ad14cd7
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cross-correlation-spectral-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A global positional correction step from NMRQNet that aligns a CRNN initial reconstruction to its target plasma 1H NMR spectrum by sliding the reconstruction over shifts from -150 to +150 steps and selecting the shift whose 30-region accumulated correlation with the target is maximal.

## When to use

Use when a CRNN (or similar) initial NMR reconstruction exhibits a global positional offset relative to the target plasma spectrum and metabolite candidates must be aligned before local optimization.

## When NOT to use

- The offset is a local or per-metabolite misalignment rather than a global positional offset of the whole spectrum — this step applies one shift to all metabolite candidates and defers finer alignment to NMRQNet's local optimization.
- Input spectra are already positionally aligned to the target, so a -150 to +150 step shift search would be redundant.
- Data are not 1H NMR plasma spectra in the NMRQNet csv format (e.g. raw FID files or feature tables).

## Inputs

- plasma spectra csv in NMRQNet format: first column chemical shift values, one subsequent column per sample (e.g. NMRQNet/data/tutorial_data/control_human_plasma.csv)
- CRNN initial reconstruction spectrum for each sample

## Outputs

- global_corrected_spectra.csv with chemical shift values and one corrected column per plasma sample
- per-sample global shift (argmax over -150 to +150 steps)

## How to apply

Load the plasma spectra csv (first column chemical shift values, one column per sample) with pandas, following the NMRQNet data format where each column is the 1H NMR spectrum for one plasma sample. For each sample, take the CRNN initial reconstruction and slide it over shifts from -150 to +150 steps relative to the target spectrum. At each candidate shift, divide the spectrum into 30 evenly spaced spectral regions and accumulate the correlation between reconstruction and target across those regions. Select the shift with the maximum accumulated correlation (argmax) as the global shift and apply it to all metabolite candidates in the reconstruction; rationale is that a single global offset correction fixes systematic misalignment before NMRQNet's local optimization, so one shift per sample suffices for all metabolites.

## Related tools

- **NMRQNet** (Pipeline whose global positional correction step this skill reproduces; consumes the plasma csv and performs the shift-search alignment of the CRNN reconstruction before local optimization.) — github.com/LiuzLab/NMRQNet
- **NMRQNet EDTA plasma analysis (NMRQNet tutorial)** (Reference notebook (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb) demonstrating the EDTA plasma analysis workflow including this global correction.)

## Examples

```
import pandas as pd; df = pd.read_csv('control_human_plasma.csv'); shifts = range(-150, 151)
```

## Evaluation signals

- global_corrected_spectra.csv exists and retains the same sample columns and chemical shift axis as the input csv.
- For each sample, the applied shift equals the argmax of the summed correlation over the 30 evenly spaced regions across the -150 to +150 step search range.
- Each corrected spectrum corresponds exactly to the CRNN reconstruction shifted by the selected global shift (single shift applied to all metabolite candidates).
- Shifted reconstruction shows higher accumulated regional correlation with the target than the unshifted reconstruction (shift 0).

## Limitations

- The search is bounded to -150 to +150 steps, so offsets beyond that range cannot be corrected.
- A single argmax shift is applied to all metabolite candidates; residual per-metabolite positional and pattern variations are left to subsequent local optimization.
- NMRQNet regularizes less on false-positive predictions, which requires final visualization check to filter metabolites downstream.

## Evidence

- [methods] NMRQNet's global correction selects the shift step whose reconstructed spectrum yields the highest correlation across regions, and applies that shift to all metabolite candidates.: [evidence span withheld — anchor status: not_found]
