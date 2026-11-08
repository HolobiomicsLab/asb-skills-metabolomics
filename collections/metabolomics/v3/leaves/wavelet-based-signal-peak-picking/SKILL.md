---
name: wavelet-based-signal-peak-picking
description: Use when starting from centroid-mode mzML raw LC–MS files (e.g., Waters
  RAW converted with ProteoWizard msconvert) and you need chromatographic/spectrum
  peak picking to build an (Rt, m/z) feature table.
when_to_use_negative:
- Input is already a preprocessed feature/data matrix rather than raw mzML files —
  peak picking on raw scans is unnecessary and would duplicate earlier processing.
- Data are profile-mode or non-centroid spectra where the centroided mzML assumption
  of the MSData read path is violated.
- The goal is feature annotation or downstream statistical analysis rather than raw-data
  peak detection — TidyMS explicitly lacks feature annotation capability, so other
  tools should handle that step.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: TidyMS
  role: Provides the MSData object-oriented interface and the CWT-based peak-picking
    algorithm used for chromatographic and spectrum peak detection, plus the centWave-based
    feature-detection routine
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Python package used to read the centroid-mode mzML raw MS data files into
    TidyMS's MSData interface
- name: Pandas
  role: Used to collate per-sample detections into a single tabular feature matrix
    and save it as feature_table.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:4e700cc156a8c8c569217606ad061f68c0af51914aa5dde77762798956c61df8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Pandas
  - TidyMS
  - pyopenms
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# wavelet-based-signal-peak-picking

## Summary

Applies a continuous wavelet transform (CWT)-based peak-picking algorithm (Du et al., Bioinformatics 2006) via TidyMS's MSData interface to detect chromatographic and spectral peaks in centroid-mode mzML LC–MS raw data, feeding feature detection in untargeted metabolomics.

## When to use

Use when starting from centroid-mode mzML raw LC–MS files (e.g., Waters RAW converted with ProteoWizard msconvert) and you need chromatographic/spectrum peak picking to build an (Rt, m/z) feature table.

## When NOT to use

- Input is already a preprocessed feature/data matrix rather than raw mzML files — peak picking on raw scans is unnecessary and would duplicate earlier processing.
- Data are profile-mode or non-centroid spectra where the centroided mzML assumption of the MSData read path is violated.
- The goal is feature annotation or downstream statistical analysis rather than raw-data peak detection — TidyMS explicitly lacks feature annotation capability, so other tools should handle that step.

## Inputs

- centroid-mode mzML raw LC–MS MS data files (e.g., from MetaboLights MTBLS1919, converted from Waters RAW with ProteoWizard msconvert using min distance = 0.01 and snr = 1)

## Outputs

- tabular feature matrix (feature_table.csv) containing retention time and m/z descriptors for detected features across samples, with features assembled as (Rt, m/z) pairs per sample

## How to apply

Read the centroid-mode mzML files in Python with the pyopenms package through TidyMS's object-oriented MSData interface. Build extracted-ion chromatograms by accumulating consecutive MS scans over the acquired m/z 50–1200 range (0.5 s scan time). Apply the CWT-based peak-picking algorithm (Du et al., Bioinformatics 2006) using the parameter preset optimized for chromatographic data to locate peaks in each chromatogram, then use the centWave-based feature-detection routine (Tautenhahn et al., BMC Bioinformatics 2008) to assemble features as (Rt, m/z) pairs per sample. Collate the per-sample detections into a single tabular feature matrix with Pandas and save it as feature_table.csv. The rationale is that wavelet-based peak picking is robust for chromatographic data where peak shapes vary, and TidyMS reuses the proven centWave algorithm for correspondence.

## Related tools

- **TidyMS** (Provides the MSData object-oriented interface and the CWT-based peak-picking algorithm used for chromatographic and spectrum peak detection, plus the centWave-based feature-detection routine) — https://github.com/griquelme/tidyms
- **pyopenms** (Python package used to read the centroid-mode mzML raw MS data files into TidyMS's MSData interface)
- **Pandas** (Used to collate per-sample detections into a single tabular feature matrix and save it as feature_table.csv)

## Examples

```
from tidyms import msdata; d = msdata.MSData('sample.mzML', mode='centroid'); d.detector = 'ms'; d.peak_detection()
```

## Evaluation signals

- The output feature_table.csv contains retention time and m/z descriptors for detected features across all samples.
- Feature detection completes at the reported speed of about 2 s per sample (66 s total for the 31 raw files of Application 1).
- Peaks are located in extracted-ion chromatograms built from consecutive MS scans over the acquired m/z 50–1200 range.
- Features are assembled as (Rt, m/z) pairs per sample and collated into one tabular feature matrix via Pandas.
- Raw files were read as centroid-mode mzML (converted from Waters RAW with ProteoWizard msconvert, min distance = 0.01, snr = 1), matching the MSData input expectation.

## Limitations

- Feature detection runs at the reported speed of about 2 s per sample, so very large cohorts may require batching.
- Peak picking is tuned for chromatographic data; spectral peak picking may need different parameter presets.
- TidyMS lacks feature annotation capability, so this skill only produces detected features, not annotated metabolites.
- m/z range (50–1200) and scan time (0.5 s) were specific to the reported LC–MS acquisition; other instruments may need recalibration of these build-EIC parameters.

## Evidence

- [results] TidyMS performs raw-data peak analysis through the MSData object, detecting chromatographic and spectrum peaks with a continuous wavelet transform (CWT)-based peak picking algorithm: [evidence span withheld — anchor status: not_found]
- [methods] The procedure included retention time alignment, peak picking, deisotoping, integration, and grouping together adducts derived from the same compound.: "The procedure included retention time alignment, peak picking, deisotoping, integration, and grouping together adducts derived from the same compound."
- [methods] The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919.: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
- [results] The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s.: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [results] The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min.: "The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min."
