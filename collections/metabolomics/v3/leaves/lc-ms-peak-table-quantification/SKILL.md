---
name: lc-ms-peak-table-quantification
description: Use when raw LC-MS metabolomics data (.mzML or .abf) needs to be converted
  into a quantified peak table with peak detection, alignment, and filtering; especially
  when reproducibility across runs, containerization, or HPC execution is required.
when_to_use_negative:
- The input is already a processed peak table or feature matrix, not raw spectral
  data; Nextflow4MS-DIAL expects .mzML or .abf raw files.
- The user needs a graphical interface or web-based upload; Nextflow4MS-DIAL is command-line-only
  and lacks a GUI.
- The system is Apple silicon (M-series); the workflow has only been tested on Intel-based
  macOS and Linux/Red Hat Enterprise Linux 8.8.
- The user wants interactive statistical analysis or pathway enrichment; the workflow
  stops at peak table generation and does not include downstream statistical or pathway
  tools.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Nextflow4MS-DIAL
  role: Containerized Nextflow workflow orchestrating MS-DIAL peak detection and MS-FLO
    filtering for LC-MS metabolomics
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: Performs peak detection, m/z and retention-time identification, and intensity
    calculation on LC-MS data
- name: MS-FLO
  role: Executes duplicate peak flagging, adduct/isotope flagging, peak alignment,
    and contaminant ion removal on the MS-DIAL output
- name: Nextflow
  role: Workflow engine that supports Docker/Singularity containerization and execution
    on local or HPC systems
  repo: https://www.nextflow.io/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:2282a97a4b911677262820bc71048ef47e58b29f99471e0d0ab1652ac5736f82
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-DIAL
  - MS-FLO
  - Nextflow
  - Nextflow4MS-DIAL
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# lc-ms-peak-table-quantification

## Summary

Converts raw LC-MS/MS spectral data (mzML or ABF) into a quantified, filtered peak table using the Nextflow4MS-DIAL workflow, which chains MS-DIAL peak detection/alignment with MS-FLO filtering, adduct/isotope flagging, and contaminant ion removal to produce a reproducible peak table ready for statistical analysis.

## When to use

Use when raw LC-MS metabolomics data (.mzML or .abf) needs to be converted into a quantified peak table with peak detection, alignment, and filtering; especially when reproducibility across runs, containerization, or HPC execution is required.

## When NOT to use

- The input is already a processed peak table or feature matrix, not raw spectral data; Nextflow4MS-DIAL expects .mzML or .abf raw files.
- The user needs a graphical interface or web-based upload; Nextflow4MS-DIAL is command-line-only and lacks a GUI.
- The system is Apple silicon (M-series); the workflow has only been tested on Intel-based macOS and Linux/Red Hat Enterprise Linux 8.8.
- The user wants interactive statistical analysis or pathway enrichment; the workflow stops at peak table generation and does not include downstream statistical or pathway tools.

## Inputs

- Raw LC-MS/MS data in .mzML or .abf format
- MS-DIAL parameter configuration file (msdial_params.txt)
- MS-FLO parameter configuration file (msflo_params.ini)
- MS1 library file (ms1_lib.txt)
- MS2 library file (ms2_lib.msp)

## Outputs

- MS-DIAL peak table (.msdial, renamed to .tsv for spreadsheet compatibility)
- MS-FLO filtered/aligned peak table
- Nextflow execution logs (execution_report.html, execution_timeline.html, execution.log)

## How to apply

1) Confirm the input data is in .mzML format (convert with ProteoWizard msConvert) or .abf format (create with Reifycs Abf Converter), and place raw files in data/raw_data/. 2) Prepare msdial_params.txt and msflo_params.ini configuration files plus ms1_lib.txt and ms2_lib.msp metabolite library files in the data/ folder. 3) Run 'nextflow run main.nf -profile docker' locally or '-profile singularity' on HPC to trigger the pipeline. 4) MS-DIAL performs comprehensive LC-focused data processing — peak detection, m/z and retention-time identification, and intensity calculation — then automatically transfers the peak table to MS-FLO. 5) MS-FLO executes duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal to yield the final quantified table. 6) Verify that the .msdial outputs (renamed to .tsv) and downstream MS-FLO tables match the article's reported counts: 25,447 MS-DIAL peaks with 1,031 true positives (4.05%) and 25,414 MS-FLO peaks with 1,030 true positives on the MTBLS733 reference dataset.

## Related tools

- **Nextflow4MS-DIAL** (Containerized Nextflow workflow orchestrating MS-DIAL peak detection and MS-FLO filtering for LC-MS metabolomics) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (Performs peak detection, m/z and retention-time identification, and intensity calculation on LC-MS data)
- **MS-FLO** (Executes duplicate peak flagging, adduct/isotope flagging, peak alignment, and contaminant ion removal on the MS-DIAL output)
- **Nextflow** (Workflow engine that supports Docker/Singularity containerization and execution on local or HPC systems) — https://www.nextflow.io/

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- Peak counts match the article's reported values: 25,447 MS-DIAL peaks and 25,414 MS-FLO peaks on the MTBLS733 dataset.
- True-positive peak counts match the article's reported values: 1,031 (4.05%) for MS-DIAL and 1,030 (4.05%) for MS-FLO.
- Five repeated executions of the workflow produce identical results, demonstrating consistent reproducibility.
- Output files use the expected extensions (.msdial renamed to .tsv) and reside in the results/ folder.
- Nextflow execution logs (execution_report.html, execution_timeline.html, execution.log) document runtime, resource usage, container info, and process-level completion.

## Limitations

- The workflow is command-line-only; there is no graphical user interface for users with limited computational experience.
- MS-FLO must be downloaded from the Internet rather than being bundled with the workflow.
- Workflow components such as MS-DIAL are updated frequently; updates are currently monitored manually with no efficient maintenance plan for the workflow and container.
- The workflow has not been tested on Apple silicon (M-series) systems.
- Only 10 samples were used in the published test dataset, limiting scalability assessment.
- Additional tools for data file format conversion, statistical analysis, and pathway analysis are not yet incorporated.
- The workflow is not yet deposited to WorkflowHub, so FAIRness of data provenance is not yet enhanced.

## Evidence

- [results] MS-DIAL identified 25,447 peaks and MS-FLO generated 25,414 peaks; MS-DIAL detected 1,031 true positive peaks (4.05%) and MS-FLO identified 1,030 true positive peaks.: [evidence span withheld — anchor status: not_found]
- [readme] Workflow accepts .mzML and .abf raw files and requires msdial_params.txt, msflo_params.ini, ms1_lib.txt, and ms2_lib.msp: [evidence span withheld — anchor status: not_found]
