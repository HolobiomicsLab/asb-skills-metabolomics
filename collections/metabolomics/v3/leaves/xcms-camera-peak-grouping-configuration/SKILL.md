---
name: xcms-camera-peak-grouping-configuration
description: Use when raw LC-MS mass-spectrometry files (e.g. the MTBLS146 maternal
  plasma subset) must be converted into an aligned feature matrix of detected spectral
  features across biological, blank, and QC samples.
when_to_use_negative:
- Input is already a processed feature table or a pre-built DSO — the XCMS feature
  detection/alignment stage is redundant
- Data are direct infusion mass spectrometry (DIMS) transients — use the SimStitch-based
  DIMS pipeline (Sum Transients, Process Transients, Mass Calibration and SIM-stitching)
  instead
- Only univariate statistics are required — Galaxy-M explicitly includes no univariate
  statistics tools
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: XCMS
  role: R package performing feature detection, retention time correction and grouping/alignment
    of spectral features across samples in the XCMS-Basic-Birmingham-Pipe workflow
  repo: https://metlin.scripps.edu/xcms/
- name: Camera
  role: Installed alongside XCMS (via biocLite) in the R environment used by the LC-MS
    workflow for metabolite feature handling
- name: Galaxy-M
  role: Galaxy platform and tool wrappers that host and store/publish the LC-MS workflow
    history containing the XCMS-Basic-Birmingham-Pipe step
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:2f0ac500ef22470129a9e08e0460e2fdb9f9ebec2153fb381e0472ac89cbb53c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Camera
  - Galaxy-M
  - PLS-Toolbox
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# xcms-camera-peak-grouping-configuration

## Summary

Configuration and execution of the XCMS-Basic-Birmingham-Pipe step of the Galaxy-M LC-MS metabolomics workflow, in which the R package XCMS (with CAMERA) performs feature detection, retention time correction, and grouping/alignment of spectral features across samples.

## When to use

Use when raw LC-MS mass-spectrometry files (e.g. the MTBLS146 maternal plasma subset) must be converted into an aligned feature matrix of detected spectral features across biological, blank, and QC samples.

## When NOT to use

- Input is already a processed feature table or a pre-built DSO — the XCMS feature detection/alignment stage is redundant
- Data are direct infusion mass spectrometry (DIMS) transients — use the SimStitch-based DIMS pipeline (Sum Transients, Process Transients, Mass Calibration and SIM-stitching) instead
- Only univariate statistics are required — Galaxy-M explicitly includes no univariate statistics tools

## Inputs

- Raw LC-MS mass spectrometry files (MTBLS146 LC-MS raw files from ~/GalaxyM-TestData/LCMS_DATA)
- CSV instruction files used by the pipeline, from the GalaxyM-TestData folder
- Sample class labels distinguishing biological, blank, and QC samples

## Outputs

- Aligned/detected spectral feature list (feature matrix across samples)
- Filtered and cleaned feature matrix ready for missing value imputation, normalization, and Create DSO

## How to apply

Run the XCMS-Basic-Birmingham-Pipe workflow inside Galaxy-M (launched from the pre-installed VM, logged in as the galaxym user at 127.0.0.1:8080) on the LC-MS raw files from ~/GalaxyM-TestData/LCMS_DATA. The tool uses the R package XCMS (installed with CAMERA via biocLite; R 3.0.1, 64-bit) to perform feature detection, retention time correction, and grouping/alignment so that the same spectral feature observed across samples is represented consistently in the matrix. The output feature list must then be passed downstream to Replicate Filtering (peaks present in at least x-out-of-n technical replicates), Blank Filtering (peaks as strong in blanks as in biological samples are removed), and Sample Filtering (peaks must appear in x-out-of-n biological samples) before missing value imputation and normalization. Correct application means the aligned feature matrix feeds a Create DSO step that combines the X data matrix with row/column labels and class labels distinguishing biological from blank/QC samples, ultimately enabling a PLS-Toolbox PCA whose Scores Test matches the published Additional file 1: Table S4 for the MTBLS146 case study.

## Related tools

- **XCMS** (R package performing feature detection, retention time correction and grouping/alignment of spectral features across samples in the XCMS-Basic-Birmingham-Pipe workflow) — https://metlin.scripps.edu/xcms/
- **Camera** (Installed alongside XCMS (via biocLite) in the R environment used by the LC-MS workflow for metabolite feature handling)
- **Galaxy-M** (Galaxy platform and tool wrappers that host and store/publish the LC-MS workflow history containing the XCMS-Basic-Birmingham-Pipe step) — https://github.com/Viant-Metabolomics/Galaxy-M

## Examples

```
biocLite("XCMS"); biocLite("CAMERA")
```

## Evaluation signals

- The aligned feature list successfully passes Replicate Filtering, Blank Filtering, and Sample Filtering without breaking downstream steps
- Create DSO produces a DSO combining the X data matrix with row/column labels and class labels (biological vs blank/QC)
- Downstream PLS-Toolbox PCA retains seven principal components describing at least 70% of the variance in the MTBLS146 LC-MS subset
- PCA Scores Test table matches the published Additional file 1: Table S4
- The stored/published Galaxy-M LC-MS workflow history is reproduced end-to-end without missing steps

## Limitations

- The specific LC-MS workflow implemented (XCMS-Basic-Birmingham-Pipe) had not been used previously in published work; it was included as a representative series of processing steps
- The PCA/PLS-Toolbox tools run as compiled executables via MATLAB Compiler Runtime; the PCA tool has no graphical output, as statistical tests of separation are considered more reliable than visual interpretation
- Installation of the WINE/MSFileReader environment is complex and error-prone (e.g. WINE must be forced to 32-bit mode to avoid 'Invalid file identifier' errors)
- The DIMS transient files were not included in the MetaboLights accession and had to be obtained from the original authors

## Evidence

- [abstract] XCMS feature detection/alignment in the LC-MS pipeline: "The LC-MS data is processed using XCMS [23], including feature detection, retention time correction and alignment"
- [abstract] XCMS grouping/alignment across samples: "groups/aligns spectral features across the samples using the R-package XCMS [23]"
- [abstract] CAMERA among the metabolite identification tools: "and the metabolites identified (e.g. Camera, PUTMEDID and MI-Pack software)"
- [readme] Tested Galaxy commit for the tool wrappers: "The Galaxy version that these files have been tested on is from the Master branch on Github: commit c429777c93680dcee449fe410f5360afbe673758."
- [abstract] Create DSO combines feature matrix with labels and class labels: "Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank"
