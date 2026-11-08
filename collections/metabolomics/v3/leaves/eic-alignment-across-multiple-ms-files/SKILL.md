---
name: eic-alignment-across-multiple-ms-files
description: Use when a batch of converted LC-MS files (.aird from TripleTOF .wiff
  or QE HF .raw) has been Target-Searched in MetaPro against a library with 'mz' and
  'rt' columns, and you need consistent automated peak selection per compound across
  all batch files prior to manual QC inspection.
when_to_use_negative:
- The data are untargeted DIA/SWATH acquisitions outside MetaPro's semi-targeted DDA
  workflow
- Only a single file is analyzed (no batch), so cross-file EIC alignment has nothing
  to enforce consistency over
- The input is already a processed feature/quantification table rather than raw converted
  MS data requiring peak picking
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MetaPro v1.1.8
  role: Web application running the Target Search result-optimization module (multi-file
    EIC alignment and RT pattern recognition) and the batch QC inspection interface
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro v1.1
  role: GUI client converting vendor .wiff/.raw files into the computation-oriented
    Aird format consumed by MetaPro
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: Redis
  role: Backing service started by MetaPro's start scripts (also used by AirdPro for
    distributed conversion tasks)
- name: MongoDB
  role: Backing service started by MetaPro's start scripts supporting the web application
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:64259dfa54cd47f61b65dd4c09b29cc7482f3503a9f54b17f3717d14a65811c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - AirdPro v1.1
  - MetaPro
  - MetaPro v1.1.8
  - MongoDB
  - Redis
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# eic-alignment-across-multiple-ms-files

## Summary

MetaPro's batch result-optimization capability: a compound-by-compound multi-file EIC alignment that enforces consistent peak selection (same RT region and integration bounds) across all files in a batch, plus RT-distribution pattern recognition using adjacent library compounds to refine peak selection before manual inspection.

## When to use

Use when a batch of converted LC-MS files (.aird from TripleTOF .wiff or QE HF .raw) has been Target-Searched in MetaPro against a library with 'mz' and 'rt' columns, and you need consistent automated peak selection per compound across all batch files prior to manual QC inspection.

## When NOT to use

- The data are untargeted DIA/SWATH acquisitions outside MetaPro's semi-targeted DDA workflow
- Only a single file is analyzed (no batch), so cross-file EIC alignment has nothing to enforce consistency over
- The input is already a processed feature/quantification table rather than raw converted MS data requiring peak picking

## Inputs

- Vendor LC-MS DDA files (TripleTOF .wiff, QE HF .raw) converted to Aird format (.aird + .json index via AirdPro v1.1)
- Project/batch folder structure importable by MetaPro v1.1.8 via Scan Files
- Analyte and internal standard library tables (.txt/.tsv) containing at least 'mz' and 'rt' columns
- MetaPro Method with preset analysis parameters

## Outputs

- Per-compound auto-selected peaks with consistent RT region and integration bounds across all batch files
- Batch QC overview on the MetaPro batch QC inspection interface with Success/Failed/Unknow status per compound
- peak_selection_consistency_report.csv listing per-compound consistency status and, for Failed/Unknow compounds, the inconsistent files

## How to apply

Convert vendor files to Aird format with AirdPro v1.1 in DDA mode, arrange them under project/batch folders in the MetaPro v1.1.8 data directory (with Redis and MongoDB running), load them via Scan Files, and upload analyte/internal-standard libraries (.txt/.tsv with at least 'mz' and 'rt' columns) into a Method. Run Target Search on the selected batches: the result-optimization module then performs compound-by-compound EIC alignment across all experiments in each batch so the same RT region and integration bounds are chosen per compound, and applies pattern recognition on the RT distributions of adjacent compounds in the library to improve peak-selection accuracy. Inspect the batch QC page and, for each compound, record whether the auto-selected peak is consistent across all files; export per-compound consistency as a Success/Failed/Unknow status report. The rationale is combining algorithm inference with artificial judgment, so ambiguous peaks (inconsistent across files) are exactly the ones routed to manual integration.

## Related tools

- **MetaPro v1.1.8** (Web application running the Target Search result-optimization module (multi-file EIC alignment and RT pattern recognition) and the batch QC inspection interface) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro v1.1** (GUI client converting vendor .wiff/.raw files into the computation-oriented Aird format consumed by MetaPro) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **Redis** (Backing service started by MetaPro's start scripts (also used by AirdPro for distributed conversion tasks))
- **MongoDB** (Backing service started by MetaPro's start scripts supporting the web application)

## Evaluation signals

- Every compound whose auto-selected peaks are consistent across all batch files is recorded with a Success check status on the batch QC page
- Any compound flagged Failed or Unknow is listed together with its inconsistent files, matching the MetaPro batch QC status scheme
- Consistent compounds share the same RT region and integration bounds across all files in the batch
- The export produces peak_selection_consistency_report.csv with per-compound consistency status

## Limitations

- MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method, limiting applicability to DDA-converted data
- Library quality gates accuracy: pattern recognition relies on RT distributions of adjacent compounds in the uploaded library, so sparse or low-quality libraries weaken peak selection
- Manual inspection remains indispensable; the algorithm only reduces, not eliminates, manual verification of quantification results

## Evidence

- [other] compound-by-compound multi-file EIC alignment and RT-based pattern recognition: [evidence span withheld — anchor status: not_found]
- [methods] workflow steps: "Click `Target Search` to start a targeted analysis. You can select multiple batches and click `Target Search`"
- [methods] batch QC interface: "#### 4.3.2 Batch Quality Control inspection interface This is the main inspection page on samples in a batch."
- [intro] manual inspection rationale: "the reproducibility crisis in metabolomics and the lack of transparency in quality assurance (QA) and quality control (QC) might lead to ambiguity among labs"
- [readme] file conversion prerequisite: "Aird Index File Suffix: .json <br/>
Aird Data File Suffix: .aird <br/>
Aird Index File and Aird Data File show be stored in the same directory with the same file name"
