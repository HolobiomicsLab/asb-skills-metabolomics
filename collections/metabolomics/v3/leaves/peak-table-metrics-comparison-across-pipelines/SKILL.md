---
name: peak-table-metrics-comparison-across-pipelines
description: Use when benchmarking peak-picking/alignment pipelines on identical mzML/mzXML
  cohorts and you need defensible sensitivity, specificity, speed, and overlap numbers,
  such as IPA vs MS-DIAL on 499-sample studies.
when_to_use_negative:
- When only one pipeline's aligned feature table is already available and no raw data
  exists to rerun a second tool — comparison requires processing the same raw files
  through both pipelines.
- When the goal is formula annotation or MS2 deconvolution rather than peak-picking
  benchmarking — those tasks belong to IDSL.UFA, IDSL.UFAx, or IDSL.CSA, not this
  skill.
- When the two pipelines differ in upstream peak-detection logic (e.g. MZmine 2/xcms-CAMERA
  chromatographic ROI methods) and no shared-peak overlap metric is planned — raw
  count deltas alone do not distinguish m/z-binning differences from true detection
  differences.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: 'Primary peak-picking pipeline being benchmarked: detects 12C/13C isotopologue
    pairs, builds EICs, characterizes peaks, corrects RT, and aligns across m/z slices
    in parallel.'
  repo: https://github.com/idslme/IDSL.IPA
- name: MS-DIAL
  role: Reference pipeline whose aligned peak table provides the shared-peak coverage
    baseline (82%, 41,172 shared peaks) for cross-tool comparison.
- name: MZmine 2
  role: Alternate benchmark pipeline cited for single-file and cohort-level peak detection
    comparison against IDSL.IPA.
- name: xcms-CAMERA
  role: Alternate benchmark pipeline whose chromatographic ROI approach is compared
    to IDSL.IPA's isotopologue-pairing strategy.
- name: R
  role: 'Implementation environment: IDSL.IPA is an R CRAN package; the IPA_workflow
    function is invoked from R/Rstudio with a parameter spreadsheet.'
  repo: https://cran.r-project.org/package=IDSL.IPA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:9eb025d2b40e07add37e705a317d6b3ecf0f4bf7cbe4dd0f2ba4e07b1c19c0c3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - MS-DIAL
  - MZmine 2
  - R
  - xcms-CAMERA
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# peak-table-metrics-comparison-across-pipelines

## Summary

Run two LC/HRMS peak-picking pipelines (e.g. IDSL.IPA and MS-DIAL) on the same raw cohort, then quantify cross-tool agreement by the number of aligned peaks, processing time, and shared-peak coverage.

## When to use

Use when benchmarking peak-picking/alignment pipelines on identical mzML/mzXML cohorts and you need defensible sensitivity, specificity, speed, and overlap numbers, such as IPA vs MS-DIAL on 499-sample studies.

## When NOT to use

- When only one pipeline's aligned feature table is already available and no raw data exists to rerun a second tool — comparison requires processing the same raw files through both pipelines.
- When the goal is formula annotation or MS2 deconvolution rather than peak-picking benchmarking — those tasks belong to IDSL.UFA, IDSL.UFAx, or IDSL.CSA, not this skill.
- When the two pipelines differ in upstream peak-detection logic (e.g. MZmine 2/xcms-CAMERA chromatographic ROI methods) and no shared-peak overlap metric is planned — raw count deltas alone do not distinguish m/z-binning differences from true detection differences.

## Inputs

- Raw LC/HRMS data files (mzML/mzXML/netCDF) for the full study cohort from both pipelines (e.g. 499 cord-blood samples from MTBLS1684)
- Per-pipeline parameter spreadsheets or configuration (input/output paths, number of processing threads = 20)
- Optional reference per-file peak counts for sanity checking (e.g. 20,253 peaks for test file 003.d)

## Outputs

- Aligned peak intensity matrix per pipeline (e.g. aligned_peak_table.csv)
- Cross-pipeline comparison metrics: total aligned peak count, processing time, and shared-peak overlap (e.g. 82%, 41,172 peaks)
- Per-file intermediate peak counts for QA (e.g. peaklists directory)

