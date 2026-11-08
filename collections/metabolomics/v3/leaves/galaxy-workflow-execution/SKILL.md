---
name: galaxy-workflow-execution
description: Use when you have direct infusion or LC-MS mass spectrometry metabolomics
  raw files (e.g. Thermo .RAW/.dat transients or vendor LC-MS files) and need to execute
  the complete published Galaxy-M pipeline in the Galaxy web UI (127.0.0.1:8080) to
  reproduce published results.
when_to_use_negative:
- Input is already a processed, aligned peak or feature table — jump directly to Create
  DSO/PCA rather than re-running raw-data processing.
- 'You need univariate statistics: the Galaxy-M implementation contains no univariate
  statistics tools.'
- You cannot run the pre-installed VM and only have a bare Galaxy instance — installing
  Wine 32-bit, Thermo MSFileReader, MATLAB MCR and PLS-Toolbox from the README is
  complex and error-prone.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: Galaxy-M
  role: Set of Metabolomics tools (tool files and .xml wrappers) installed in Galaxy
    to run the DIMS and LC-MS pipelines end to end
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Platform on which the workflows are defined, stored, published and executed
    from the web UI
  repo: https://github.com/galaxyproject/galaxy
- name: XCMS
  role: R package invoked by the LC-MS workflow for feature detection, retention time
    correction and grouping/alignment
- name: MI-Pack
  role: Metabolite annotation tool run as the final step against the peak list (uses
    Parallel Python optionally for e.g. Empirical Formula search)
  repo: https://github.com/Viant-Metabolomics/MI-Pack
- name: PLS-Toolbox
  role: PCA engine (compiled executables run via MATLAB MCR) used for the Scores Test
    within the workflow
  repo: http://www.eigenvector.com/software/pls_toolbox.htm
- name: SimStitch
  role: Suite (Sum Transients, Process Transients, Mass Calibration and SIM-stitching)
    processing the FT-ICR transients before alignment
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_007
  - task_001
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:55119f985f83860d09e1f174e216e3dae34ab112658887d13471d95f312b33c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - MI-Pack
  - PLS-Toolbox
  - SimStitch
  - XCMS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# galaxy-workflow-execution

## Summary

Re-run the Galaxy-M published workflow histories (DIMS SIM-stitch or LC-MS XCMS pipelines) inside the pre-installed Galaxy-M VM, from raw mass spectra upload through filtered peak tables and PCA Scores Test outputs, using stored workflows rather than ad hoc tool chaining.

## When to use

Use when you have direct infusion or LC-MS mass spectrometry metabolomics raw files (e.g. Thermo .RAW/.dat transients or vendor LC-MS files) and need to execute the complete published Galaxy-M pipeline in the Galaxy web UI (127.0.0.1:8080) to reproduce published results.

## When NOT to use

- Input is already a processed, aligned peak or feature table — jump directly to Create DSO/PCA rather than re-running raw-data processing.
- You need univariate statistics: the Galaxy-M implementation contains no univariate statistics tools.
- You cannot run the pre-installed VM and only have a bare Galaxy instance — installing Wine 32-bit, Thermo MSFileReader, MATLAB MCR and PLS-Toolbox from the README is complex and error-prone.

## Inputs

- DIMS FT-ICR transient files (.dat, e.g. MTBLS79 cow vs sheep heart subset from GigaDB) or Thermo .RAW files readable via MSFileReader
- LC-MS raw mass spectra files (MTBLS146 maternal plasma subset)
- CSV instruction files in ~/GalaxyM-TestData for DSO row/column labels and class labels
- Stored/published Galaxy-M workflow definitions ('DIMS Test Data processing', 'XCMS-Basic-Birmingham-Pipe')

## Outputs

- PCA Scores Test t-test p-value text file (DIMS history)
- MI-Pack metabolite annotation tab-delimited file
- Aligned, replicate/blank/sample-filtered peak table exported as CSV
- Processed XCMS feature table with CAMERA annotation (LC-MS history)

## How to apply

