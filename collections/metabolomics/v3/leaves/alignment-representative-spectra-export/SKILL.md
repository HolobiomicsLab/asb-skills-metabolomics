---
name: alignment-representative-spectra-export
description: Use when you must reproduce MS-DIAL's aligned LC-MS output (text export
  and .dcl representative spectra) from source without building a GUI project file
  or EIC cache, e.g. for large file-count cohorts where the full alignment mode is
  too memory-heavy.
when_to_use_negative:
- 'When you need isotope/adduct or ion-abundance correlation links between aligned
  peaks — light mode with ''Alignment light correlation links: False'' discards them
  and collapses each spot to one representative peak.'
- When your input is already a feature/alignment table and you only need downstream
  statistics, not re-running MS-DIAL alignment.
- When you rely on the 'Debug/Release vendor unsupported' build to read proprietary
  vendor raw formats — per the README it only supports abf, cdf, and mzml.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MS-DIAL 5 (MsdialWorkbench)
  role: Source-built LC-MS Console (MSDIALCUI.exe) whose light-mode alignment runner
    streams peak matching, gap filling, and representative MSDec export to the .dcl
    and alignment text.
  repo: https://github.com/systemsomicslab/MsdialWorkbench
- name: rawdataviewer.exe
  role: Companion application bundled with MS-DIAL packages for inspecting raw spectra
    during verification of exported results.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-54137-w
    title: MS-DIAL 5 multimodal mass spectrometry data mining unveils lipidome complexities
  source_content_hash: sha256:67f801f4922dd81f9607ef6dafc96b56dac47e78e516520eb5ef38fd5c832ece
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systemsomicslab/MsdialWorkbench
  tools:
  - MS-DIAL 5
  - MS-DIAL 5 (MsdialWorkbench)
  - rawdataviewer.exe
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 21
---

# alignment-representative-spectra-export

## Summary

Exports MS-DIAL alignment results in a headless, memory-bounded 'light mode' streaming path: each aligned spot's AlignedPeakProperties is reduced to a single representative peak and MSDec results are written to the alignment .dcl via AlignmentLightQuantValueAccessor-backed exporters, byte-identical to the full-mode alignment export.

## When to use

Use when you must reproduce MS-DIAL's aligned LC-MS output (text export and .dcl representative spectra) from source without building a GUI project file or EIC cache, e.g. for large file-count cohorts where the full alignment mode is too memory-heavy.

## When NOT to use

- When you need isotope/adduct or ion-abundance correlation links between aligned peaks — light mode with 'Alignment light correlation links: False' discards them and collapses each spot to one representative peak.
- When your input is already a feature/alignment table and you only need downstream statistics, not re-running MS-DIAL alignment.
- When you rely on the 'Debug/Release vendor unsupported' build to read proprietary vendor raw formats — per the README it only supports abf, cdf, and mzml.

## Inputs

- Vendor raw LC-MS/MS files (e.g. SCIEX WIFF FastLC demo files) listed in an analysis_files.csv
- Console method file (method.txt) with 'Alignment light mode: True' and 'Alignment light correlation links: False'
- MsdialWorkbench source (LcmsAlignmentLightRunner, AlignmentLightPeakStore, AlignmentLightQuantValueAccessor)

## Outputs

- Alignment text export (mdalign-format) with quant values for retained spots
- Alignment .dcl file containing representative (single-peak) MSDec results per spot
- No GUI project file and no EIC cache (expected absence is part of the contract)

## How to apply

Build the MsdialWorkbench LC-MS Console (MSDIALCUI.exe, Release/net48) with 'Alignment light mode: True' set in the console method file, then run the LcmsAlignmentLightRunner streaming path on the input files, building a master peak list and writing matched rows to the fixed-size spot-by-file AlignmentLightPeakStore binary matrix. Run gap filling file-by-file over retained spots, writing gap-filled rows directly to the compact store while keeping spot-level aggregates (PeakCountFilter, QcFilter, DetectedNumberFilter, BlankFilter) instead of per-spot detected-file sets. Keep 'Alignment light correlation links: False' so each spot's AlignedPeakProperties reduces to one representative peak, and write representative alignment MSDec results to the alignment .dcl from a streaming enumerable guarded by a bounded LRU cache. Finally export the alignment text through the AlignmentLightQuantValueAccessor-backed exporters. The rationale is that the light path avoids materializing per-spot detected-file sets and EIC caches, which is what makes headless streaming feasible for multi-file runs.

## Related tools

- **MS-DIAL 5 (MsdialWorkbench)** (Source-built LC-MS Console (MSDIALCUI.exe) whose light-mode alignment runner streams peak matching, gap filling, and representative MSDec export to the .dcl and alignment text.) — https://github.com/systemsomicslab/MsdialWorkbench
- **rawdataviewer.exe** (Companion application bundled with MS-DIAL packages for inspecting raw spectra during verification of exported results.)

## Examples

```
MSDIALCUI.exe lcms -i <demo>/analysis_files.csv -o <out> -m <demo>/method.txt
```

## Evaluation signals

- The retained spot count matches the full-mode baseline (908 LC-MS alignment spots on the seven SCIEX WIFF FastLC demo files).
- The light-mode mdalign export is byte-for-byte identical to the task_001 full-mode baseline, verified by identical SHA256 hash.
- No GUI project file and no EIC cache are produced in the output directory.
- Each exported spot carries exactly one representative peak (AlignedPeakProperties reduced to a single representative peak, correlation links off).
- The four spot-level filters (PeakCountFilter, QcFilter, DetectedNumberFilter, BlankFilter) apply correctly from aggregates retained in the compact AlignmentLightPeakStore.

## Limitations

- 'Alignment light correlation links: False' means ion-abundance correlation links between peaks are not exported; only representative single-peak rows survive.
- The 'Debug/Release vendor unsupported' configuration cannot read proprietary manufacturer formats; only abf (Reifycs), cdf (NetCDF), and mzml are supported in that configuration.
- A bounded LRU cache during streaming means spot spectra are evicted and re-fetched; memory is bounded but very large cohorts depend on streaming order.
- The GUI of MS-DIAL is restricted to Windows; headless/light-mode use must go through the console entry point.

## Evidence

- [methods] Alignment light mode: [evidence span withheld — anchor status: not_found]
- [methods] Alignment light correlation links: "`Alignment light correlation links: False`"
- [methods] PeakCountFilter: "`PeakCountFilter`: count of detected peaks"
- [results] MsdialWorkbench repository: "The source code is publicly accessible from the GitHub repository (https://github.com/systemsomicslab/MsdialWorkbench)"
- [readme] Vendor format support: [evidence span withheld — anchor status: external_doc]
- [readme] Supported open formats: [evidence span withheld — anchor status: external_doc]
