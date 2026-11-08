---
name: optical-correction-mode-accuracy-quantification
description: Use when you have paired ablation-mark coordinate files and macroMS geometry
  files from multiple imaging modes and need to measure how perspective transformation
  or distortion correction affects MALDI-TOF targeting accuracy in μm.
when_to_use_negative:
- Samples smaller than 300 μm, where the article states microMS is the more appropriate
  optically guided MS workflow
- Only raw MALDI mass spectra are available without the paired ablation-mark coordinate
  files and macroMS geometry files needed to compute spatial offsets
- The geometry metadata lacks the imaging resolution needed to convert device pixel/step
  units to micrometers
tools:
- name: macroMS
  role: Source of the optical images, correction tools (perspective transformation
    and distortion correction), and the output geometry files whose target positions
    are compared against measured ablation-mark coordinates
  repo: github.com/macroMS/macroMS_web
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:48c61b74ce3dfb7d76ff94843de5a3ba898179b97d4767f6dc862d47d32c8083
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# optical-correction-mode-accuracy-quantification

## Summary

Quantifies the targeting accuracy of macroMS imaging modes (flatbed scanner vs. cell-phone camera with perspective transformation and/or optical distortion correction) by measuring Euclidean offsets between intended target positions and actual ablation-mark centers in micrometers.

## When to use

Use when you have paired ablation-mark coordinate files and macroMS geometry files from multiple imaging modes and need to measure how perspective transformation or distortion correction affects MALDI-TOF targeting accuracy in μm.

## When NOT to use

- Samples smaller than 300 μm, where the article states microMS is the more appropriate optically guided MS workflow
- Only raw MALDI mass spectra are available without the paired ablation-mark coordinate files and macroMS geometry files needed to compute spatial offsets
- The geometry metadata lacks the imaging resolution needed to convert device pixel/step units to micrometers

## Inputs

- Deposited PACII ablation-mark coordinate files (measured mark centers)
- macroMS output geometry files per imaging mode (intended target positions)
- Imaging resolution metadata from macroMS geometry files (μm per pixel/step)
- Mode labels: flatbed scanner, phone perspective-only, distortion-corrected, laser-shift, combined correction

## Outputs

- Per-spot Euclidean offsets in μm for each imaging mode
- Per-mode mean ± SD targeting-accuracy values
- accuracy_summary.csv with the per-mode accuracy table

## How to apply

Download the deposited PACII ablation-mark coordinate files and corresponding macroMS output geometry files for each imaging mode (flatbed scanner, phone mode with distortion correction, laser-shift correction, and combined correction). Parse each pair with Python (pandas/numpy) to extract intended target positions from the geometry files and actual ablation-mark centers from the coordinate files. Convert device pixel/step units to micrometers using the imaging resolution stated in the macroMS geometry metadata, then compute the Euclidean offset between each intended position and its measured ablation mark. Aggregate offsets per mode into mean ± SD and compare against the reported benchmarks (scanner 115 ± 40 μm; phone modes 115 ± 65, 165 ± 95, 80 ± 15, and 30 ± 15 μm for perspective-only, perspective-plus, distortion-corrected, and combined-correction configurations). Write the per-mode summary to accuracy_summary.csv. The key decision point is which correction pipeline was applied per mode, since adding distortion correction to perspective transformation improved accuracy from 165 ± 95 μm (15 of 46 spots off-target or near miss) to 80 ± 15 μm with all 46 spots correctly targeted.

## Related tools

- **macroMS** (Source of the optical images, correction tools (perspective transformation and distortion correction), and the output geometry files whose target positions are compared against measured ablation-mark coordinates) — github.com/macroMS/macroMS_web

## Examples

```
import pandas as pd, numpy as np; off = np.linalg.norm(target_xy - mark_xy, axis=1) * um_per_px; print(f'{off.mean():.0f} +/- {off.std():.0f} um')
```

## Evaluation signals

- Perspective-transformation-only phone mode reproduces 165 ± 95 μm mean accuracy with 15 of 46 spots off-target or near miss
- Adding distortion correction reproduces 80 ± 15 μm with all 46 spots correctly targeted
- Flatbed scanner mode reproduces 115 ± 40 μm; combined correction reproduces 30 ± 15 μm
- Offsets are reported in μm, not raw device pixels/step units — verify the unit conversion against geometry metadata resolution
- accuracy_summary.csv contains one mean ± SD row per imaging mode covering all five reported modes

## Limitations

- macroMS version information is not stated in the article
- Sample targeting reliability is only stated for 300 μm samples with no detail on replicates or statistical treatment
- The ~10,000-target upper limit is presented without measurement details or uncertainty
- Comparison to microMS is qualitative with no quantitative benchmark for small samples

## Evidence