## How to apply

Process the identical raw files through each pipeline with parameters matched to the study scale — for IDSL.IPA, fill the parameter spreadsheet (input/output locations, thread count = 20) and run the IPA_workflow function through its six sections, which pair 12C/13C isotopologues per scan, build EICs, apply mass correction/peak smoothing/local-noise baseline/peak-quality checks, correct RT via dynamic retention index markers, and align in parallel across m/z slices. Export each tool's aligned peak intensity matrix (e.g. aligned_peak_table.csv). Then compare at three levels: total aligned peak count (214,000 peaks for 499 cord-blood samples in MTBLS1684), wall-clock processing time (~1 hour with 20 threads), and shared-peak coverage against the reference pipeline (82% = 41,172 shared peaks with MS-DIAL). The rationale is that raw peak counts alone overstate tool differences, so a union/intersection peak overlap metric is needed to attribute disagreement to genuine sensitivity/specificity gaps rather than m/z binning or RT-shift artifacts. Per-tool single-file counts (e.g. 20,253 total peaks for MTBLS1684 file 003.d) can serve as a fast sanity check before committing to the full cohort run.

## Related tools

- **IDSL.IPA** (Primary peak-picking pipeline being benchmarked: detects 12C/13C isotopologue pairs, builds EICs, characterizes peaks, corrects RT, and aligns across m/z slices in parallel.) — https://github.com/idslme/IDSL.IPA
- **MS-DIAL** (Reference pipeline whose aligned peak table provides the shared-peak coverage baseline (82%, 41,172 shared peaks) for cross-tool comparison.)
- **MZmine 2** (Alternate benchmark pipeline cited for single-file and cohort-level peak detection comparison against IDSL.IPA.)
- **xcms-CAMERA** (Alternate benchmark pipeline whose chromatographic ROI approach is compared to IDSL.IPA's isotopologue-pairing strategy.)
- **R** (Implementation environment: IDSL.IPA is an R CRAN package; the IPA_workflow function is invoked from R/Rstudio with a parameter spreadsheet.) — https://cran.r-project.org/package=IDSL.IPA

## Examples

```
library(IDSL.IPA); IPA_workflow('path/to/IPA_parameters.xlsx')
```

## Evaluation signals

- Sanity-check per-file detection: a single test LC/HRMS file from the cohort should yield approximately the reported count (e.g. 20,253 total peaks for MTBLS1684 file 003.d).
- Cohort-scale invariant: the aligned peak table row count should land in the reported magnitude (e.g. ~214,000 aligned peaks for 499 samples), not orders of magnitude smaller or larger.
- Cross-tool overlap: shared-peak coverage against the reference pipeline should approach the reported value (82%, i.e. 41,172 peaks); a large shortfall suggests misconfigured thread counts, RT correction, or m/z binning.
- Throughput check: the full cohort run should complete in the reported wall-clock window (≈1 hour with 20 parallel threads); substantial slowdowns indicate thread or I/O misconfiguration.
- Schema check: the exported table must be a valid CSV containing an aligned peak intensity matrix indexed by m/z-RT, with per-sample columns matching the study's sample count.

## Limitations

- IDSL.IPA aligned data matrices still require downstream processing (adduct/fragment identification, statistical filtering, molecular formula annotation) before they are analysis-ready — comparison metrics alone do not produce usable biology.
- A majority of manually rejected EICs were low-abundance peaks that passed the S/N ratio threshold; additional filtering rules are still needed to auto-remove these, so raw peak counts can overstate tool sensitivity.
- The RT correction and peak quality thresholds differ per pipeline; without careful parameter matching, overlap metrics conflate algorithmic differences with tuning differences.

## Evidence

- [results] IPA generated 214,000 aligned peaks in 1 hour: [evidence span withheld — anchor status: not_found]
- [results] 82% shared-peak coverage with MS-DIAL: [evidence span withheld — anchor status: not_found]
- [results] Rejection of low-abundance peaks: [evidence span withheld — anchor status: not_found]
- [results] Benchmark comparison tools: [evidence span withheld — anchor status: not_found]
