---
name: aird-format-ms-data-loading
description: Use when vendor DDA LC-MS files (.wiff, .raw) must be converted to Aird
  format and imported into MetaPro for multi-batch target search or QC inspection.
when_to_use_negative:
- Data are already converted to .aird/.json pairs — go straight to loading into MetaPro
  rather than reconverting
- The downstream analysis tool does not read the Aird format (e.g., pipelines expecting
  mzML/mzXML) — although AirdPro can export mzML/mzXML, this skill targets MetaPro
  ingestion
- Acquisition is a mode unsupported by the AirdPro version in use (e.g., MRM/SRM or
  wiff2 requires AirdPro V5.1.0 or later, not the v1.1 release referenced by MetaPro)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AirdPro
  role: GUI client that converts vendor .wiff/.raw files to the Aird format in DDA
    mode before MetaPro loading
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: MetaPro
  role: Web application that scans the project/batch folders of converted Aird files
    and loads them for Target Search and batch QC inspection
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: Redis
  role: Backing service started with MetaPro; also supports distributed batch conversion
    tasks in AirdPro via the ConvertTask channel
- name: MongoDB
  role: Backing service started alongside MetaPro during installation startup
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:2ca7a2d8900e2bd3f87591aa23e7641be056ec130df016649f66775b5eaea45e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MetaPro
  - MongoDB
  - Redis
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# aird-format-ms-data-loading

## Summary

Converts vendor LC-MS instrument files (TripleTOF .wiff, QE HF .raw) into the Aird computation-oriented format with AirdPro and loads them into MetaPro (backed by Redis and MongoDB) for batch semi-targeted analysis and QC inspection.

## When to use

Use when vendor DDA LC-MS files (.wiff, .raw) must be converted to Aird format and imported into MetaPro for multi-batch target search or QC inspection.

## When NOT to use

- Data are already converted to .aird/.json pairs — go straight to loading into MetaPro rather than reconverting
- The downstream analysis tool does not read the Aird format (e.g., pipelines expecting mzML/mzXML) — although AirdPro can export mzML/mzXML, this skill targets MetaPro ingestion
- Acquisition is a mode unsupported by the AirdPro version in use (e.g., MRM/SRM or wiff2 requires AirdPro V5.1.0 or later, not the v1.1 release referenced by MetaPro)

## Inputs

- Vendor MS instrument files: TripleTOF .wiff (and .wiff2 in later AirdPro versions) and QE HF .raw files acquired in DDA mode
- Project/batch folder structure containing the converted files

## Outputs

- Paired Aird files: .aird data files and .json index files with matching filenames in the same directory
- Loaded batches visible in the MetaPro web interface (localhost:8080) ready for Target Search

## How to apply

Run AirdPro v1.1 to convert vendor files in DDA mode, producing paired Aird index (.json) and data (.aird) files that must share the same filename and reside in the same directory so AirdScanUtil can scan them. Arrange the converted files under project/batch folders in the MetaPro v1.1.8 data directory, since MetaPro imports multi-batch files by scanning folder structure. Start MetaPro via MetaproStart.bat/sh, which launches Redis and MongoDB as backing services, then open localhost:8080 and use the Scan Files / Create Project function on the Project page to load the batches. Rationale: the Aird format's compression and indexing (e.g., ZDPD, Aird-Slice) speeds XIC computation, which is what MetaPro's Target Search relies on.

## Related tools

- **AirdPro** (GUI client that converts vendor .wiff/.raw files to the Aird format in DDA mode before MetaPro loading) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **MetaPro** (Web application that scans the project/batch folders of converted Aird files and loads them for Target Search and batch QC inspection) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **Redis** (Backing service started with MetaPro; also supports distributed batch conversion tasks in AirdPro via the ConvertTask channel)
- **MongoDB** (Backing service started alongside MetaPro during installation startup)

## Examples

```
bash MetaproStart.sh  # launches Redis, MongoDB, and MetaPro; then open http://localhost:8080 and use Scan Files / Create Project
```

## Evaluation signals

- Every converted compound has both a .aird data file and a .json index file with the same filename in the same directory (AirdScanUtil requirement)
- MetaPro's Scan Files function discovers and lists all converted instrument data under the project/batch folders (e.g., 8 TripleTOF datasets / 16 files, 10 QE HF datasets / 20 files in the benchmark)
- The MetaPro startup terminal shows the sequence 'Start Redis', 'Start MongoDB', 'Start Metapro' and localhost:8080 serves the web interface
- Conversion was performed with type='DDA' (or the correct acquisition mode) since MetaPro supports DDA-acquired data

## Limitations

- MetaPro v1.1.8 is for academic use only; commercial use requires contacting the authors
- MetaPro supports MS data acquired with DDA; other acquisition strategies are outside the documented workflow for v1.1.8
- The .json index and .aird data file must share a filename and directory; misnamed or separated pairs will not be scanned by AirdScanUtil

## Evidence

- [methods] MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method.: "MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method."
- [methods] MetaPro enables efficient multi-batch files import by scanning folder structure: "MetaPro enables efficient multi-batch files import by scanning folder structure"
- [readme] Aird Index File and Aird Data File stored together: [evidence span withheld — anchor status: external_doc]
- [readme] Aird: a computation-oriented MS data format: [evidence span withheld — anchor status: not_found]
