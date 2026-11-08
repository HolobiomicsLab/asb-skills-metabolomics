---
name: lc-ms-feature-detection-centwave
description: Use when starting from centroid-mode mzML raw LC–MS files and per-sample
  chromatographic/spectral peaks must be located and converted into (retention time,
  m/z) features for downstream feature matrices.
when_to_use_negative:
- Input is already an extracted feature table — this skill operates on raw centroid-mode
  mzML data before correspondence
- The task requires feature annotation — TidyMS lacks feature annotation capability
  per the article's comparison table
- Data are profile-mode RAW vendor files that have not been converted (e.g., with
  ProteoWizard msconvert) to centroid mzML first
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: TidyMS
  role: Python package whose MSData interface performs CWT-based peak picking and
    centWave-based feature detection on raw LC–MS data
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Underlying reader used through TidyMS's MSData object-oriented interface to
    read raw MS data in Python
- name: Python
  role: Language in which the feature detection pipeline is executed (with Pandas
    for collating the feature matrix)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:c086443f04d4057fef23c07d901f6fb00b9a19150ce0ec895009407d5abd6f09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Python
  - TidyMS
  - pyopenms
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lc-ms-feature-detection-centwave

## Summary

Detects chromatographic and spectral peaks in centroid-mode raw LC–MS data (mzML) using TidyMS's CWT-based peak-picking algorithm and centWave-style feature detection, assembling (Rt, m/z) features into a feature table for untargeted metabolomics preprocessing.

## When to use

Use when starting from centroid-mode mzML raw LC–MS files and per-sample chromatographic/spectral peaks must be located and converted into (retention time, m/z) features for downstream feature matrices.

## When NOT to use

- Input is already an extracted feature table — this skill operates on raw centroid-mode mzML data before correspondence
- The task requires feature annotation — TidyMS lacks feature annotation capability per the article's comparison table
- Data are profile-mode RAW vendor files that have not been converted (e.g., with ProteoWizard msconvert) to centroid mzML first

## Inputs

- Centroid-mode mzML raw MS data files (e.g., Waters RAW converted with ProteoWizard msconvert, min distance = 0.01, snr = 1)
- MetaboLights public repository data set identifier MTBLS1919 as a concrete example source

## Outputs

- Detected chromatographic and spectrum peaks per sample from the MSData object
- Per-sample features as (Rt, m/z) pairs
- Collated tabular feature matrix (e.g., feature_table.csv) with retention time and m/z descriptors across samples

## How to apply

Read the centroid-mode mzML files (e.g., from MetaboLights data set MTBLS1919) into TidyMS's MSData object-oriented interface, which wraps pyopenms. Build extracted-ion chromatograms by accumulating consecutive MS scans over the acquired m/z range (e.g., m/z 50–1200 at ~0.5 s scan time), then apply the continuous wavelet transform (CWT)-based peak-picking algorithm of Du et al. (Bioinformatics 2006) with the parameter preset optimized for chromatographic data. Next, assemble features as (Rt, m/z) pairs per sample using the centWave-based feature detection routine (Tautenhahn et al., BMC Bioinformatics 2008), and collate per-sample detections into a single tabular feature matrix with Pandas. The full procedure sits upstream of retention time alignment, deisotoping, integration, and adduct grouping, and is designed to run at roughly 2 s per sample.

## Related tools

- **TidyMS** (Python package whose MSData interface performs CWT-based peak picking and centWave-based feature detection on raw LC–MS data) — https://github.com/griquelme/tidyms
- **pyopenms** (Underlying reader used through TidyMS's MSData object-oriented interface to read raw MS data in Python)
- **Python** (Language in which the feature detection pipeline is executed (with Pandas for collating the feature matrix))

## Examples

```
from tidyms import MSData; d = MSData('sample.mzML', mode='centroid', instrument='qtof'); d.detect_peaks('centroid'); d.detect_features('asari')
```

## Evaluation signals

- Feature detection runs at the reported speed of about 2 s per sample (66 s total for the 31 raw files of Application 1)
- The output feature table contains retention time and m/z descriptors for detected features across samples
- Per-sample detections are successfully collated into a single tabular feature matrix (e.g., feature_table.csv)
- Input files are confirmed centroid-mode mzML (converted from Waters RAW via ProteoWizard msconvert with min distance = 0.01 and snr = 1)
- Features are represented as (Rt, m/z) pairs consistent with the article's feature definition

## Limitations

- TidyMS lacks feature annotation capability, so detected features are unannotated (Rt, m/z) pairs
- No software version numbers are stated for TidyMS or Python in the article, which may affect reproducibility
- The CWT/centWave parameter preset is optimized for chromatographic data; other data types may need different settings
- No accession IDs are provided for the NIST candidate RM 8231 or SRM 1950 materials used in the exemplar applications

## Evidence

- [results] TidyMS performs raw-data peak analysis through the MSData object: [evidence span withheld — anchor status: not_found]
- [methods] workflow includes peak picking: "The procedure included retention time alignment, peak picking, deisotoping, integration, and grouping together adducts derived from the same compound."
- [results] runtime benchmark: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [methods] data provenance: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
- [intro] output of feature detection: "These software packages perform feature detection and correspondence, and provide an extracted data matrix as output for subsequent analysis."
