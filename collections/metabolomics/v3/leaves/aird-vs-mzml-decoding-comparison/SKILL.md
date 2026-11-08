---
name: aird-vs-mzml-decoding-comparison
description: Use when choosing or validating an MS data storage format for MRM acquisition
  files, or when replicating decoding-speed benchmarks between Aird and mzML on full-file
  and random-access reading modes.
when_to_use_negative:
- The question concerns quantification accuracy or manual inspection features of MRM
  analysis tools (e.g. MRMPro vs Skyline) rather than raw data-format decoding speed
- The input is already an analysis result (e.g. xlsx identification results or a feature
  table) rather than raw MRM spectral data in Aird or mzML
- The data are DDA/DIA/PASEF rather than MRM, since this benchmark is specific to
  full MRM file reading and MRM chromatogram random access
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Aird-SDK
  role: Java-based SDK used to load and time the decoding of Aird-format MRM files
    for the benchmark
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: AirdPro
  role: C# GUI client used to convert vendor MRM files into the Aird format (supports
    MRM acquisition since V5.1.0) prior to benchmarking
  repo: https://github.com/CSi-Studio/AirdPro
- name: MRMPro
  role: Web-based MRM analysis platform whose fast rendering depends on the Aird compression
    and decoding performance being benchmarked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:e66a7cfca9a2c9214599b8fc1dfea46e150c38af9e28a6e72669b6be7738b6fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird-SDK
  - AirdPro
  - MRMPro
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# aird-vs-mzml-decoding-comparison

## Summary

A benchmarking skill for quantifying the wall-clock decoding speedup of the Aird mass spectrometry format over mzML for MRM data, reporting approximately 33x for full-file reading and approximately 2.5x for single-chromatogram random access.

## When to use

Use when choosing or validating an MS data storage format for MRM acquisition files, or when replicating decoding-speed benchmarks between Aird and mzML on full-file and random-access reading modes.

## When NOT to use

- The question concerns quantification accuracy or manual inspection features of MRM analysis tools (e.g. MRMPro vs Skyline) rather than raw data-format decoding speed
- The input is already an analysis result (e.g. xlsx identification results or a feature table) rather than raw MRM spectral data in Aird or mzML
- The data are DDA/DIA/PASEF rather than MRM, since this benchmark is specific to full MRM file reading and MRM chromatogram random access

## Inputs

- MRM mass spectrometry datasets in vendor format converted to Aird (.aird + . index pair)
- The same MRM datasets in mzML format
- Three evaluation datasets in both formats

## Outputs

- Per-dataset timing measurements for full-file decoding and single-chromatogram random access (benchmark_results.csv)
- Average speedup factors per access mode (mzML time / Aird time), expected ~33x full-file and ~2.5x random access

## How to apply

Convert the vendor MRM files to Aird using AirdPro (which supports the MRM acquisition method since V5.1.0), producing an .aird data file plus a . index file stored in the same directory with the same file name. Load each of the three evaluation datasets in both Aird and mzML formats using the Java-based AirdSDK and the MZmine3 mzML reader respectively. Time full-file decoding by reading the entire file content, and time single-chromatogram random access by retrieving individual chromatograms at randomly selected indices, repeating each measurement over all three datasets and averaging per-format elapsed times. Compute speedup factors as mzML time divided by Aird time for each access mode; the rationale is Aird's higher compression ratio (more than 60% and 80% smaller than vendor and mzML respectively) and JSON-encoded metadata, with SIMD-optimized decoding.

## Related tools

- **Aird-SDK** (Java-based SDK used to load and time the decoding of Aird-format MRM files for the benchmark) — https://github.com/CSi-Studio/Aird-SDK
- **AirdPro** (C# GUI client used to convert vendor MRM files into the Aird format (supports MRM acquisition since V5.1.0) prior to benchmarking) — https://github.com/CSi-Studio/AirdPro
- **MRMPro** (Web-based MRM analysis platform whose fast rendering depends on the Aird compression and decoding performance being benchmarked)

## Examples

```
pip install AirdSDK
```

## Evaluation signals

- Computed average full-file speedup (mzML time / Aird time) is approximately 33x within a reasonable tolerance across the three evaluation datasets
- Computed average single-chromatogram random-access speedup is approximately 2.5x within a reasonable tolerance
- Per-dataset timing measurements for both formats and both access modes are present in benchmark_results.csv
- Each Aird dataset consists of an .aird data file and a . index file with the same file name in the same directory
- Aird file sizes are substantially smaller than mzML (article reports >60% and >80% reduction vs vendor and mzML formats respectively)

## Limitations

- The ~33x and ~2.5x speedups were measured on three evaluation MRM datasets with specific readers (AirdSDK for Aird, MZmine3 for mzML) and may not generalize to other datasets, acquisition modes, or reader implementations
- Aird requires the . index and .aird data files to be stored in the same directory with the same file name, adding deployment constraints versus a single mzML file
- AirdPro only added MRM acquisition support in V5.1.0, so older conversions of MRM data may not be possible

## Evidence

- [other] For full MRM file reading, the Aird format is about 33 times faster than the mzML format on average, attributed to its higher compression ratio and JSON-encoded metadata.: [evidence span withheld — anchor status: not_found]
- [abstract] Compression advantage of Aird over vendor and mzML formats: "which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format"
- [readme] SIMD-optimized decoding speed of Aird: [evidence span withheld — anchor status: external_doc]
- [readme] Aird file layout requirement (paired .aird and . index files): [evidence span withheld — anchor status: external_doc]
