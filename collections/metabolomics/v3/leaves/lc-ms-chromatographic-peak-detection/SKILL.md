---
name: lc-ms-chromatographic-peak-detection
description: Use when raw centroid-mode LC–MS mzML files need chromatographic peak
  picking and feature detection prior to feature table generation in untargeted metabolomics.
when_to_use_negative:
- Input is already a preprocessed feature table or data matrix — peak picking has
  already occurred downstream of this skill
- The raw files are in profile mode or in a vendor format (e.g. Waters RAW) that has
  not been converted to centroid-mode mzML via msconvert
- The goal is post-detection data curation (Rt filter, blank filter, RSD filter, D-ratio
  filter) rather than locating peaks in raw chromatograms
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: TidyMS
  role: Primary library providing the MSData interface, CWT-based peak picking, and
    centWave-based feature detection
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Underlying reader used by TidyMS's MSData object-oriented interface to load
    raw MS data in Python
- name: Python
  role: Runtime language for the pipeline; TidyMS is a package for the Python programming
    language
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:7abb9fd6c51c8f9943e35c06bec99fd6835136414d5165e832428435648ff472
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Python
  - TidyMS
  - pyopenms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lc-ms-chromatographic-peak-detection

## Summary

Applies TidyMS's continuous wavelet transform (CWT)-based peak-picking algorithm to raw LC–MS chromatograms and spectrum peaks via the MSData interface, enabling feature detection as (Rt, m/z) pairs in centroid-mode mzML files.

## When to use

Use when raw centroid-mode LC–MS mzML files need chromatographic peak picking and feature detection prior to feature table generation in untargeted metabolomics.

## When NOT to use

- Input is already a preprocessed feature table or data matrix — peak picking has already occurred downstream of this skill
- The raw files are in profile mode or in a vendor format (e.g. Waters RAW) that has not been converted to centroid-mode mzML via msconvert
- The goal is post-detection data curation (Rt filter, blank filter, RSD filter, D-ratio filter) rather than locating peaks in raw chromatograms

## Inputs

- Centroid-mode mzML raw LC–MS data files (e.g. from MetaboLights MTBLS1919, converted from Waters RAW with ProteoWizard msconvert)
- Instrument metadata: scan time (e.g. 0.5 s), acquired m/z range (e.g. 50–1200)
- Python environment with TidyMS and pyopenms installed

## Outputs

- Detected chromatographic and spectrum peaks per sample (CWT peak picking output)
- Feature set as (Rt, m/z) pairs per sample from centWave-based feature detection
- Collated tabular feature matrix (e.g. feature_table.csv) with retention time and m/z descriptors for detected features across samples

## How to apply

Read the raw centroid-mode mzML files with TidyMS's MSData object-oriented interface, which relies on pyopenms. Build extracted-ion chromatograms by accumulating consecutive MS scans over the m/z 50–1200 range (0.5 s scan time), then apply the CWT-based peak-picking algorithm (Du et al., Bioinformatics 2006) with the parameter preset optimized for chromatographic data to locate peaks in each chromatogram. Assemble detected features as (Rt, m/z) pairs per sample using the centWave-based feature detection routine (Tautenhahn et al., BMC Bioinformatics 2008), and collate the per-sample detections into a single tabular feature matrix (e.g. saved with Pandas as feature_table.csv). TidyMS executes feature detection at roughly 2 s per sample on the demonstrated raw data, so runtime can serve as a sanity check.

## Related tools

- **TidyMS** (Primary library providing the MSData interface, CWT-based peak picking, and centWave-based feature detection) — https://github.com/griquelme/tidyms
- **pyopenms** (Underlying reader used by TidyMS's MSData object-oriented interface to load raw MS data in Python)
- **Python** (Runtime language for the pipeline; TidyMS is a package for the Python programming language)

## Examples

```
from tidyms.data import MSData; data = MSData('sample.mzML'); data.find_peaks(); features = data.extract_features()
```

## Evaluation signals

- Output feature table contains both retention time and m/z descriptors for detected features across samples
- Feature detection runtime is on the order of ~2 s per sample (66 s total for the 31 raw files in Application 1)
- Per-sample peak detections are non-empty and correspond to features within the acquired m/z range (50–1200)
- Feature matrix collates detections from all samples into a single table loadable with Pandas

## Limitations

- Requires centroid-mode data — raw vendor files must first be converted (e.g. Waters RAW → mzML with ProteoWizard msconvert using min distance = 0.01 and snr = 1)
- Feature annotation is not supported by TidyMS; a separate tool is needed after peak detection
- Peak picking was demonstrated on chromatographic data with a specific parameter preset; other data types may need different settings

## Evidence

- [methods] TidyMS performs raw-data peak analysis through the MSData object, detecting chromatographic and spectrum peaks with a continuous wavelet transform (CWT)-based peak picking algorithm: [evidence span withheld — anchor status: not_found]
- [methods] Build extracted-ion chromatograms by accumulating consecutive MS scans over the acquired m/z 50-1200 range (0.5 s scan time): [evidence span withheld — anchor status: not_found]
- [results] Feature detection runtime for 31 raw data files: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [methods] Raw data deposition as mzML in MetaboLights: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
- [abstract] TidyMS is a Python package for LC–MS preprocessing in untargeted metabolomics: "The present work introduces TidyMS, a package for the Python programming language for preprocessing LC–MS data for QC procedures in untargeted metabolomics workflows."
