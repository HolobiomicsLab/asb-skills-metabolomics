---
name: performance-speedup-computation
description: Use when comparing file-format read performance for MRM mass-spectrometry
  data, or when validating that Aird conversion delivers the reported ~33x full-file
  and ~2.5x single-chromatogram speedups over mzML.
when_to_use_negative:
- The goal is quantification accuracy or manual peak inspection rather than read-speed
  benchmarking — use the MRMPro analysis workflow instead
- Data are only available in vendor-native formats (.wiff2, .d) and cannot be converted
  to both Aird and mzML with AirdPro, so no paired comparison is possible
- Comparing compression ratio or file size alone rather than decoding/reading wall-clock
  time — this skill measures speed, not storage savings
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Aird-SDK
  role: Java-based SDK used to read Aird (.aird/.) files and time full-file decoding
    and single-chromatogram random access
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: AirdPro
  role: GUI client (C#, ProteoWizard-based) used to convert vendor MRM files into
    Aird format to create the benchmark inputs
  repo: https://github.com/CSi-Studio/AirdPro
- name: MRMPro
  role: Web-based MRM analysis platform whose fast rendering of hundreds of chromatograms
    motivates the Aird speed benchmarks
  repo: github.com/CSi-Studio/MRMPro-Doc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:3007876df7a049547b0b2ffe965b6b1a6e4ae6a81f89ef98a8ed9184642ab15c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird-SDK
  - AirdPro
  - MRMPro
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# performance-speedup-computation

## Summary

Benchmark the wall-clock reading speed of the Aird mass-spectrometry format against mzML for MRM data, computing speedup factors for full-file decoding and single-chromatogram random access. It quantifies the computational benefit of Aird's higher compression ratio and JSON-encoded metadata.

## When to use

Use when comparing file-format read performance for MRM mass-spectrometry data, or when validating that Aird conversion delivers the reported ~33x full-file and ~2.5x single-chromatogram speedups over mzML.

## When NOT to use

- The goal is quantification accuracy or manual peak inspection rather than read-speed benchmarking — use the MRMPro analysis workflow instead
- Data are only available in vendor-native formats (.wiff2, .d) and cannot be converted to both Aird and mzML with AirdPro, so no paired comparison is possible
- Comparing compression ratio or file size alone rather than decoding/reading wall-clock time — this skill measures speed, not storage savings

## Inputs

- Three evaluation MRM datasets available in both Aird (.aird + . index pair) and mzML formats
- Aird index files (.) paired with Aird data files (.aird) in the same directory with the same filename
- AirdSDK (Java, C#, or Python) and MZmine3 mzML reader

## Outputs

- Per-dataset wall-clock timing measurements for full-file decoding in both formats
- Per-dataset wall-clock timing measurements for single-chromatogram random access in both formats
- Average speedup factors (mzML time / Aird time) per access mode, written to benchmark_results.csv

## How to apply

Load each of three evaluation datasets in both Aird and mzML formats, using the Java-based AirdSDK for Aird and the MZmine3 mzML reader for mzML. Time full-file decoding by reading the entire content of each dataset in both formats and record elapsed wall-clock time. Time single-chromatogram random access by retrieving chromatograms at randomly selected indices, recording per-access elapsed time. Average the per-format times across all three datasets for each access mode, then compute speedup as mzML time divided by Aird time. Accept the benchmark as consistent if average speedups match the reported ~33x (full-file) and ~2.5x (random access) within reasonable tolerance; rationale is Aird's higher compression ratio and JSON-encoded metadata that avoid mzML's parsing overhead.

## Related tools

- **Aird-SDK** (Java-based SDK used to read Aird (.aird/.) files and time full-file decoding and single-chromatogram random access) — https://github.com/CSi-Studio/Aird-SDK
- **AirdPro** (GUI client (C#, ProteoWizard-based) used to convert vendor MRM files into Aird format to create the benchmark inputs) — https://github.com/CSi-Studio/AirdPro
- **MRMPro** (Web-based MRM analysis platform whose fast rendering of hundreds of chromatograms motivates the Aird speed benchmarks) — github.com/CSi-Studio/MRMPro-Doc

## Examples

```
pip install AirdSDK
```

## Evaluation signals

- Computed average full-file decoding speedup (mzML time / Aird time) is approximately 33x across the three datasets, within a reasonable tolerance
- Computed average single-chromatogram random-access speedup is approximately 2.5x, within a reasonable tolerance
- Timing measurements exist for all three datasets in both formats and both access modes (full-file and random access) before averaging
- benchmark_results.csv contains per-dataset timing rows plus the computed average speedup factors
- Each Aird input resolves as a paired .aird data file and . index file with the same filename in the same directory

## Limitations

- Speedup factors are averages over three specific MRM evaluation datasets; results may vary with chromatogram count, file size, and instrument vendor
- Benchmark uses a Java-based AirdSDK reader versus the MZmine3 mzML reader, so implementation differences (not just format) could contribute to measured differences
- Speed gains depend on Aird's compression strategy (e.g. ZDPD/ComboComp for m/z arrays); other compressor choices may shift decoding time
- The Aird-SDK README lists supported acquisition methods as DIA/SWATH, DDA, PRM, DIA_PASEF, DDA_PASEF — MRM chromatogram support requires a compatible Aird version with chromatogramIndex

## Evidence

- [readme] Aird index file and Aird data file should be stored in the same directory with the same file name but with different suffix: [evidence span withheld — anchor status: external_doc]
- [readme] Aird format enables higher compression ratio and less decoding time: "Aird: a computation-oriented mass spectrometry data format enables a higher compression ratio and less decoding time"
- [abstract] MRMPro's compression reduced data volume vs vendor and mzML formats: "which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format"
- [intro] Efficient compression algorithm needed for fast web-side chromatogram rendering: "An efficient compression algorithm to meet the needs of fast rendering of hundreds of chromatograms on the web side"
