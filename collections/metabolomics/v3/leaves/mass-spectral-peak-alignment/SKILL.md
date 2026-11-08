---
name: mass-spectral-peak-alignment
description: Use when you have per-sample DIMS peak lists (e.g. SIM-stitched FT-ICR
  m/z peaks from MTBLS79 cow/sheep heart samples plus QC and blank) and need a cross-sample
  aligned matrix before replicate/blank/sample filtering and PCA.
when_to_use_negative:
- Input is already an aligned feature table or data matrix (e.g. post-filtering DSO)
  — alignment would be redundant
- Processing LC-MS data, where Galaxy-M instead uses the XCMS-Basic-Birmingham-Pipe
  (feature detection, retention time correction and alignment via XCMS) rather than
  the DIMS Align Samples tool
- Raw unprocessed .dat transients — SIM-stitch peak picking must precede alignment
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Galaxy-M
  role: Hosts the 'Align Samples' tool that aligns peaks across all samples in the
    DIMS workflow
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: SimStitch
  role: Upstream tool whose Mass Calibration and SIM-stitching step produces the per-sample
    peak lists (SNR threshold e.g. 3.5:1) that Align Samples consumes
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Platform in which the alignment and subsequent filtering/statistics steps
    are chained as a workflow
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:118679c2574b7931d32d4e3921f27141ba5b3de9913641d6dfd3442bbf4ff1c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - PLS-Toolbox
  - SimStitch
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mass-spectral-peak-alignment

## Summary

Aligning peak lists across samples in the Galaxy-M DIMS workflow so that m/z features measured in multiple samples are matched into a single data matrix ready for filtering, imputation, and PCA. This step converts per-sample SIM-stitched peak lists into the consistent X matrix the Galaxy-M statistics tools require.

## When to use

Use when you have per-sample DIMS peak lists (e.g. SIM-stitched FT-ICR m/z peaks from MTBLS79 cow/sheep heart samples plus QC and blank) and need a cross-sample aligned matrix before replicate/blank/sample filtering and PCA.

## When NOT to use

- Input is already an aligned feature table or data matrix (e.g. post-filtering DSO) — alignment would be redundant
- Processing LC-MS data, where Galaxy-M instead uses the XCMS-Basic-Birmingham-Pipe (feature detection, retention time correction and alignment via XCMS) rather than the DIMS Align Samples tool
- Raw unprocessed .dat transients — SIM-stitch peak picking must precede alignment

## Inputs

- Per-sample SIM-stitched DIMS peak lists (m/z and intensity) derived from .dat FT-ICR transient files in ~/GalaxyM-TestData/DIMS_DATA
- Sample set including biological samples (MTBLS79 cow heart vs sheep heart), QC samples, and blank samples

## Outputs

- A single aligned peak table / X data matrix with consistent m/z feature columns across all samples
- Downstream (after filtering, imputation, normalization, scaling): DSO input for PCA and the Scores Test t-test p-value text file

## How to apply

In the Galaxy-M VM, run the 'Align Samples' tool after the SIM-stitch tools (Sum Transients, Process Transients with Hanning apodization/zero-filling/Fourier transformation/baseline correction, and Mass Calibration and SIM-stitching with an SNR threshold such as 3.5:1) on the DIMS .dat transient files in ~/GalaxyM-TestData/DIMS_DATA. Align Samples matches detected peaks by m/z across all samples (biological samples, QC, and blanks) so the same feature occupies one column. The aligned peak table then feeds Replicate Filtering, Blank Filtering (removing peaks as strong in blanks as in biological samples), and Sample Filtering (peaks appearing in fewer than x-of-n samples), followed by missing value imputation, normalization and scaling before Create DSO and PCA. Correct alignment is judged by the workflow completing and producing a coherent X matrix whose columns are consistent peaks across all samples, ultimately yielding PCA scores and a Scores Test t-test p-value matching the published DIMS history.

## Related tools

- **Galaxy-M** (Hosts the 'Align Samples' tool that aligns peaks across all samples in the DIMS workflow) — https://github.com/Viant-Metabolomics/Galaxy-M
- **SimStitch** (Upstream tool whose Mass Calibration and SIM-stitching step produces the per-sample peak lists (SNR threshold e.g. 3.5:1) that Align Samples consumes) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Platform in which the alignment and subsequent filtering/statistics steps are chained as a workflow) — https://github.com/galaxyproject/galaxy

## Evaluation signals

- The aligned output is a matrix in which the same m/z feature appears in one column across all biological, QC, and blank samples
- The workflow completes in Galaxy and downstream Replicate Filtering, Blank Filtering, and Sample Filtering run without errors on the aligned table
- After Create DSO, PCA via PLS-Toolbox separates cow vs sheep heart biological classes from blanks, and the Scores Test t-test p-value text file is produced
- MI-Pack annotation of the aligned/filtered peaks produces a tab-delimited metabolite annotation file matching the published 'DIMS Test Data processing' history outputs (Additional file 1: Table S2)

## Limitations

- The DIMS .dat transient files are not included in the MetaboLights MTBLS79 accession; they must be obtained from GigaDB or the original authors, otherwise alignment cannot be reached
- WINE must be forced to 32-bit mode (WINEARCH=win32) or the SimStitch tools fail with 'Invalid file identifier' errors and never produce peaks to align
- No univariate statistics tools are included in the current Galaxy-M implementation; alignment output only supports the PCA/Scores Test path

## Evidence

- [abstract] Align Samples: aligns peaks across all samples: "Align Samples: aligns peaks across all samples"
- [abstract] SIM-stitch peak picking with SNR threshold: "picks peaks with a certain signal-to-noise-ratio threshold (e.g. 3.5:1), calibrates each SIM window to convert the frequency domain to m/z"
- [readme] Data location on the VM: "the Mass Spectra can be found in :

* `~/GalaxyM-TestData/LCMS_DATA`
* `~/GalaxyM-TestData/DIMS_DATA`"
- [readme] WINE 32-bit requirement for SimStitch tools: "if you do not do this, you will likely get errors from the SimStitch tools along the lines of 'Invalid file identifier'"
