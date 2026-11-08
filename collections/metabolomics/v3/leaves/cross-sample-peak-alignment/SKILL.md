---
name: cross-sample-peak-alignment
description: Use when you have per-sample peak lists (e.g. m/z values from SIM-stitched
  DIMS or XCMS feature tables) and need one aligned peak table across all biological
  and blank samples before filtering or statistics.
when_to_use_negative:
- Input is already an aligned peak/feature table — re-aligning would duplicate features
  or split true matches.
- You are still at the raw-transient stage (.dat/Thermo .RAW files) — alignment requires
  per-sample peak lists produced after Mass Calibration and SIM-stitching.
- You need to combine peaks within a single sample across SIM windows — that is SIM-stitching,
  not cross-sample alignment.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: Galaxy-M
  role: Workflow platform hosting the Align Samples tool that matches DIMS peaks across
    all samples
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: MI-Pack
  role: Provides the underlying DIMS processing functions (SIM-stitch peak lists)
    consumed by the alignment step
  repo: https://github.com/Viant-Metabolomics/MI-Pack
- name: XCMS
  role: Performs the analogous feature detection, retention time correction and alignment
    of spectral features for the LC-MS workflow
- name: Galaxy
  role: Platform in which the alignment workflow is executed and exported as a reproducible
    history
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:b003e35378184b3653e76dfefc74dd225f871ded7948bb991564eef5e270effd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - MI-Pack
  - XCMS
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cross-sample-peak-alignment

## Summary

Align peak lists (m/z values) across all samples so that identical metabolite features detected independently in each sample are matched into a single row of an aligned peak table, ready for data cleansing and statistics. In the Galaxy-M DIMS workflow this is performed by the Align Samples tool on the SIM-stitched peak lists.

## When to use

Use when you have per-sample peak lists (e.g. m/z values from SIM-stitched DIMS or XCMS feature tables) and need one aligned peak table across all biological and blank samples before filtering or statistics.

## When NOT to use

- Input is already an aligned peak/feature table — re-aligning would duplicate features or split true matches.
- You are still at the raw-transient stage (.dat/Thermo .RAW files) — alignment requires per-sample peak lists produced after Mass Calibration and SIM-stitching.
- You need to combine peaks within a single sample across SIM windows — that is SIM-stitching, not cross-sample alignment.

## Inputs

- Per-sample SIM-stitched DIMS peak lists (m/z values, post mass calibration and SIM-stitching)
- Optionally, replicate-filtered peak lists (peaks retained in x-out-of-n technical replicates)
- LC-MS centroided/feature data processed by XCMS for the LC-MS analogue

## Outputs

- Aligned peak table (CSV) with peaks matched across all samples
- Feature matrix suitable for blank/sample filtering, missing value imputation and PCA

## How to apply

In the Galaxy-M DIMS chain, first produce a per-sample peak list by running Sum Transients (average transients per SIM window), Process Transients (Hanning apodization, zero-filling, Fourier transform, baseline correction), and Mass Calibration and SIM-stitching (peak picking at ~3.5:1 signal-to-noise, frequency-to-m/z calibration per SIM window, stitching windows). Optionally apply Replicate Filtering so only peaks present in at least x-out-of-n technical replicates survive. Then run the Align Samples tool to match peaks by m/z across all samples and export the aligned peak table as a CSV file. For LC-MS data the analogous step is handled inside the XCMS-Basic-Birmingham-Pipe, where XCMS performs feature detection, retention time correction and grouping/alignment of spectral features across samples.

## Related tools

- **Galaxy-M** (Workflow platform hosting the Align Samples tool that matches DIMS peaks across all samples) — https://github.com/Viant-Metabolomics/Galaxy-M
- **MI-Pack** (Provides the underlying DIMS processing functions (SIM-stitch peak lists) consumed by the alignment step) — https://github.com/Viant-Metabolomics/MI-Pack
- **XCMS** (Performs the analogous feature detection, retention time correction and alignment of spectral features for the LC-MS workflow)
- **Galaxy** (Platform in which the alignment workflow is executed and exported as a reproducible history) — https://github.com/galaxyproject/galaxy

## Evaluation signals

- The aligned output is a CSV peak table with one row per unique m/z feature and one column per sample.
- Peaks that passed Replicate Filtering (present in at least x-out-of-n technical replicates) should not disappear during alignment.
- m/z values matched across samples should fall within the high mass accuracy expected of the SIM-stitch method (very high mass accuracy reported for the DIMS workflow).
- Downstream Blank Filtering and Sample Filtering (x-out-of-n biological samples) should operate on the aligned table without missing-column errors.
- Aligned table row count should be comparable between replicate runs of the workflow on the same DIMS test data (MetaboLights MTBLS79 subset).

## Limitations

- The DIMS transient (.dat) files needed upstream of alignment were not included in the MetaboLights accession and had to be obtained from the original authors.
- The test data (MetaboLights MTBLS79 for DIMS) are a subset of larger datasets, so alignment behaviour on full-scale data may differ.
- Setting up the SimStitch environment (32-bit WINE, WINEARCH=win32, Thermo MSFileReader 3.0 sp2) is fragile; wrong WINE mode yields 'Invalid file identifier' errors before any peak list exists to align.

## Evidence

- [abstract] Align Samples: aligns peaks across all samples: "Align Samples: aligns peaks across all samples"
- [abstract] SIM-stitching picks peaks and calibrates windows: "picks peaks with a certain signal-to-noise-ratio threshold (e.g. 3.5:1), calibrates each SIM window to convert the frequency domain to m/z measurements and stitches all the SIM windows together"
- [abstract] Replicate filtering precedes downstream steps: "Replicate Filtering: filters peaks that fail to appear in at least x-out-of-n technical replicates"
- [abstract] XCMS aligns LC-MS features across samples: "groups/aligns spectral features across the samples using the R-package XCMS [23]"
- [abstract] Transient files missing from MetaboLights accession: "These transient files were obtained from the original authors of the data as they are not included in the MetaboLights accession"
