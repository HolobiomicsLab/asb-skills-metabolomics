---
name: quantification-report-export
description: Use when a finished MetaPro Target Search run on Aird-converted LC-MS
  batches (e.g. TripleTOF 6600 or Q Exactive HF) needs its peak-area results extracted
  as a spreadsheet for downstream fold-change or benchmarking analysis.
when_to_use_negative:
- The goal is curated, manually verified quantification — this skill reproduces the
  no-inspection first run and deliberately skips Manual Integration, Reset TargetRT,
  and integration-parameter resets
- Input data are still vendor files (.raw, .wiff) not yet converted to Aird format
  or imported via Scan Files
- The analysis is untargeted metabolomics; MetaPro's Export Report covers the semi-targeted
  (library-driven) workflow only
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaPro
  role: Web application whose Export Report function produces the .xlsx quantification
    overview after Target Search completes
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro
  role: Converts vendor .raw/.wiff files to Aird format so batches can be searched
    and their results exported
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:b63315d64c1b2b4d9ce06a01cb6a1627c22f4af45cbfe3193c6dbe3c4536ed05
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MetaPro
  - MongoDB
  - Redis
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# quantification-report-export

## Summary

Export MetaPro's semi-targeted quantification results via the Export Report function as an .xlsx overview of peak areas, then derive per-compound log-transformed blank/analyte fold changes (SB/SA) to assess quantification accuracy and stability without any manual peak curation.

## When to use

Use when a finished MetaPro Target Search run on Aird-converted LC-MS batches (e.g. TripleTOF 6600 or Q Exactive HF) needs its peak-area results extracted as a spreadsheet for downstream fold-change or benchmarking analysis.

## When NOT to use

- The goal is curated, manually verified quantification — this skill reproduces the no-inspection first run and deliberately skips Manual Integration, Reset TargetRT, and integration-parameter resets
- Input data are still vendor files (.raw, .wiff) not yet converted to Aird format or imported via Scan Files
- The analysis is untargeted metabolomics; MetaPro's Export Report covers the semi-targeted (library-driven) workflow only

## Inputs

- Completed MetaPro v1.1.8 Target Search run on batches of Aird-converted (.aird + .) TripleTOF 6600 and Q Exactive HF DDA data
- Assigned ANALYTES and internal standard (IS) libraries (.txt/.tsv with mz and rt header items)
- Method per instrument platform created with preset recommended parameters

## Outputs

- Quantification results overview exported as an .xlsx file (per-compound peak areas)
- Per-compound log-transformed SB/SA fold changes computed from the exported peak areas

## How to apply

After Target Search completes on all batches (monitored via the MetaPro v1.1.8 Task page), use the Export Report function to export the results overview as an .xlsx file containing per-compound peak areas for analyte (SA) and blank (SB) conditions. Skip any manual QC steps — no 'Reset integration params', 'Reset TargetRT', or 'Manual Integration' — if the goal is to reproduce the paper's first, fully automatic run. From the exported peak areas, compute per-compound log-transformed fold changes of signal in blank (SB) relative to analyte sample (SA), matching the Fig. 4B/4F comparison conditions. Correctness is judged by comparing the automatic-run result distribution before and after manual inspection in the benchmark datasets: a stable distribution indicates the export captured accurate quantification.

## Related tools

- **MetaPro** (Web application whose Export Report function produces the .xlsx quantification overview after Target Search completes) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro** (Converts vendor .raw/.wiff files to Aird format so batches can be searched and their results exported) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1

## Examples

```
MetaPro v1.1.8 → Batch page → select all batches → Target Search → Task page (wait for completion) → Export Report → results.xlsx; then in Python: log2_fc = np.log2(df['SB_area'] / df['SA_area'])
```

## Evaluation signals

- Exported .xlsx opens and contains peak-area rows for the compounds in the ANALYTES library across all searched batches
- Per-compound log2(SB/SA) fold-change distribution from the automatic run matches the distribution obtained after manual inspection (the paper's Fig. 4B/4F benchmark comparison)
- No compounds required post-hoc 'Reset integration params' or 'Manual Integration' for the first-run reproduction
- Both instrument platforms (AB SCIEX TripleTOF 6600 and Thermo Q Exactive HF) were searched with their own preset-recommended Method and both batches appear in the export

## Limitations

- Quantification accuracy is inferred by comparing automatic results to manually inspected results; without that reference the export alone does not prove accuracy
- MetaPro v1.1.8 supports DDA-acquired data only; MRM/DIA-SWATH vendor data require other AirdPro modes and are outside this workflow
- First-run results depend on library quality (mz/rt tables); poor library entries propagate into the exported areas even without manual inspection

## Evidence

- [methods] Export Report produces the quantification results as an .xlsx file: [evidence span withheld — anchor status: not_found]
- [methods] Compute per-compound log-transformed SB/SA fold changes from exported peak areas: [evidence span withheld — anchor status: not_found]
- [results] First automatic run demonstrated accuracy and stability against manual inspection: [evidence span withheld — anchor status: not_found]
- [methods] Target Search starts the semi-targeted analysis on selected batches: "Click `Target Search` to start a targeted analysis. You can select multiple batches and click `Target Search`"
