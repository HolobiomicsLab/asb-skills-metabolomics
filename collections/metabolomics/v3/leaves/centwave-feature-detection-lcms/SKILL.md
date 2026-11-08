---
name: centwave-feature-detection-lcms
description: Use when starting from raw LC–MS files (e.g., mzML) from untargeted metabolomics
  acquisitions and you need feature detection — peak picking, deisotoping, adduct
  grouping — to build the extracted data matrix that subsequent curation filters will
  operate on.
when_to_use_negative:
- Input is already an extracted feature table or data matrix (e.g., exported from
  MZmine2, XCMS, or MSDIAL) rather than raw mzML — TidyMS curation pipelines then
  operate on the DataContainer instead.
- 'Feature annotation is the goal: the paper shows TidyMS lacks feature annotation
  capability, so it is the wrong tool for assigning compound identities to detected
  features.'
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: TidyMS
  role: 'Primary framework: provides MSData, the CWT-based peak-picking algorithm,
    chromatogram building, centWave feature detection, and the DataContainer/Pipeline
    architecture.'
  repo: https://github.com/griquelme/tidyms
- name: Python
  role: Implementation language; TidyMS is a Python package for preprocessing LC–MS
    data.
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Library used to read the raw mzML input files into TidyMS MSData objects.
- name: Pandas
  role: Stores the extracted data matrix and associated feature and sample descriptors
    inside the DataContainer.
- name: Numpy
  role: Numerical backing for the data matrix and spectral computations.
- name: Scipy
  role: Numerical backing for the CWT-based peak-picking implementation.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:192b61f483bfe983538f1d5e7d651064e30a3a99dabe1344dd0f9844a463f1e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Numpy
  - Pandas
  - Python
  - Scipy
  - TidyMS
  - pyopenms
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# centwave-feature-detection-lcms

## Summary

Extract chromatographic spectral features from raw LC–MS data using TidyMS's CWT-based peak picking, chromatogram building, and centWave-style feature-detection routines, turning mzML files into a feature table (data matrix plus feature/sample descriptors).

## When to use

Use when starting from raw LC–MS files (e.g., mzML) from untargeted metabolomics acquisitions and you need feature detection — peak picking, deisotoping, adduct grouping — to build the extracted data matrix that subsequent curation filters will operate on.

## When NOT to use

- Input is already an extracted feature table or data matrix (e.g., exported from MZmine2, XCMS, or MSDIAL) rather than raw mzML — TidyMS curation pipelines then operate on the DataContainer instead.
- Feature annotation is the goal: the paper shows TidyMS lacks feature annotation capability, so it is the wrong tool for assigning compound identities to detected features.

## Inputs

- Raw LC–MS data files in mzML format (e.g., from NIST candidate RM 8231 plasma and NIST SRM 1950 injections)
- Sample mapping / class annotations associating each injection to sample types (study samples, pooled/intrastudy QC, blanks)

## Outputs

- TidyMS DataContainer holding the extracted data matrix (samples × features) as a Pandas DataFrame
- Per-feature descriptors (retention time, m/z) and per-sample descriptors as Pandas DataFrames
- Serialized processed DataContainer file for downstream curation

## How to apply

Read the raw LC–MS data from the input mzML file into TidyMS MSData objects via pyopenms. Extract spectral features using the CWT-based peak-picking algorithm with parameters optimized for chromatographic data, plus chromatogram building and centWave feature-detection routines. The upstream procedure includes retention time alignment, peak picking, deisotoping, integration, and grouping together adducts derived from the same compound. Wrap the resulting data matrix and its associated feature and sample descriptors (Pandas DataFrames) in a TidyMS DataContainer, populating sample-mapping so later Processor filters know sample classes. Serialize the DataContainer or hand it to a Pipeline of Processors; on the paper's 31 raw files the full analysis ran in 66 s, with feature detection about 2 s per sample.

## Related tools

- **TidyMS** (Primary framework: provides MSData, the CWT-based peak-picking algorithm, chromatogram building, centWave feature detection, and the DataContainer/Pipeline architecture.) — https://github.com/griquelme/tidyms
- **Python** (Implementation language; TidyMS is a Python package for preprocessing LC–MS data.) — https://github.com/griquelme/tidyms
- **pyopenms** (Library used to read the raw mzML input files into TidyMS MSData objects.)
- **Pandas** (Stores the extracted data matrix and associated feature and sample descriptors inside the DataContainer.)
- **Numpy** (Numerical backing for the data matrix and spectral computations.)
- **Scipy** (Numerical backing for the CWT-based peak-picking implementation.)

## Examples

```
from tidyms import MSData; ms = MSData.from_pyopenms('sample.mzML'); features = ms.extract_features()
```

## Evaluation signals

- Feature detection completes on each mzML sample in roughly 2 s (66 s total for the 31 raw data files in the paper's Application 1).
- Output DataContainer contains a samples × features data matrix plus per-feature (m/z, retention time) and per-sample descriptors stored as Pandas DataFrames.
- sample-mapping attribute correctly associates each sample to its class so downstream filters (e.g., blank subtraction, QC RSD filter) behave as intended.
- Serialized DataContainer loads back as a valid TidyMS DataContainer and can be passed to a Pipeline of Processors.
- Detected features are plausibly m/z / retention-time pairs after deisotoping and adduct grouping — no unprocessed raw spectra remain in the output.

## Limitations

- Feature detection performance depends on parameters optimized for chromatographic LC–MS data; settings tuned elsewhere may not transfer.
- TidyMS lacks feature annotation capability, so detected features are unassigned m/z / RT pairs.
- MzML input is required via pyopenms; other vendor formats are not described in the article.
- Feature detection is only the first stage — the article frames it within a pipeline whose later steps (LOESS batch correction, D-ratio filter with 10% threshold, RSD 20% cutoff) operate on the extracted matrix.

## Evidence

- [results] CWT-based peak-picking and centWave detection: [evidence span withheld — anchor status: not_found]
- [methods] Upstream processing includes alignment, picking, deisotoping, adduct grouping: "The procedure included retention time alignment, peak picking, deisotoping, integration, and grouping together adducts derived from the same compound."
- [results] Runtime for feature detection: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [abstract] TidyMS purpose as a preprocessing package: "The present work introduces TidyMS, a package for the Python programming language for preprocessing LC–MS data for quality control (QC) procedures in untargeted metabolomics workflows."
- [results] Annotated feature table can be rebuilt: "Feature annotation ❌ ✅ ❌ ✅ ❌ ❌ ❌"
