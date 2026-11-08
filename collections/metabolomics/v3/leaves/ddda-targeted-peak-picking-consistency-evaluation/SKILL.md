---
name: ddda-targeted-peak-picking-consistency-evaluation
description: Use when a batch of DDA-acquired LC-MS vendor files (.wiff, .raw) has
  been converted to Aird and run through MetaPro Target Search, and you must record
  per-compound peak-selection consistency prior to manual inspection.
when_to_use_negative:
- The data are untargeted feature tables or already-quantified peak tables — this
  skill evaluates raw/converted file peak picking in MetaPro's Target Search workflow,
  not post-hoc feature tables.
- Only a single file is being inspected — the multi-file EIC alignment and RT-adjacency
  pattern recognition require multiple experiments within a batch to assess consistency.
- The acquisition is MRM/SRM or another non-DDA mode — MetaPro's documented workflow
  here supports data-dependent acquisition (DDA) data.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaPro
  role: Runs Target Search, performs multi-file EIC alignment and RT-distribution
    pattern recognition, and exposes the batch QC inspection page used to record Success/Failed/Unknow
    peak-selection status
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro
  role: Converts vendor .wiff/.raw files to Aird format (.aird) in DDA mode so they
    can be scanned and loaded into MetaPro batches
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: Redis
  role: Backing service required by MetaPro for in-memory caching to speed up analysis
    of the loaded MS data
- name: MongoDB
  role: Backing service started alongside MetaPro to persist project, library, method,
    and analysis state
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:aeb4527c0b32a4836386465a23ae2df96dfccb129d24cea8232a7694fe30b6ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MetaPro
  - MongoDB
  - Redis
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ddda-targeted-peak-picking-consistency-evaluation

## Summary

Evaluate whether automatically selected LC-MS peaks for each compound are consistent (same RT region and integration bounds) across all files in a batch, using MetaPro's batch QC inspection interface backed by its multi-file EIC alignment and RT-distribution pattern recognition algorithms, to decide which compounds still need manual curation.

## When to use

Use when a batch of DDA-acquired LC-MS vendor files (.wiff, .raw) has been converted to Aird and run through MetaPro Target Search, and you must record per-compound peak-selection consistency prior to manual inspection.

## When NOT to use

- The data are untargeted feature tables or already-quantified peak tables — this skill evaluates raw/converted file peak picking in MetaPro's Target Search workflow, not post-hoc feature tables.
- Only a single file is being inspected — the multi-file EIC alignment and RT-adjacency pattern recognition require multiple experiments within a batch to assess consistency.
- The acquisition is MRM/SRM or another non-DDA mode — MetaPro's documented workflow here supports data-dependent acquisition (DDA) data.

## Inputs

- Batch of Aird-converted instrument files (.aird with .json index) from TripleTOF .wiff or QE HF .raw data, arranged under project/batch folders
- MetaPro Target Search results for the batch (backed by Redis and MongoDB)
- Analyte and internal standard library tables (.txt/.tsv with at least 'mz' and 'rt' columns)

## Outputs

- Per-compound peak-selection consistency status (Success/Failed/Unknow) per the MetaPro batch QC interface
- peak_selection_consistency_report.csv listing each compound's status and, for Failed/Unknow compounds, the files with inconsistent peak selection

## How to apply

After running Target Search in MetaPro v1.1.8 on the loaded batch, open the Batch Quality Control inspection interface, which is the main inspection page for samples in a batch. For each compound, check whether the auto-selected peak occupies the same RT region and integration bounds across every file in the batch; MetaPro's result-optimization module supports this by applying a compound-by-compound multi-file EIC alignment (to enforce peak-selection consistency) and a pattern recognition algorithm that refines peak selection from the RT distributions of adjacent compounds in the library. Record the per-compound status using the interface's Success/Failed/Unknow scheme, listing the inconsistent files for any compound flagged Failed or Unknow. Compounds flagged inconsistent can then be refined with 'Reset integration params', 'Reset TargetRT', or 'Manual Integration' (choosing two data points) before re-checking.

## Related tools

- **MetaPro** (Runs Target Search, performs multi-file EIC alignment and RT-distribution pattern recognition, and exposes the batch QC inspection page used to record Success/Failed/Unknow peak-selection status) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro** (Converts vendor .wiff/.raw files to Aird format (.aird) in DDA mode so they can be scanned and loaded into MetaPro batches) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **Redis** (Backing service required by MetaPro for in-memory caching to speed up analysis of the loaded MS data)
- **MongoDB** (Backing service started alongside MetaPro to persist project, library, method, and analysis state)

## Examples

```
# Start MetaPro (Redis + MongoDB auto-start), then browse to the Batch QC page
bash MetaproStart.sh  # -> http://localhost:8080 -> select batch -> Target Search -> Batch Quality Control inspection
```

## Evaluation signals

- Every compound whose auto-selected peaks are consistent (same RT region and integration bounds) across all batch files is recorded with a Success check status.
- Any compound flagged Failed or Unknow in the batch QC interface is listed in the report together with its specific inconsistent files.
- The compound-by-compound multi-file EIC alignment yields the same peak selection for a compound across all experiments in the batch.
- Refinement with the RT-distribution pattern recognition of adjacent library compounds moves previously inconsistent compounds toward consistent selection.
- Manual corrections applied via 'Reset integration params', 'Reset TargetRT', or 'Manual Integration' change the compound's status on re-inspection.

## Limitations

- MetaPro supports MS data acquired with the DDA method, so non-DDA acquisitions are outside this workflow.
- The pattern recognition depends on RT distributions of adjacent compounds in the library, so sparse library regions may leave compounds flagged Unknow.
- Accuracy depends on library quality; identification accuracy is highly dependent on the quality of the spectral library.
- Manual inspection remains indispensable for compounds the automated optimization cannot make consistent.

## Evidence

- [methods] multi-file EIC alignment and RT pattern recognition: [evidence span withheld — anchor status: not_found]
- [methods] batch QC inspection interface: "#### 4.3.2 Batch Quality Control inspection interface This is the main inspection page on samples in a batch."
- [methods] manual correction options: "users can `Reset integration params` and `Reset TargetRT` of a compound."
- [methods] manual integration of two points: "Click on `Manual Integration`, you can choose two data points to conduct manual integration functions."
- [methods] DDA-only support: "**Step 1: File Conversion** MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method."
- [readme] in-memory caching: "********* Start Redis *********
/Users/commands/Documents/MetaPro/MetaproForMac/redis
********* Start MongoDB *********"
