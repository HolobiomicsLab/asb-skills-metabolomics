---
name: metabolomics-project-batch-import
description: Use when Thermo .raw or SCIEX .wiff files have been converted to Aird
  format and must be loaded into MetaPro as organized projects/batches before library
  assignment and Target Search.
when_to_use_negative:
- Data is still in vendor format (.raw/.wiff) and has not been converted to Aird with
  AirdPro — MetaPro imports Aird files, not vendor files
- Analysis goal is untargeted metabolomics feature detection rather than semi-targeted
  analysis with an analyte/IS library, which is MetaPro's supported workflow
- Data were acquired with MRM/SRM or other non-DDA acquisition methods outside MetaPro's
  supported DDA scope
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MetaPro v1.1.8
  role: Web application whose Project page Scan Files function imports the folder-arranged
    Aird batches
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro v1.1
  role: GUI client used upstream to convert vendor .raw/.wiff files into Aird format
    in DDA mode before import
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: Redis
  role: Backend service started alongside MetaPro (and optionally used by AirdPro
    distributed conversion) that must be running for MetaPro startup
- name: MongoDB
  role: Backend database started with MetaPro that persists project/batch metadata
    during import
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:a6cc444a01bd66018a59207f33902f5b203f2823598809fbffb6cacab113eab2
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# metabolomics-project-batch-import

## Summary

Import vendor-converted LC-MS data into MetaPro by arranging Aird files (.aird + .) under a project/batch folder hierarchy and using the Scan Files function, enabling efficient multi-batch import for semi-targeted analysis on instruments such as the AB SCIEX TripleTOF 6600 and Thermo Q Exactive HF.

## When to use

Use when Thermo .raw or SCIEX .wiff files have been converted to Aird format and must be loaded into MetaPro as organized projects/batches before library assignment and Target Search.

## When NOT to use

- Data is still in vendor format (.raw/.wiff) and has not been converted to Aird with AirdPro — MetaPro imports Aird files, not vendor files
- Analysis goal is untargeted metabolomics feature detection rather than semi-targeted analysis with an analyte/IS library, which is MetaPro's supported workflow
- Data were acquired with MRM/SRM or other non-DDA acquisition methods outside MetaPro's supported DDA scope

## Inputs

- Aird-converted MS files (.aird data file plus . index file) in DDA mode
- Vendor files already converted with AirdPro 1.1 (Thermo .raw, SCIEX .wiff sources)
- Project/batch folder hierarchy inside the MetaPro analysis data directory

## Outputs

- Imported batches visible on the MetaPro Project page
- Batch assignments ready for library (ANALYTES/IS) and Method attachment
- Multi-batch dataset (e.g., TripleTOF dataset: 8 converted instrument data / 16 files; QE HF dataset: 10 converted instrument data / 20 files)

## How to apply

First convert vendor MS files (Thermo .raw, SCIEX .wiff) to Aird format (.aird plus . index) with AirdPro in DDA mode; the .aird and . index files must be stored in the same directory with the same file name so MetaPro's scanner can pair them. Arrange the converted files under a project/batch folder hierarchy in the MetaPro analysis data directory, since MetaPro imports multi-batch files by scanning folder structure rather than by individual file selection. Open the Project page in MetaPro v1.1.8 and use the Scan Files function to import the batches; each batch corresponds to one instrument platform (e.g., AB SCIEX TripleTOF 6600 or Thermo Q Exactive HF) so that a platform-specific Method can be assigned later. After import, verify the file count per batch matches the source dataset (e.g., 16 files for the TripleTOF dataset, 20 files for the QE HF dataset) before proceeding to library loading and Target Search.

## Related tools

- **MetaPro v1.1.8** (Web application whose Project page Scan Files function imports the folder-arranged Aird batches) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro v1.1** (GUI client used upstream to convert vendor .raw/.wiff files into Aird format in DDA mode before import) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **Redis** (Backend service started alongside MetaPro (and optionally used by AirdPro distributed conversion) that must be running for MetaPro startup)
- **MongoDB** (Backend database started with MetaPro that persists project/batch metadata during import)

## Examples

```
bash MetaproStart.sh  # start MetaPro (Redis, MongoDB, MetaPro) then open localhost:8080 -> Project page -> Scan Files
```

## Evaluation signals

- Each imported batch pairs a .aird data file with a same-named . index file in the same directory (required by AirdScanUtil)
- File counts per batch match the source dataset (TripleTOF: 16 files across 8 converted instrument data; QE HF: 20 files across 10 converted instrument data)
- Batches appear correctly grouped by instrument platform on the MetaPro Project page so the correct Method (TripleTOF 6600 vs Q Exactive HF presets) can be assigned
- MetaPro startup succeeded (Redis, MongoDB, MetaPro services running) and localhost:8080 is reachable before import
- No manual QC inspection or library curation is performed at this stage when reproducing the paper's first automatic run

## Limitations

- MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method; other acquisition modes may not import correctly
- Aird index (.) and data (.aird) files must share the same directory and file name or the scanner cannot detect them
- AirdPro conversion precision defaults to mzPrecision = 0.0001; changing it affects downstream search fidelity
- Folder hierarchy must follow MetaPro's expected project/batch structure; arbitrary layouts will not be scanned into batches

## Evidence

- [methods] MetaPro enables efficient multi-batch files import by scanning folder structure: "**Step 2: File directory arrangement** MetaPro enables efficient multi-batch files import by scanning folder structure"
- [methods] MetaPro supports DDA data: "**Step 1: File Conversion** MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method."
- [methods] Benchmark dataset file counts: "- **TripleTOF dataset** - 8 converted instrument data (16 files)"
- [readme] Aird index/data pairing requirement: [evidence span withheld — anchor status: external_doc]
- [readme] Conversion type parameter for AirdPro jobs: [evidence span withheld — anchor status: external_doc]
