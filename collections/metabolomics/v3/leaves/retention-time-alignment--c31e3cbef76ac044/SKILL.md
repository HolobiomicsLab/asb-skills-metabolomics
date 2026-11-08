---
name: retention-time-alignment
description: Use when raw LC/MS mzML files from multiple injections are being converted
  into EIC data matrices for downstream peak scoring, and retention-time drift between
  runs could misalign the 60-datapoint chromatographic windows for the same peak.
when_to_use_negative:
- Input is already a feature/peak-intensity table with no raw chromatographic data
  — alignment of EICs is not applicable
- Only a single mzML file is analyzed and no cross-run RT drift correction is needed
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakDetective
  role: Provides rawData.readRawDataFile and makeDataMatrix(raw_data, mz, rt, align=True)
    which performs the retention-time alignment while building the EIC data matrix
  repo: https://github.com/pattilab/PeakDetective
- name: XCMS
  role: Generates the input peak list (mz, rt) whose retention times are aligned across
    samples during matrix construction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:2825999554c68c5c9060f64e40ea8959d7069d552a52eee77d6064133a670df0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - PeakDetective
  - Python
  - TensorFlow
  - XCMS
  - numpy
  - pyteomics
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# retention-time-alignment

## Summary

Within the PeakDetective peak-curation pipeline, retention-time alignment is toggled when building the EIC data matrix so that each peak's extracted ion chromatogram is aligned across raw mzML files before autoencoder compression and classification.

## When to use

Use when raw LC/MS mzML files from multiple injections are being converted into EIC data matrices for downstream peak scoring, and retention-time drift between runs could misalign the 60-datapoint chromatographic windows for the same peak.

## When NOT to use

- Input is already a feature/peak-intensity table with no raw chromatographic data — alignment of EICs is not applicable
- Only a single mzML file is analyzed and no cross-run RT drift correction is needed

## Inputs

- List of PeakDetective.rawData objects loaded from mzML files (25.5 ppm MS1 tolerance)
- Peak list with mz and rt columns (e.g. XCMS peaks_formatted.csv)

## Outputs

- Aligned n x 60 EIC data matrix (X), 60 sampled datapoints per 1-minute EIC window per peak

## How to apply

After loading each mzML file into a PeakDetective.rawData object with a 25.5 ppm MS1 tolerance and loading the XCMS-generated peak list (mz, rt columns), initialize the PeakDetective object with numCores=10 and resolution=60. Call makeDataMatrix with the peak m/z and rt values, an EIC window of 1.0 minute sampled at 60 data points, and align=True so that retention-time alignment is performed before the EICs are stacked into the n x 60 matrix. The aligned matrix (X) then feeds trainSmoother (min_peaks=100000, smooth_epochs=10, batchSizeSmoother=64, validationSplitSmoother=0.1) and the active-learning classifier, so misalignment would corrupt the latent representation each EIC encodes. If chromatographic separation is highly reproducible, alignment can be skipped with align=False, but the default workflow in the paper's notebook sets align=True.

## Related tools

- **PeakDetective** (Provides rawData.readRawDataFile and makeDataMatrix(raw_data, mz, rt, align=True) which performs the retention-time alignment while building the EIC data matrix) — https://github.com/pattilab/PeakDetective
- **XCMS** (Generates the input peak list (mz, rt) whose retention times are aligned across samples during matrix construction)

## Examples

```
integ = PeakDetective.PeakDetective(numCores=10, resolution=60); X = integ.makeDataMatrix(raw_data, peaklist['mz'].values, peaklist['rt'].values, align=True)
```

## Evaluation signals

- makeDataMatrix returns an n x 60 matrix (resolution=60, window=1.0 min) with one row per peak in the input peak list
- align=True was passed so every EIC window is RT-aligned before training; verify via the integ object configuration
- Downstream smoother training (trainSmoother) converges using min_peaks=100000 EICs without anomalies that would suggest misaligned windows
- Score histogram from active learning becomes bimodal after 5-10 rounds, indicating EICs are consistently represented

## Limitations

- Alignment operates on EIC windows of fixed 1.0-minute width at resolution=60; peaks outside the sampled window or with drift beyond the window will be poorly represented
- The README marks resolution, window, and related model parameters as 'not recomended to change'

## Evidence

- [abstract] EIC concatenation: [evidence span withheld — anchor status: not_found]
