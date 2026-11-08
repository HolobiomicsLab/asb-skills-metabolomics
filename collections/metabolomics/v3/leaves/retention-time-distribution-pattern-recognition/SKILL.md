---
name: retention-time-distribution-pattern-recognition
description: Use when running semi-targeted Target Search over multiple batches of
  DDA-acquired LC-MS files (e.g., TripleTOF .wiff or QE HF .raw converted to Aird)
  and you need automated, consistent peak selection before manual inspection.
when_to_use_negative:
- Data were acquired in modes other than DDA (MetaPro supports MS data acquired with
  the data-dependent acquisition method)
- MS files are still in vendor format (.wiff/.raw) and have not been converted to
  Aird with AirdPro
- The analysis goal is untargeted feature detection rather than semi-targeted quantification
  against an existing library with mz and rt values
edam_operation: http://edamontology.org/operation_3219
tools:
- name: MetaPro v1.1.8
  role: Web application whose Target Search result-optimization module runs the RT-distribution
    pattern recognition and multi-file EIC alignment; also hosts the batch QC inspection
    interface
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro v1.1
  role: GUI client used upstream to convert vendor MS files to Aird format so MetaPro
    can analyze them efficiently
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: Redis
  role: Backing service started alongside MetaPro that supports its in-memory caching
    and batch processing
- name: MongoDB
  role: Backing service started alongside MetaPro that stores analysis state for batch
    inspection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:798ca13782743bc6c9be77e3c3ccaf546fbce1ae29265319dab9584ea0808b3a
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
  edam_operation: http://edamontology.org/operation_3219
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Retention-time distribution pattern recognition for peak selection refinement

## Summary

MetaPro's automated result-optimization algorithm that refines peak selection in semi-targeted LC-MS batch analysis by recognizing patterns in the RT distributions of adjacent compounds in the spectral library, reducing manual inspection burden in batch QC.

## When to use

Use when running semi-targeted Target Search over multiple batches of DDA-acquired LC-MS files (e.g., TripleTOF .wiff or QE HF .raw converted to Aird) and you need automated, consistent peak selection before manual inspection.

## When NOT to use

- Data were acquired in modes other than DDA (MetaPro supports MS data acquired with the data-dependent acquisition method)
- MS files are still in vendor format (.wiff/.raw) and have not been converted to Aird with AirdPro
- The analysis goal is untargeted feature detection rather than semi-targeted quantification against an existing library with mz and rt values

## Inputs

- Vendor MS files (e.g., TripleTOF .wiff, QE HF .raw) converted to Aird format (.aird/.) in DDA mode
- Analyte and internal standard library tables (.txt/.tsv) with at least 'mz' and 'rt' columns
- MetaPro Method with preset analysis parameters (created on the Method page)
- Batch-folder-organized project directory loaded via MetaPro's Scan Files function

## Outputs

- Automatically selected chromatographic peaks per compound per file, refined by RT-distribution pattern recognition and multi-file EIC alignment
- Batch QC overview page showing per-compound peak-selection consistency status (Success/Failed/Unknow)
- Compounds flagged Failed or Unknow with the specific files showing inconsistent RT regions or integration bounds

## How to apply

Convert vendor MS files (TripleTOF .wiff, QE HF .raw) to Aird format (.aird + . index) with AirdPro v1.1 in DDA mode, then arrange them under project/batch folders so MetaPro v1.1.8 (with Redis and MongoDB as backing services) can load them via Scan Files. Upload analyte/internal-standard library tables containing at least 'mz' and 'rt' columns and create a Method with the preset analysis parameters. Run Target Search on the selected batches: the result-optimization module performs compound-by-compound multi-file EIC alignment for cross-file peak-selection consistency and applies RT-distribution pattern recognition using the retention times of adjacent compounds in the library to improve peak-selection accuracy. Inspect the results on the batch QC page, judging each compound by whether the auto-selected peak occupies the same RT region and integration bounds across all files in the batch (Success/Failed/Unknow).

## Related tools

- **MetaPro v1.1.8** (Web application whose Target Search result-optimization module runs the RT-distribution pattern recognition and multi-file EIC alignment; also hosts the batch QC inspection interface) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro v1.1** (GUI client used upstream to convert vendor MS files to Aird format so MetaPro can analyze them efficiently) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **Redis** (Backing service started alongside MetaPro that supports its in-memory caching and batch processing)
- **MongoDB** (Backing service started alongside MetaPro that stores analysis state for batch inspection)

## Examples

```
bash MetaproStart.sh  # starts Redis + MongoDB + MetaPro; then open browser at localhost:8080, create Project/Library/Method, and click Target Search on the loaded batches
```

## Evaluation signals

- Every compound whose auto-selected peaks occupy the same RT region and integration bounds across all files in a batch is recorded with a Success check status
- Any compound flagged Failed or Unknow is listed together with its inconsistent files, matching the MetaPro batch QC status scheme
- The library tables loaded into MetaPro contain at least the 'mz' and 'rt' columns required for RT-based pattern recognition
- All batches submitted to Target Search complete the result-optimization module before manual inspection begins, reducing the number of compounds needing manual integration

## Limitations

- The algorithm is semi-targeted and library-dependent: pattern recognition relies on RT distributions of adjacent compounds in the uploaded library, so a sparse or low-quality library weakens refinement
- MetaPro supports DDA-acquired data; other acquisition modes fall outside this workflow
- Manual inspection remains indispensable: the algorithm is designed to improve manual inspection efficiency, and manual functions (Manual Integration, Reset integration params, Reset TargetRT) exist for compounds flagged Failed or Unknow

## Evidence

- [abstract] Semi-targeted workflow combining judgment and algorithms: "A semi-targeted analysis workflow is designed based on the concept of combining artificial judgment and algorithm inference."
- [methods] Target Search over multiple batches: "Click `Target Search` to start a targeted analysis. You can select multiple batches and click `Target Search`"
- [methods] Batch QC inspection interface: "This is the main inspection page on samples in a batch."
- [readme] AirdPro converts vendor files to Aird: [evidence span withheld — anchor status: external_doc]
- [readme] MetaPro backing services: "********* Start Redis *********
/Users/commands/Documents/Documents/MetaPro/MetaproForMac/redis
********* Start MongoDB *********"
