---
name: warpgroup-style-mean-boundary-peak-reintegration
description: Use when LC/MS feature peak boundaries are inconsistent across samples
  (artifacts flagged by PeakDetective) and you want to improve boundary definition
  and reduce peak-area CV.
when_to_use_negative:
- If the classifier scores all peaks of a feature as true peaks in every sample, there
  are no artifact boundaries to fix and reintegration is unnecessary.
- If you only need peak classification/filtering rather than improved boundary definition
  and reduced CV, curatePeaks alone suffices without reintegration.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakDetective
  role: Provides rawData/makeDataMatrix for EIC extraction and the peak-boundary/integration
    functions used to reintegrate artifact peaks using mean boundaries from non-artifact
    samples
  repo: https://github.com/pattilab/PeakDetective
- name: pyteomics
  role: Required package used by PeakDetective to read raw mzML data
- name: numpy
  role: Used to compute coefficient of variation (SD/mean) of peak areas before and
    after reintegration
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:e9d686b1f18b4e041bede89e0a8de21560895540aa31553c48adad5dbf006e3b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - PeakDetective
  - Python
  - numpy
  - pyteomics
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# warpgroup-style-mean-boundary-peak-reintegration

## Summary

Reintegrates peaks labeled as artifacts using mean peak boundaries derived from non-artifact samples of the same feature, analogous to Warpgroup, to better define peak boundaries and reduce coefficient of variation of peak areas in LC/MS metabolomics.

## When to use

Use when LC/MS feature peak boundaries are inconsistent across samples (artifacts flagged by PeakDetective) and you want to improve boundary definition and reduce peak-area CV.

## When NOT to use

- If the classifier scores all peaks of a feature as true peaks in every sample, there are no artifact boundaries to fix and reintegration is unnecessary.
- If you only need peak classification/filtering rather than improved boundary definition and reduced CV, curatePeaks alone suffices without reintegration.

## Inputs

- Raw LC/MS mzML files
- XCMS-generated peak list (mz, rt columns)
- Trained PeakDetective classifier with artifact/true-peak scores (cutoff = 0.8)

## Outputs

- Reintegrated peak areas per feature per sample (improved boundary definition)
- Per-feature CV of peak areas before and after reintegration
- CV comparison plot (e.g. figure_s10_cv_comparison.png)

## How to apply

Load raw mzML files into PeakDetective.rawData objects with ms1ppm = 25.5 ppm tolerance, generate EICs with makeDataMatrix (resolution = 60 points per minute, window = 1.0 min, align = True), and score the XCMS-generated peak list with the trained classifier at cutoff = 0.8 to label peaks as artifacts or true peaks. For each artifact peak, extract EICs and derive mean peak boundaries from the non-artifact samples of the same feature, then reintegrate the artifact EICs with PeakDetective's peak-boundary/integration functions (defaultWidth = 0.5 min, smooth = False), mirroring the Warpgroup approach. The rationale is that a true peak's shape observed in non-artifact samples provides a consensus boundary template that fixes poorly defined boundaries in artifact samples. Afterwards, compute the coefficient of variation (CV = SD/mean) of peak areas per feature before and after reintegration to quantify boundary improvement, as demonstrated across Datasets 1-3 (Figure S10).

## Related tools

- **PeakDetective** (Provides rawData/makeDataMatrix for EIC extraction and the peak-boundary/integration functions used to reintegrate artifact peaks using mean boundaries from non-artifact samples) — https://github.com/pattilab/PeakDetective
- **pyteomics** (Required package used by PeakDetective to read raw mzML data)
- **numpy** (Used to compute coefficient of variation (SD/mean) of peak areas before and after reintegration)

## Examples

```
from PeakDetective import PeakDetective as PD; integ = PD(numCores=10, resolution=60); X = integ.makeDataMatrix(raw_data, peaklist['mz'].values, peaklist['rt'].values, align=True)
```

## Evaluation signals

- Post-reintegration peak-area CV distribution is shifted lower than the pre-reintegration distribution across Datasets 1-3, consistent with Figure S10
- Per-feature CV (SD/mean) computed with numpy/pandas decreases after reintegration
- Artifact peaks identified by classifier cutoff = 0.8 are all successfully reintegrated with boundaries derived from non-artifact samples
- Reintegrated peak areas are saved to CSV with mz, rt, and per-sample intensity columns

## Limitations

- The paper reports a qualitative check (shift in CV distribution) rather than a numeric CV threshold, so success is judged by comparing CV distributions before vs. after
- Reintegration requires at least some non-artifact samples of the same feature to derive mean boundaries; if all samples for a feature are artifacts, no mean template exists
- EIC extraction is at 60 points/min with a 1.0 min window; if the chromatographic peak is narrower or wider than this window, boundaries may be poorly resolved

## Evidence

- [results] Artifact-aware peak reintegration enables peak boundaries to be better defined, decreasing the CV of peak areas across Datasets 1-3, as shown in Figure S10.: [evidence span withheld — anchor status: not_found]
