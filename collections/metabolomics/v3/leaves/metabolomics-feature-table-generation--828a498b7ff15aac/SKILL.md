---
name: metabolomics-feature-table-generation
description: Use when raw centroid-mode LC–MS mzML files must be converted into a
  per-sample feature matrix with (Rt, m/z) descriptors for downstream QC filters or
  statistical analysis in untargeted metabolomics.
when_to_use_negative:
- The data are already in the form of a processed feature table from tools such as
  MZmine2, XCMS, or MSDIAL — this skill operates on centroid-mode mzML raw files.
- Only centroided profile data that have not been converted from vendor RAW format
  (e.g. Waters RAW) are available; conversion via ProteoWizard msconvert is a prerequisite
  step, not part of this skill.
- The goal is feature annotation rather than feature detection — the article explicitly
  notes TidyMS lacks feature annotation capability.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TidyMS
  role: provides the MSData object-oriented interface and the CWT-based peak-picking
    plus centWave feature detection routines used to build the feature table
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Python package used through TidyMS's MSData interface to read the raw mzML
    MS data
- name: Python
  role: programming language in which the pipeline is implemented (Python 3.6)
- name: Pandas
  role: used to collate per-sample feature detections into the single tabular feature
    matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:3f33a4cd1a631599a6e21de52cd13dd84bff9140f917fa32b71489d9e0993a84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - AutoTuner
  - IPO
  - OpenMS
  - Pandas
  - Python
  - R
  - TidyMS
  - UmetaFlow
  - pyOpenMS
  - pyopenms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- untargeted-metabolomics-feature-table-construction
- untargeted-metabolomics-feature-table-generation
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolomics-feature-table-generation

## Summary

Generates a tabular feature matrix of (retention time, m/z) pairs from centroid-mode LC–MS mzML raw files using TidyMS's continuous wavelet transform (CWT) peak picking and centWave-based feature correspondence, the feature detection stage of TidyMS's untargeted metabolomics preprocessing pipeline.

## When to use

Use when raw centroid-mode LC–MS mzML files must be converted into a per-sample feature matrix with (Rt, m/z) descriptors for downstream QC filters or statistical analysis in untargeted metabolomics.

## When NOT to use

- The data are already in the form of a processed feature table from tools such as MZmine2, XCMS, or MSDIAL — this skill operates on centroid-mode mzML raw files.
- Only centroided profile data that have not been converted from vendor RAW format (e.g. Waters RAW) are available; conversion via ProteoWizard msconvert is a prerequisite step, not part of this skill.
- The goal is feature annotation rather than feature detection — the article explicitly notes TidyMS lacks feature annotation capability.

## Inputs

- centroid-mode LC–MS mzML raw data files (e.g. Waters RAW previously converted with ProteoWizard msconvert using min distance = 0.01 and snr = 1)

## Outputs

- single tabular feature matrix (feature_table.csv) with (Rt, m/z) descriptors per detected feature across samples

## How to apply

Read the centroid-mode mzML raw files into TidyMS's MSData object via the pyopenms package in Python 3.6. Build extracted-ion chromatograms by accumulating consecutive MS scans over the acquired m/z 50–1200 range (0.5 s scan time), then apply the CWT-based peak-picking algorithm (Du et al., Bioinformatics 2006) with the parameter preset optimized for chromatographic data. Locate peaks in each chromatogram and assemble features as (Rt, m/z) pairs per sample using the centWave-based feature detection routine (Tautenhahn et al., BMC Bioinformatics 2008). Collate the per-sample detections into a single tabular feature matrix with Pandas and save it as feature_table.csv; expect roughly 2 s per sample for feature detection (66 s for 31 raw files in the article's Application 1).

## Related tools

- **TidyMS** (provides the MSData object-oriented interface and the CWT-based peak-picking plus centWave feature detection routines used to build the feature table) — https://github.com/griquelme/tidyms
- **pyopenms** (Python package used through TidyMS's MSData interface to read the raw mzML MS data)
- **Python** (programming language in which the pipeline is implemented (Python 3.6))
- **Pandas** (used to collate per-sample feature detections into the single tabular feature matrix)

## Examples

```
import tidyms as ms; msdata = ms.MSData.from_mzml('MTBLS1919_file.mzML'); msdata.detect_features()
```

## Evaluation signals

- The output feature_table.csv contains retention time and m/z descriptors for detected features across samples.
- Feature detection runs at the reported speed of about 2 s per sample (66 s for the 31 raw files of Application 1).
- Every per-sample (Rt, m/z) detection has been collated into a single table before downstream filtering steps such as the Rt filter at 90 s or the RSD < 20% filter in pooled QC samples.
- The raw mzML files trace back to a documented conversion step (ProteoWizard msconvert, min distance = 0.01, snr = 1) so provenance is preserved.

## Limitations

- TidyMS lacks feature annotation capability, so the feature table produced by this skill must be annotated with other tools.
- The CWT/centWave feature detection assumes centroid-mode data acquired over the m/z 50–1200 range with 0.5 s scan time; parameters were preset for chromatographic data of this kind.
- The feature table alone is not analytically robust — downstream removal of unwanted features via QC filters (Rt filter, blank filter, RSD filter, D-ratio filter) is required before statistical analysis.

## Evidence

- [results] CWT peak picking on MSData: [evidence span withheld — anchor status: not_found]
- [methods] raw mzML input and conversion: [evidence span withheld — anchor status: not_found]
- [results] feature detection speed: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [intro] tools used for comparison / context: [evidence span withheld — anchor status: not_found]
- [methods] TidyMS repository: "Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms"
- [results] no annotation capability: "Feature annotation ❌ ✅ ❌ ✅ ❌ ❌ ❌"
