---
name: metabolomics-figure-s10-reproduction
description: Use when evaluating whether artifact-aware peak reintegration improves
  LC/MS peak boundary definition — i.e., when comparing pre- vs post-reintegration
  peak-area CV distributions for feature lists scored by a trained PeakDetective classifier.
when_to_use_negative:
- When the goal is peak detection or artifact classification itself (training the
  autoencoder/classifier via active learning) rather than reintegration-based CV evaluation
  of an already-scored peak list
- When raw mzML files are unavailable — reintegration requires extracting EICs from
  raw data, so a peak-intensity table alone is insufficient
- When peak boundaries are already well-defined (e.g., peaks flagged as true peaks
  at cutoff 0.8, which are not re-integrated)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PeakDetective
  role: Provides rawData loading, makeDataMatrix EIC extraction, findPeakBoundaries/integratePeak
    reintegration, and the trained classifier scores used to identify artifact peaks
  repo: https://github.com/pattilab/PeakDetective
- name: XCMS
  role: Supplies the pre-generated peak list whose artifact peaks are scored and re-integrated
- name: numpy
  role: Computes per-feature coefficient of variation (SD/mean) of peak areas before
    and after reintegration
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:df63cdf7c14aaea936d0034839d748d29fa77935537ca7d101c60f5525eb1de1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - PeakDetective
  - Python
  - XCMS
  - numpy
  - pyteomics
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metabolomics-figure-s10-reproduction

## Summary

Reproduces the PeakDetective artifact-aware peak reintegration analysis: artifact peaks from an XCMS peak list are re-integrated from raw mzML data using mean peak boundaries learned from non-artifact samples, and the resulting reduction in peak-area coefficient of variation (CV) across Datasets 1–3 is quantified and plotted as Figure S10.

## When to use

Use when evaluating whether artifact-aware peak reintegration improves LC/MS peak boundary definition — i.e., when comparing pre- vs post-reintegration peak-area CV distributions for feature lists scored by a trained PeakDetective classifier.

## When NOT to use

- When the goal is peak detection or artifact classification itself (training the autoencoder/classifier via active learning) rather than reintegration-based CV evaluation of an already-scored peak list
- When raw mzML files are unavailable — reintegration requires extracting EICs from raw data, so a peak-intensity table alone is insufficient
- When peak boundaries are already well-defined (e.g., peaks flagged as true peaks at cutoff 0.8, which are not re-integrated)

## Inputs

- Raw mzML files for Datasets 1–3 (LC/MS data)
- Pre-generated XCMS peak list (mz, rt) formatted as CSV
- Trained PeakDetective model weights (PeakDetectiveObject directory)
- Classifier score cutoff (0.8) and peak labels (artifact vs true peak)

## Outputs

- Reintegrated peak areas per feature saved to CSV
- Per-feature CV values before and after reintegration (CV = SD/mean)
- figure_s10_cv_comparison.png — CV distribution plots before vs. after reintegration for Datasets 1–3

## How to apply

Load raw mzML files from Datasets 1–3 into PeakDetective.rawData objects with an MS1 tolerance of 25.5 ppm, then score a pre-generated XCMS peak list with the trained PeakDetective classifier at cutoff = 0.8 to label each peak as artifact or true. For artifact peaks, extract EICs with makeDataMatrix (resolution = 60 points per minute, window = 1.0 min), derive mean peak boundaries from non-artifact samples of the same feature, and re-integrate the artifact EICs with PeakDetective's findPeakBoundaries/integratePeak using defaultWidth = 0.5 min and smooth = False (analogous to Warpgroup). For every feature, compute CV = SD/mean of peak areas before and after reintegration using numpy/pandas, plot the CV distributions per dataset, and save the reintegrated areas to CSV plus the comparison plot as figure_s10_cv_comparison.png. The rationale is that artifact peaks have poorly defined boundaries, so borrowing boundary statistics from non-artifact samples of the same feature should tighten integration and lower within-feature area variability.

## Related tools

- **PeakDetective** (Provides rawData loading, makeDataMatrix EIC extraction, findPeakBoundaries/integratePeak reintegration, and the trained classifier scores used to identify artifact peaks) — https://github.com/pattilab/PeakDetective
- **XCMS** (Supplies the pre-generated peak list whose artifact peaks are scored and re-integrated)
- **numpy** (Computes per-feature coefficient of variation (SD/mean) of peak areas before and after reintegration)

## Examples

```
X = integ.makeDataMatrix(raw_data, peaklist["mz"].values, peaklist["rt"].values, align=True)
```

## Evaluation signals

- Post-reintegration peak-area CV distribution is shifted lower than the pre-reintegration distribution in all three datasets (qualitative check, matching Figure S10; no numeric CV threshold is reported)
- Only peaks scored as artifacts at cutoff = 0.8 are re-integrated; true peaks retain their original XCMS areas
- EICs are built at resolution = 60 points/min with a 1.0 min window, and reintegration uses defaultWidth = 0.5 min with smooth = False
- CV is computed as SD/mean per feature, paired before/after for the same feature identity across Datasets 1–3
- Outputs include a CSV of reintegrated areas and figure_s10_cv_comparison.png

## Limitations

- Validation is qualitative — the paper reports no numeric CV threshold, so success is judged by visual downward shift of the CV distribution
- Reintegration quality depends on having non-artifact samples of the same feature to derive mean peak boundaries; features with few good samples may not benefit
- Classifier cutoff (0.8) is user-set based on FDR/TPR preferences, so artifact labels — and hence which peaks are re-integrated — depend on that choice

## Evidence

- [supplementary] Artifact-aware peak reintegration enables peak boundaries to be better defined, decreasing the CV of peak areas across Datasets 1–3: [evidence span withheld — anchor status: not_found]
