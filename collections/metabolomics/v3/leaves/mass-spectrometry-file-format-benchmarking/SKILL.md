---
name: mass-spectrometry-file-format-benchmarking
description: Use when comparing read/decode performance of MS data formats (Aird,
  mzML) on the same MRM or DIA datasets, or when deciding whether converting vendor
  data to Aird will pay off in loading/rendering speed.
when_to_use_negative:
- The goal is comparing peak-picking or quantification accuracy between tools rather
  than raw file read/decode speed
- Only one format is available for the datasets — no paired Aird/mzML versions exist
  to time against each other
- The concern is conversion-time or file-size (compression ratio) rather than decoding/random-access
  speed, which the MRMPro/Aird papers report as separate metrics
tools:
- name: Aird-SDK
  role: Java/C#/Python SDK used to load and decode the Aird-format MRM files during
    timing measurements
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: AirdPro
  role: GUI client used to convert vendor/mzML files into Aird format so that paired-format
    benchmark inputs exist
  repo: https://github.com/CSi-Studio/AirdPro
- name: MRMPro
  role: Web-based MRM analysis platform whose need for fast rendering of hundreds
    of chromatograms motivates the Aird-vs-mzML reading-speed benchmark
  repo: github.com/CSi-Studio/MRMPro-Doc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:0ed422dd656be3b5a3b7d140c172ee96114569497d5e90a04872a524a13372c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird-SDK
  - AirdPro
  - MRMPro
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mass-spectrometry-file-format-benchmarking

## Summary

Benchmark mass-spectrometry data file formats (e.g. Aird vs mzML) by timing full-file decoding and single-chromatogram random access across the same datasets, then computing average speedup factors. Aird's higher compression ratio and JSON-encoded metadata make it roughly 33x faster than mzML for full MRM file reading and ~2.5x faster for single-chromatogram access.

## When to use

Use when comparing read/decode performance of MS data formats (Aird, mzML) on the same MRM or DIA datasets, or when deciding whether converting vendor data to Aird will pay off in loading/rendering speed.

## When NOT to use

- The goal is comparing peak-picking or quantification accuracy between tools rather than raw file read/decode speed
- Only one format is available for the datasets — no paired Aird/mzML versions exist to time against each other
- The concern is conversion-time or file-size (compression ratio) rather than decoding/random-access speed, which the MRMPro/Aird papers report as separate metrics

## Inputs

- Same MRM acquisition datasets stored in both Aird (.aird + . index) and mzML formats
- AirdSDK (Java, C#, or Python) for reading Aird files
- MZmine3 mzML reader for reading mzML files

## Outputs

- Per-dataset elapsed wall-clock times for full-file decoding in each format
- Per-dataset elapsed wall-clock times for single-chromatogram random access in each format
- benchmark_results.csv with averaged per-format times and computed speedup factors

## How to apply

Load each evaluation dataset in both Aird and mzML formats using the Java-based AirdSDK and the MZmine3 mzML reader respectively. Time full-file decoding by reading the entire file content in both formats and recording elapsed wall-clock time; separately time single-chromatogram random access by retrieving individual chromatograms at randomly selected indices. Repeat each measurement over all datasets and average per-format elapsed times, then compute speedup as mzML time divided by Aird time for each access mode. Accept the benchmark as consistent if average speedups match reported values (~33x full-file, ~2.5x random access) within reasonable tolerance, writing per-dataset timings and speedups to benchmark_results.csv. The rationale is that Aird's combinable compressors (e.g. ZDPD, Zlib-based ComboComp) yield smaller files and SIMD-optimized decoding, which translates directly into faster full-file and random-access reads.

## Related tools

- **Aird-SDK** (Java/C#/Python SDK used to load and decode the Aird-format MRM files during timing measurements) — https://github.com/CSi-Studio/Aird-SDK
- **AirdPro** (GUI client used to convert vendor/mzML files into Aird format so that paired-format benchmark inputs exist) — https://github.com/CSi-Studio/AirdPro
- **MRMPro** (Web-based MRM analysis platform whose need for fast rendering of hundreds of chromatograms motivates the Aird-vs-mzML reading-speed benchmark) — github.com/CSi-Studio/MRMPro-Doc

## Examples

```
pip install AirdSDK  # then time full-file decode of .aird/. vs MZmine3 mzML reader and write timings to benchmark_results.csv
```

## Evaluation signals

- Computed average full-file decoding speedup (mzML time / Aird time) is approximately 33x across all three datasets within reasonable tolerance
- Computed average single-chromatogram random-access speedup is approximately 2.5x within reasonable tolerance
- benchmark_results.csv contains timing rows for all three datasets in both formats, not just aggregated averages
- Aird . index and .aird data file are co-located with the same filename so AirdScanUtil can scan both files
- Both formats are read from the identical underlying acquisitions (converted via AirdPro), so timing differences are attributable to format rather than content

## Limitations

- Aird currently supports only specific acquisition types (DIA, DDA, PRM, DIA_PASEF, DDA_PASEF, MRM, MSI_MALDI, COMMON); formats outside this set cannot be benchmarked
- Speedup factors are averages over three evaluation datasets and may not generalize to other instruments or acquisition methods
- Random-access timing depends on randomly selected chromatogram indices, so results can vary across index draws; repeat measurements and averaging are required

## Evidence

- [full_text] For full MRM file reading, the Aird format is about 33 times faster than the mzML format on average, attributed to its higher compression ratio and JSON-encoded metadata.: [evidence span withheld — anchor status: not_found]
- [abstract] Aird compression ratio claim: "which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format"
- [readme] Aird format description: [evidence span withheld — anchor status: external_doc]
- [readme] Aird index/data file layout: [evidence span withheld — anchor status: external_doc]
