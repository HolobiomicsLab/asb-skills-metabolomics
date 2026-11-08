---
name: lcms-eic-extraction
description: Use when raw LC/MS mzML files and a generic peak list (mz, rt columns,
  e.g. from XCMS) must be converted into per-peak EIC matrices to feed peak-classification
  or reintegration models.
when_to_use_negative:
- When the input is already a curated feature/peak-intensity table rather than raw
  mzML data — EIC extraction requires raw scans.
- When peaks are absent, i.e. no peak list (mz, rt) exists — run peak detection such
  as XCMS first to generate the peak list.
- When the goal is artifact detection with a different model architecture that expects
  other EIC resolutions or window sizes, since these parameters are fixed by the trained
  PeakDetective networks.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakDetective
  role: Provides rawData.readRawDataFile for loading mzML at 25.5 ppm MS1 tolerance
    and makeDataMatrix for building the n x 60 EIC matrix with RT alignment
  repo: https://github.com/pattilab/PeakDetective
- name: XCMS
  role: Upstream R-based peak detection that generates the mz/rt peak list consumed
    by EIC extraction
- name: pyteomics
  role: Required dependency for parsing raw mass-spectrometry data files during rawData
    loading
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:4b8496c5d5e9610e59b3ebc6bce1d37df6df7748a4a3248a4a4290697630b93e
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# lcms-eic-extraction

## Summary

Extracts ion chromatograms (EICs) from raw LC/MS mzML files around each peak's m/z and retention time, producing a fixed-size n x 60 data matrix suitable for downstream deep-learning peak curation in PeakDetective.

## When to use

Use when raw LC/MS mzML files and a generic peak list (mz, rt columns, e.g. from XCMS) must be converted into per-peak EIC matrices to feed peak-classification or reintegration models.

## When NOT to use

- When the input is already a curated feature/peak-intensity table rather than raw mzML data — EIC extraction requires raw scans.
- When peaks are absent, i.e. no peak list (mz, rt) exists — run peak detection such as XCMS first to generate the peak list.
- When the goal is artifact detection with a different model architecture that expects other EIC resolutions or window sizes, since these parameters are fixed by the trained PeakDetective networks.

## Inputs

- Raw LC/MS mzML files (one or more per sample)
- Peak list CSV with mz and rt columns (e.g. XCMS-generated peaks_formatted.csv)
- ms1ppm tolerance (25.5 ppm used in the paper)
- align flag for retention-time alignment (True/False)

## Outputs

- n x 60 EIC data matrix (60 sampled datapoints per 1-min EIC window per peak, per sample)
- Populated PeakDetective.rawData objects holding parsed raw MS1 data

## How to apply

Load each mzML file into a PeakDetective.rawData object with readRawDataFile using a 25.5 ppm MS1 tolerance, and read the peak list keeping only the mz and rt columns. Initialize a PeakDetective object (numCores=10, resolution=60) and call makeDataMatrix with the peak mz/rt arrays, an EIC window of 1.0 min sampled at 60 data points per minute, and align=True to perform retention-time alignment across samples. Each peak's EIC is concatenated into an n x 60 matrix (n peaks), which serves as the input for training the smoother autoencoder and classifier or for reintegrating artifact peaks. The 1.0-min window with 60 points/min is a model parameter the README marks as 'not recommended to change', so it should be preserved unless retraining the networks.

## Related tools

- **PeakDetective** (Provides rawData.readRawDataFile for loading mzML at 25.5 ppm MS1 tolerance and makeDataMatrix for building the n x 60 EIC matrix with RT alignment) — https://github.com/pattilab/PeakDetective
- **XCMS** (Upstream R-based peak detection that generates the mz/rt peak list consumed by EIC extraction)
- **pyteomics** (Required dependency for parsing raw mass-spectrometry data files during rawData loading)

## Examples

```
X = integ.makeDataMatrix(raw_data, peaklist["mz"].values, peaklist["rt"].values, align=True)
```

## Evaluation signals

- The resulting data matrix has exactly 60 columns per peak (resolution=60 data points in the 1-min EIC window) and n rows equal to the number of peaks in the peak list.
- All mzML files in the folder were loaded without error and each raw_data entry is a PeakDetective.rawData object.
- Downstream trainSmoother / trainClassifierActive runs accept the matrix without dimension errors, confirming the EIC window (1.0 min) and alignment were applied consistently.
- With align=True, EICs across samples are retention-time aligned, which can be sanity-checked by visually inspecting a few EICs for peak coincidence.

## Limitations

- resolution=60 and window=1.0 are model parameters the README says are 'not recommended to change', so EICs extracted at other resolutions may not be compatible with pretrained PeakDetective weights.
- EIC quality depends on the 25.5 ppm MS1 tolerance matching the instrument's mass accuracy; a mismatch can truncate or distort extracted ion chromatograms.
- A pre-existing peak list is required; the skill cannot discover peaks de novo (XCMS detection with s2n=13.6, prefilter=5, etc. is a separate step).

## Evidence

- [supplementary] EICs are extracted and concatenated into matrix: [evidence span withheld — anchor status: not_found]
