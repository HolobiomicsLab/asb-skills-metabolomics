---
name: lc-ms-feature-detection-and-alignment
description: Use when raw LC-MS metabolomics data (e.g. MTBLS146 maternal plasma files
  in ~/GalaxyM-TestData/LCMS_DATA) must be converted from vendor spectra into a peak/feature
  table aligned across biological, blank and QC samples before filtering or PCA.
when_to_use_negative:
- Input is already a processed feature table or DSO — filtering, imputation and Create
  DSO should be applied instead of re-running feature detection.
- Data are direct infusion mass spectrometry (DIMS) transients — the Sum Transients
  / Process Transients / Mass Calibration and SIM-stitching / Align Samples DIMS pipeline
  applies, not XCMS-Basic-Birmingham-Pipe.
- The goal is metabolite identification rather than feature detection — use MI-Pack
  empirical formula search on the processed peaks instead.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: XCMS
  role: R package performing feature detection, retention time correction and grouping/alignment
    of spectral features across samples within the XCMS-Basic-Birmingham-Pipe Galaxy-M
    tool
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Camera
  role: Installed alongside XCMS (via biocLite) in the R environment used by the LC-MS
    feature detection workflow
- name: Galaxy-M
  role: Wraps the XCMS pipeline as Galaxy tool files (.xml wrappers and original code)
    executed in the pre-installed VM from GigaDB
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Platform hosting the workflow/history; tested at Master branch commit c429777c93680dcee449fe410f5360afbe673758
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:eac688f27236dbd092f1a4ab7b55ed0fa8729e4eedea453b2b346f5bf8aa36e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Camera
  - Galaxy
  - Galaxy-M
  - PLS-Toolbox
  - XCMS
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# lc-ms-feature-detection-and-alignment

## Summary

Detection, retention-time correction and cross-sample alignment of LC-MS spectral features using the XCMS-Basic-Birmingham-Pipe workflow (R XCMS with CAMERA) as implemented as a Galaxy-M tool, producing the feature table that feeds all downstream filtering, statistics and annotation steps.

## When to use

Use when raw LC-MS metabolomics data (e.g. MTBLS146 maternal plasma files in ~/GalaxyM-TestData/LCMS_DATA) must be converted from vendor spectra into a peak/feature table aligned across biological, blank and QC samples before filtering or PCA.

## When NOT to use

- Input is already a processed feature table or DSO — filtering, imputation and Create DSO should be applied instead of re-running feature detection.
- Data are direct infusion mass spectrometry (DIMS) transients — the Sum Transients / Process Transients / Mass Calibration and SIM-stitching / Align Samples DIMS pipeline applies, not XCMS-Basic-Birmingham-Pipe.
- The goal is metabolite identification rather than feature detection — use MI-Pack empirical formula search on the processed peaks instead.

## Inputs

- LC-MS raw mass spectrometry files (e.g. MTBLS146 maternal plasma subset from ~/GalaxyM-TestData/LCMS_DATA)
- CSV instruction files from GalaxyM-TestData used by downstream workflow steps
- Stored/published Galaxy-M LCMS workflow/history definition

## Outputs

- Aligned spectral feature table (peaks detected, RT-corrected and grouped across samples)
- CAMERA-annotated feature list ready for Replicate/Blank/Sample Filtering and missing value imputation

## How to apply

Run the XCMS-Basic-Birmingham-Pipe workflow from the stored/published Galaxy-M history (Galaxy at 127.0.0.1:8080), uploading the MTBLS146 LC-MS raw files. The pipeline performs feature detection, retention time correction, and grouping/alignment of spectral features across samples using the R-package XCMS, with CAMERA installed via biocLite for adduct/isotope annotation of the detected features. It relies on Python 2.7, R 3.0.1, the Thermo Scientific MSFileReader package and WineHQ (version 1.7) in the reference VM, and the tested Galaxy commit is c429777c93680dcee449fe410f5360afbe673758. The output feature list is then subjected to Replicate Filtering, Blank Filtering and Sample Filtering, followed by missing value imputation and normalization, before Create DSO assembles the X matrix with row/column and class labels. This specific LC-MS workflow had not been used previously in published work; it was included as a representative series of processing steps, so treat parameters as exemplar rather than optimal.

## Related tools

- **XCMS** (R package performing feature detection, retention time correction and grouping/alignment of spectral features across samples within the XCMS-Basic-Birmingham-Pipe Galaxy-M tool) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Camera** (Installed alongside XCMS (via biocLite) in the R environment used by the LC-MS feature detection workflow)
- **Galaxy-M** (Wraps the XCMS pipeline as Galaxy tool files (.xml wrappers and original code) executed in the pre-installed VM from GigaDB) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Platform hosting the workflow/history; tested at Master branch commit c429777c93680dcee449fe410f5360afbe673758) — https://github.com/galaxyproject/galaxy

## Evaluation signals

- Feature detection, RT correction and alignment complete without error for all LC-MS raw files in LCMS_DATA within the Galaxy history.
- Downstream Replicate Filtering, Blank Filtering and Sample Filtering accept the aligned feature list and proceed to missing value imputation and normalization.
- Create DSO successfully combines the X data matrix with row/column labels and biological vs blank/QC class labels.
- End-to-end validation: the subsequent PCA Scores Test table matches the published Additional file 1: Table S4 for the MTBLS146 early vs late pregnancy comparison.
- Reproducibility check: the same Galaxy commit (c429777c93680dcee449fe410f5360afbe673758) and MI-Pack commit (06ce0ace643ee1cf1d27550769a5272b7ea50825) as the published history are used.

## Limitations

- This specific LC-MS workflow had not been used previously in published work; it was chosen as a representative series of processing steps, not a validated optimum.
- The published MTBLS146 subset is only a portion of the full MetaboLights MTBLS146 accession, so feature counts will not match the original study.
- Recreating the environment is error-prone: WINE must be forced to 32-bit (WINEARCH=win32) or tools fail with 'Invalid file identifier' when MSFileReader cannot open the data; installation instructions are described as complex and involving a lot of troubleshooting.
- No univariate statistics tools are included in the current Galaxy-M implementation, and the PCA tool produces no graphical output.

## Evidence

- [abstract] XCMS-Basic-Birmingham-Pipe performs feature detection and alignment: "The LC-MS data is processed using XCMS [23], including feature detection, retention time correction and alignment"
- [abstract] Workflow groups/aligns features across samples: "groups/aligns spectral features across the samples using the R-package XCMS [23]"
- [abstract] Workflow is a representative example, not previously published: "The specific LC-MS workflow implemented here has not been used previously in published work; it has been included as a representative series of processing steps"
- [readme] Tool files and tested Galaxy version in README: "Included are the tool files (both original code and .xml wrappers) for Metabolomics analysis and the Galaxy config files from a working installation of Galaxy."
- [readme] WINE 32-bit requirement for MSFileReader: "if you do not do this, you will likely get errors from the SimStitch tools along the lines of 'Invalid file identifier' - this is because they can't use the MSFileReader tool to open the DIMS data."
- [readme] Location of LC-MS test data in VM: "the Mass Spectra can be found in :

* `~/GalaxyM-TestData/LCMS_DATA`"
