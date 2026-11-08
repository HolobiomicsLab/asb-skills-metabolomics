---
name: macroms-geometry-file-interoperability
description: Use when converting target positions picked from flatbed-scanner or cell-phone
  optical images into macroMS geometry files for MALDI-TOF acquisition, or when quantifying
  targeting accuracy in micrometers from geometry/ablation-mark file pairs.
when_to_use_negative:
- Samples smaller than 300 μm — the article states microMS is more effective for those
- Analysis of the acquired spectra themselves (data analysis/visualization of mass
  spectra) rather than target-to-geometry coordinate interoperability
- Targets requiring microscope-grade imaging precision; this workflow intentionally
  uses low-cost scanner/phone images with only moderate-distortion correction
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: macroMS
  role: Web platform performing sample finding, target filtering, optical correction
    (perspective + distortion), and device coordinate (geometry file) generation
  repo: https://github.com/macroMS/macroMS_web
- name: microMS
  role: Alternative optically guided MS tool for samples smaller than 300 μm; used
    here only as the scope boundary for macroMS geometry generation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:e6341c735d5396ccfed93afaa3b9403859a76e71f516946f0973cdc74c90fbb0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/macroMS/macroMS_web
  tools:
  - macroMS
  - microMS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# macroms-geometry-file-interoperability

## Summary

Interoperate with macroMS's custom device geometry files: generate device coordinates from optically found targets via perspective transformation and optical distortion correction, and validate targeting accuracy by computing Euclidean offsets between intended positions and measured ablation marks.

## When to use

Use when converting target positions picked from flatbed-scanner or cell-phone optical images into macroMS geometry files for MALDI-TOF acquisition, or when quantifying targeting accuracy in micrometers from geometry/ablation-mark file pairs.

## When NOT to use

- Samples smaller than 300 μm — the article states microMS is more effective for those
- Analysis of the acquired spectra themselves (data analysis/visualization of mass spectra) rather than target-to-geometry coordinate interoperability
- Targets requiring microscope-grade imaging precision; this workflow intentionally uses low-cost scanner/phone images with only moderate-distortion correction

## Inputs

- Optical images of macroscopic (>300 μm) samples from a flatbed scanner or cell phone camera
- Target/geometry files output by the macroMS web platform (intended target positions with device pixel/step units and imaging-resolution metadata)
- Deposited ablation-mark coordinate files recording measured mark centers per imaging mode
- Fiducial marks used for perspective transformation and distortion correction

## Outputs

- macroMS custom geometry file consumable by the MS device for targeted MALDI-TOF acquisition
- Per-target Euclidean offsets in μm between intended positions and measured ablation marks
- Per-imaging-mode accuracy summary (mean ± SD, e.g. accuracy_summary.csv)

## How to apply

Pick and filter targets from the optical image in the macroMS web platform, which generates a custom geometry file that is fed to the MS device for acquisition at the selected locations. Because common imaging devices introduce moderate distortion and imperfect perspective, apply the macroMS optical correction tools — perspective transformation plus distortion correction — before generating coordinates; the task card shows that perspective transformation alone on 460-μm PACII matrix spots gives 165 ± 95 μm accuracy (15 of 46 spots off-target), while adding distortion correction yields 80 ± 15 μm with all 46 spots on-target. To verify interoperability, parse each geometry-file/ablation-mark-coordinate pair with pandas/numpy, convert device pixel or step units to micrometers using the imaging resolution in the geometry metadata, and compute per-mode mean ± SD Euclidean offsets (expected: ~115 ± 40 μm scanner; 115 ± 65, 165 ± 95, 80 ± 15, 30 ± 15 μm for phone modes). Note the platform's throughput budget: coordinate generation and acquisition run at 1–2 s per sample for up to ~10,000 targets.

## Related tools

- **macroMS** (Web platform performing sample finding, target filtering, optical correction (perspective + distortion), and device coordinate (geometry file) generation) — https://github.com/macroMS/macroMS_web
- **microMS** (Alternative optically guided MS tool for samples smaller than 300 μm; used here only as the scope boundary for macroMS geometry generation)

## Examples

```
import pandas as pd, numpy as np; tgt = pd.read_csv('geometry.csv'); mk = pd.read_csv('marks.csv'); d = np.linalg.norm(tgt[['x_um','y_um']].values - mk[['x_um','y_um']].values, axis=1); print(d.mean(), d.std())
```

## Evaluation signals

- With perspective transformation plus distortion correction on 460-μm PACII spots, all 46 targets are correctly hit (accuracy 80 ± 15 μm); if 15 of 46 are off-target (165 ± 95 μm), distortion correction was omitted
- Per-mode mean ± SD Euclidean offsets reproduce reported values: scanner 115 ± 40 μm; phone modes 115 ± 65, 165 ± 95, 80 ± 15, and 30 ± 15 μm
- Pixel/step-to-micrometer conversion is consistent with the imaging resolution stated in the macroMS geometry metadata
- Geometry file is accepted by the MS device and spectra are acquired at the selected locations at 1–2 s per sample
- Target count stays within the platform's ~10,000-target upper limit

## Limitations

- Optical correction only handles moderate distortion and imperfect perspective inherent to common imaging devices; severe optical defects are out of scope
- Perspective transformation alone leaves substantial error (165 ± 95 μm, 15 of 46 spots off-target on 460-μm PACII matrix spots)
- Upper limit of ~10,000 targets, presented without measurement details or uncertainty
- Targeting reliability for 300-μm samples is stated without replicate or statistical detail; microMS is preferred below 300 μm

## Evidence
