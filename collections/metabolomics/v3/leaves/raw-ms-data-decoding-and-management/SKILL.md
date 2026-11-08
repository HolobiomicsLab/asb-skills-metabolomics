---
name: raw-ms-data-decoding-and-management
description: Use when starting an untargeted LC-MS metabolomics pipeline from raw
  vendor data — e.g. a 100+ GB Astral MS file or a Bruker timsTOF DDA-PASEF .d directory
  — and scans must be loaded once and reused across feature detection, gap filling,
  and alignment.
when_to_use_negative:
- The data is already a processed feature table or aligned feature matrix — decoding
  is the first step from raw vendor files only.
- The .mcraw cache already exists for the project; reload it instead of re-decoding
  the source to avoid redundant parsing cost.
- The Bruker .d directory uses a compression, MsMsType, or calibration model outside
  the supported set (e.g. non-DDA-PASEF or quadratic model-1 with nonzero C0/C1/C2
  as decoded here).
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassCube
  role: Provides read_raw_file_to_obj for parsing mzML and supported Bruker TDF2 .d
    data into scan objects, and manages the .mcraw workflow cache.
  repo: https://github.com/huaxuyu/masscube/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:7ddac44af1fea98ee4e333d0dfd0ca1f67211aa5aeb021c7103fd463019a7e1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# raw-ms-data-decoding-and-management

## Summary

Decode raw LC-MS acquisition data (mzML files or supported Bruker TDF2 .d directories) into a scan-structured in-memory object with per-scan m/z/intensity signal arrays, then cache the decoded result as a versioned .mcraw file so downstream feature detection, gap filling, and aligned feature grouping reload the cache instead of reparsing the vendor source.

## When to use

Use when starting an untargeted LC-MS metabolomics pipeline from raw vendor data — e.g. a 100+ GB Astral MS file or a Bruker timsTOF DDA-PASEF .d directory — and scans must be loaded once and reused across feature detection, gap filling, and alignment.

## When NOT to use

- The data is already a processed feature table or aligned feature matrix — decoding is the first step from raw vendor files only.
- The .mcraw cache already exists for the project; reload it instead of re-decoding the source to avoid redundant parsing cost.
- The Bruker .d directory uses a compression, MsMsType, or calibration model outside the supported set (e.g. non-DDA-PASEF or quadratic model-1 with nonzero C0/C1/C2 as decoded here).

## Inputs

- Raw LC-MS acquisition file in mzML format
- Bruker timsTOF TDF2 .d directory (DDA-PASEF, TimsCompressionType=2, m/z calibration model 1 with dC2=0 and C3=C4=0, static TIMS calibration model 2)
- Optional: MASSCUBE_ZSTD_LIBRARY environment variable pointing to a nonstandard libzstd location on macOS

## Outputs

- In-memory masscube data object with .scans, .ms1_idx_arr, per-scan .signals (m/z, intensity columns), and for timsTOF data .inv_mobility and .inv_mobility_range
- Versioned .mcraw workflow cache written under the project tmp directory
- data.metadata.mobility_pressure_compensated == False record for timsTOF frames (static 1/K0 calibration only, no per-frame pressure compensation)

## How to apply

Import read_raw_file_to_obj from masscube and pass the raw file path: for mzML this parses the standard XML format; for Bruker .d directories it applies the TDF2 decoding constraints (TimsCompressionType=2, MsMsType=8 DDA-PASEF, m/z calibration model 1 with dC2=0 and C3=C4=0, static TIMS calibration model 2), loading system libzstd via ctypes rather than OpenTIMS, AlphaTims, or Bruker's TDF-SDK. The returned object exposes data.scans indexed by data.ms1_idx_arr, with each scan providing .signals columns (m/z, intensity), plus .inv_mobility and .inv_mobility_range for timsTOF frames. Once the untargeted workflow runs, a versioned .mcraw cache is written under the project tmp directory, storing shared MS1/MS2 metadata once and per-scan values plus concatenated peak arrays as NumPy columns; subsequent stages reload this cache. Retain the original vendor file as the archival source since the cache is a derived artifact, not a substitute.

## Related tools

- **MassCube** (Provides read_raw_file_to_obj for parsing mzML and supported Bruker TDF2 .d data into scan objects, and manages the .mcraw workflow cache.) — https://github.com/huaxuyu/masscube/

## Examples

```
from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj("sample.d"); scan = data.scans[data.ms1_idx_arr[0]]; print(scan.signals)
```

## Evaluation signals

- The returned object has a non-empty .scans list and data.ms1_idx_arr indexes MS1 scans whose .signals columns contain m/z and intensity values.
- For a Bruker .d input, scan.inv_mobility_range reports an aligned observed [min, max] 1/K0 and data.metadata.mobility_pressure_compensated is False.
- After running the untargeted workflow, a .mcraw file exists under the project tmp directory and downstream feature detection reloads it rather than reparsing the mzML/.d source.
- Processing throughput matches the paper's benchmark: 105 GB of Astral MS data decoded on a laptop within 64 min (8–24× faster than MS-DIAL, MZmine3, or XCMS on the same file).
- For development verification, tools/validate_tdf_sdk.py sample.d /path/to/libtimsdata.so produces a JSON report showing exact scan/TOF/intensity coordinates and calibrated m/z matching the official TDF-SDK.

## Limitations

- Only TDF2 (TimsCompressionType=2) Bruker data with DDA-PASEF (MsMsType=8) and specific calibration models is supported; other instrument configurations require extending conventional data objects.
- The reported 1/K0 applies the static mobility calibration but not the vendor's per-frame pressure compensation (data.metadata.mobility_pressure_compensated == False).
- The .mcraw cache is a derived artifact; the original vendor data must be retained as the archival source.
- The decoder depends on the system libzstd runtime via ctypes; on macOS it must be installed (brew install zstd) or located via MASSCUBE_ZSTD_LIBRARY.
- Single-file TXT output does not embed full chromatographic traces by default; Params.output_peak_shape must be set to True when those traces are needed.

## Evidence

- [abstract] MassCube is a Python-based open-source framework for MS data processing: "We present MassCube, a Python-based open-source framework for MS data processing"
- [abstract] MassCube speed on large raw files: "capable for handling 105 GB of Astral MS data on a laptop within 64 min, while other programs took 8–24 times longer."
- [intro] MassCube chose Python for array programming and numerical libraries: "We chose Python for MassCube to leverage the advancements in array programming and to take advantage of Python’s extensive ecosystem of numerical libraries"
- [readme] .mcraw cache reload rationale: "The untargeted workflow decodes each mzML or supported Bruker `.d` source once and stores a versioned `.mcraw` cache under the project `tmp` directory. Feature detection, gap filling, and aligned"
- [readme] Bruker reader decode constraints: "Currently supported Bruker data are TDF2 (`TimsCompressionType=2`), DDA-PASEF (`MsMsType=8`), m/z calibration model 1 with `dC2=0` and `C3=C4=0`, and static TIMS calibration model 2."
- [readme] Retention of archival source: "The original vendor data should still be retained as the archival source."
