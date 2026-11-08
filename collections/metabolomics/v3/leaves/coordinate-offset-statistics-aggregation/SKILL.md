---
name: coordinate-offset-statistics-aggregation
description: Use when you have paired files of intended target coordinates (device
  geometry output) and measured ablation-mark positions, and need per-mode targeting-accuracy
  statistics in micrometers.
when_to_use_negative:
- Samples smaller than 300 μm, where microMS is the more effective optically guided
  MS approach
- Input is a single image or mass spectrum without paired intended/measured coordinate
  files — no offset can be computed
- Raw mass-spectra analysis tasks (compound identification, visualization of acquired
  spectra) rather than targeting-accuracy assessment
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: macroMS
  role: Generates the device geometry files and fiducial/correction workflow whose
    targeting accuracy is quantified; its optical correction tools (perspective and
    distortion) are the variables being evaluated
  repo: github.com/macroMS/macroMS_web
- name: microMS
  role: Alternative optically guided MS suite for smaller samples; relevant as the
    scope boundary (<300 μm) for this macro-scale accuracy workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:45c4e54a0eeb0bb099769db283e4c04444a9a25b035554a0c2089e201c7600f9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  - microMS
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# coordinate-offset-statistics-aggregation

## Summary

Quantifies image-guided targeting accuracy by computing per-target Euclidean offsets (μm) between intended MALDI-ablation positions and measured ablation-mark centers, then aggregating mean ± SD per imaging mode for comparison against macroMS-reported benchmarks (e.g., 165 ± 95 μm vs 80 ± 15 μm after distortion correction).

## When to use

Use when you have paired files of intended target coordinates (device geometry output) and measured ablation-mark positions, and need per-mode targeting-accuracy statistics in micrometers.

## When NOT to use

- Samples smaller than 300 μm, where microMS is the more effective optically guided MS approach
- Input is a single image or mass spectrum without paired intended/measured coordinate files — no offset can be computed
- Raw mass-spectra analysis tasks (compound identification, visualization of acquired spectra) rather than targeting-accuracy assessment

## Inputs

- Deposited PACII ablation-mark coordinate files (measured mark centers) per imaging mode
- macroMS output geometry files (intended target positions) per imaging mode (scanner, phone modes, laser-shift correction, combined correction)
- Imaging resolution metadata from the macroMS geometry files for μm-per-pixel/step conversion

## Outputs

- Per-target Euclidean offsets in μm for each imaging mode
- Per-mode mean ± standard deviation targeting accuracy (e.g., 80 ± 15 μm)
- accuracy_summary.csv — summary table of per-mode mean ± SD values

## How to apply

Download the deposited ablation-mark coordinate files and the corresponding macroMS output geometry files for each imaging mode (flatbed scanner, phone modes with/without distortion correction, laser-shift correction, and combined correction). Parse each pair with pandas/numpy to extract intended target positions from the geometry files and actual ablation-mark centers from the coordinate files. For each mode, compute the Euclidean offset in μm between intended and measured positions, converting device pixel/step units to micrometers using the imaging resolution stated in the macroMS geometry metadata. Aggregate offsets per mode into mean ± SD, then compare against the reported benchmarks (scanner 115 ± 40 μm; phone modes 115 ± 65, 165 ± 95, 80 ± 15, and 30 ± 15 μm; e.g., perspective transformation alone gives 165 ± 95 μm with 15 of 46 spots off-target or near miss, while adding distortion correction yields 80 ± 15 μm with all 46 spots correctly targeted). Assemble the per-mode values into a summary table (accuracy_summary.csv).

## Related tools

- **macroMS** (Generates the device geometry files and fiducial/correction workflow whose targeting accuracy is quantified; its optical correction tools (perspective and distortion) are the variables being evaluated) — github.com/macroMS/macroMS_web
- **microMS** (Alternative optically guided MS suite for smaller samples; relevant as the scope boundary (<300 μm) for this macro-scale accuracy workflow)

## Examples

```
import pandas as pd, numpy as np; d = pd.read_csv('targets.csv'); off = np.hypot(d.x_int-d.x_meas, d.y_int-d.y_meas)*UM_PER_PX; print(f'{off.mean():.0f} ± {off.std():.0f} μm')
```

## Evaluation signals

- Per-mode mean ± SD offsets reproduce the reported benchmarks within their stated uncertainties (scanner 115 ± 40 μm; phone 115 ± 65, 165 ± 95, 80 ± 15, 30 ± 15 μm)
- Distortion-corrected phone mode should target all 46 PACII matrix spots with mean ≈ 80 μm and SD ≈ 15 μm (no off-target or near-miss spots)
- Perspective-transformation-only mode should show ~15 of 46 spots off-target or near miss, consistent with 165 ± 95 μm
- Unit conversion sanity check: offsets in μm must be derived using the imaging resolution in the geometry metadata, not raw device pixels
- accuracy_summary.csv contains one mean ± SD row per imaging mode covering all five reported conditions

## Limitations

- Version information for macroMS is not stated, so reproducibility is tied to the web platform rather than a pinned release
- The ~10,000-target upper limit is presented without measurement details or uncertainty, so scaling beyond ~46-spot validation sets is unverified
- Reliable targeting is validated at 300 μm sample size with no detail on replicates or statistical treatment beyond per-mode mean ± SD
- Comparison to microMS is qualitative with no quantitative benchmark, so accuracy gains over the micro-scale tool cannot be quantified from this data

## Evidence

- [abstract] correct optical distortion issues for improved spatial location of selected targets: "correct optical distortion issues for improved spatial location of selected targets"
- [intro] new optical correction tools were added to resolve optical defects inherent to common imaging devices: "new optical correction tools were added to resolve optical defects inherent to common imaging devices, including correction of moderate distortion and imperfect perspective"
- [discussion] macroMS also provides correction tools for obtaining accurate locations of samples from images with moderate optical distortion: "macroMS also provides correction tools for obtaining accurate locations of samples from images with moderate optical distortion and imperfect viewpoints"