Launch the Galaxy-M virtual machine, log in as the galaxym user (email <maintainer e-mail, see the repository README>, password galaxym) and open the stored/published workflow or history — 'DIMS Test Data processing' for FT-ICR direct infusion data or 'XCMS-Basic-Birmingham-Pipe' for LC-MS feature detection, retention time correction and alignment. Point the workflow at the mass spectra in ~/GalaxyM-TestData/DIMS_DATA or ~/GalaxyM-TestData/LCMS_DATA and supply the CSV instruction files the workflows require from GalaxyM-TestData for row/column labels and class labels. The DIMS pipeline chains Sum Transients, Process Transients (Hanning apodization, zero-filling, Fourier transformation, baseline correction), Mass Calibration and SIM-stitching (SNR threshold e.g. 3.5:1), Align Samples, Replicate/Blank/Sample Filtering, imputation, normalization, Create DSO and PCA (PLS-Toolbox); the LC-MS pipeline runs XCMS with CAMERA, filtering, imputation (e.g. KNN under the 80% Sample Filtering threshold) and PCA. The rationale is that running the stored published history, not re-implementing steps individually, preserves tool order and parameterization so outputs match the publication's validation tables. For the DIMS transients, use the GigaDB .dat files since the MetaboLights MTBLS79 accession lacks them.

## Related tools

- **Galaxy-M** (Set of Metabolomics tools (tool files and .xml wrappers) installed in Galaxy to run the DIMS and LC-MS pipelines end to end) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Platform on which the workflows are defined, stored, published and executed from the web UI) — https://github.com/galaxyproject/galaxy
- **XCMS** (R package invoked by the LC-MS workflow for feature detection, retention time correction and grouping/alignment)
- **MI-Pack** (Metabolite annotation tool run as the final step against the peak list (uses Parallel Python optionally for e.g. Empirical Formula search)) — https://github.com/Viant-Metabolomics/MI-Pack
- **PLS-Toolbox** (PCA engine (compiled executables run via MATLAB MCR) used for the Scores Test within the workflow) — http://www.eigenvector.com/software/pls_toolbox.htm
- **SimStitch** (Suite (Sum Transients, Process Transients, Mass Calibration and SIM-stitching) processing the FT-ICR transients before alignment) — https://github.com/Viant-Metabolomics/Galaxy-M

## Examples

```
sh run.sh  # start Galaxy on the VM, then open http://127.0.0.1:8080, log in as <maintainer e-mail, see the repository README> and run the stored 'DIMS Test Data processing' workflow
```

## Evaluation signals

- DIMS history is successful when the workflow completes in Galaxy with both the PCA Scores Test t-test p-value text file and the MI-Pack annotation tab-delimited file produced.
- LC-MS history validates when the PCA Scores Test table matches the published Additional file 1: Table S4 and the MI-Pack annotation output is generated and consistent with the published history.
- Peaks entering and leaving each filter step satisfy peaks_out = peaks_in - peaks_removed for Replicate, Blank and Sample Filtering.
- The specific LC-MS workflow (XCMS-Basic-Birmingham-Pipe) matches the tool-chain order and parameters used in the stored published history rather than a re-derived chain.
- Wine runs in 32-bit mode (WINEARCH=win32) so SimStitch tools can open Thermo data via MSFileReader without 'Invalid file identifier' errors.

## Limitations

- GigaDB accession for the VM was 'to follow' at time of writing; the article and README describe the VM but do not provide a final accession in the text.
- SimStitch tools fail with 'Invalid file identifier' if Wine is not forced to 32-bit mode (WINEARCH=win32) before installing MSFileReader.
- Transient (.dat) files are not in the MetaboLights MTBLS79 accession and must be obtained from GigaDB.
- Installation to recreate the VM is complex and can involve a lot of troubleshooting; working from the pre-installed VM is advised.
- The Galaxy version tested is a specific master commit (c429777c93680dcee449fe410f5360afbe673758); other Galaxy versions may not be compatible with these tool wrappers.

## Evidence

- [readme] Launching the pre-installed VM and loading stored histories: "The GalaxyM installation has a user already registered and two workflows have been stored/published."
- [abstract] SIM-stitching tools and peak picking threshold: "picks peaks with a certain signal-to-noise-ratio threshold (e.g. 3.5:1), calibrates each SIM window to convert the frequency domain to m/z measurements and stitches all the SIM windows together"
- [results] DIMS validation outputs: [evidence span withheld — anchor status: not_found]
- [abstract] LC-MS feature detection, RT correction and alignment: "groups/aligns spectral features across the samples using the R-package XCMS [23]"
- [abstract] No univariate statistics available: "Note that no univariate statistics tools are included in the current implementation"
- [readme] Dataset location on the VM: "the Mass Spectra can be found in : * `~/GalaxyM-TestData/LCMS_DATA` * `~/GalaxyM-TestData/DIMS_DATA`"
- [abstract] Reproducibility recommendation via published workflows: "We recommend that Galaxy-M workflow files are included within the supplementary information of publications, enabling metabolomics studies to achieve greater reproducibility"
- [readme] Wine 32-bit requirement for SimStitch: "you will likely get errors from the SimStitch tools along the lines of 'Invalid file identifier'"
