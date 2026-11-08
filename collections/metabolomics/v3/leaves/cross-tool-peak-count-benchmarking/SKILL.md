---
name: cross-tool-peak-count-benchmarking
description: Use when a novel peak-picking pipeline for untargeted LC/HRMS data needs
  quantitative benchmarking against MZmine 2, xcms-CAMERA, or MS-DIAL on a shared
  raw test file.
when_to_use_negative:
- When the input is already a processed/aligned peak table rather than raw LC/HRMS
  acquisitions — re-running peak detection tools on peak tables will not yield comparable
  raw counts.
- When the goal is quantifying alignment/annotation quality across tools rather than
  raw peak-detection sensitivity — this skill measures only peak counts and threshold
  failures, not retention-time alignment or annotation accuracy.
- When baseline tools require vendor-specific file formats not available for the study
  (e.g. MZmine 2 or MS-DIAL may not parse Agilent .d directly) — the file must be
  convertible/parseable by all four tools before benchmarking.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: IDSL.IPA
  role: Candidate pipeline under benchmark — its `IPA_workflow` function (driven by
    a parameter spreadsheet) performs 12C/13C isotopologue pairing, mass correction,
    peak smoothing, local-noise baseline development, and peak quality determination
    on the raw test file to produce the peak list being benchmarked.
  repo: https://github.com/idslme/IDSL.IPA
- name: MZmine 2
  role: Baseline peak-detection tool run on the same file to provide a reference peak
    list for fold-ratio computation.
- name: xcms-CAMERA
  role: Baseline R pipeline (chromatographic region-of-interest detection plus CAMERA
    grouping) run on the same file to provide a reference peak list for fold-ratio
    computation.
- name: MS-DIAL
  role: Baseline untargeted peak-detection tool run on the same file to provide a
    reference peak list for fold-ratio computation.
- name: R
  role: Execution environment for the candidate pipeline (IDSL.IPA) and the xcms-CAMERA
    baseline, and for aggregating the comparison table.
  repo: https://cran.r-project.org/package=IDSL.IPA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:7219e131841c39624b9ab6828ec0e88a197468935063a100e936030311b2a29a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - MS-DIAL
  - MZmine
  - MZmine 2
  - R
  - enviPat
  - xcms-CAMERA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# cross-tool-peak-count-benchmarking

## Summary

Benchmark a new peak-detection pipeline against established untargeted LC/HRMS tools (MZmine 2, xcms-CAMERA, MS-DIAL) by running all four on the same raw file and comparing total peak counts, fold ratios, and threshold-failure rates.

## When to use

Use when a novel peak-picking pipeline for untargeted LC/HRMS data needs quantitative benchmarking against MZmine 2, xcms-CAMERA, or MS-DIAL on a shared raw test file.

## When NOT to use

- When the input is already a processed/aligned peak table rather than raw LC/HRMS acquisitions — re-running peak detection tools on peak tables will not yield comparable raw counts.
- When the goal is quantifying alignment/annotation quality across tools rather than raw peak-detection sensitivity — this skill measures only peak counts and threshold failures, not retention-time alignment or annotation accuracy.
- When baseline tools require vendor-specific file formats not available for the study (e.g. MZmine 2 or MS-DIAL may not parse Agilent .d directly) — the file must be convertible/parseable by all four tools before benchmarking.

## Inputs

- A single raw untargeted LC/HRMS test file (e.g. 003.d from the MTBLS1684 study, an Agilent .d acquisition exported for processing)
- Installations of the candidate pipeline (IDSL.IPA R package) and baseline tools (MZmine 2, xcms-CAMERA via R, MS-DIAL)
- The candidate pipeline's parameter spreadsheet (IPA parameters sheet with isotopologue-pairing, smoothing, baseline and peak-quality settings)

## Outputs

- Total peak count per tool (e.g. 20,253 peaks for IDSL.IPA on 003.d)
- Fold-ratio table comparing candidate peak count to each baseline (1.6x MZmine 2, 5.4x xcms-CAMERA, 2.2x MS-DIAL)
- Rate at which baseline-only peaks fail the candidate's isotopologue-pairing and peak-quality thresholds
- Aggregated comparison table, e.g. peak_count_comparison.csv

## How to apply

Run each tool independently on the identical raw LC/HRMS file (here, the 003.d test file from the MTBLS1684 study) to obtain its peak list. For the candidate pipeline — IDSL.IPA, invoked via `IPA_workflow` with its parameter spreadsheet covering 12C/13C isotopologue pairing, mass correction, peak smoothing, local-noise baseline development, and peak quality determination — record the total peak count (reported: 20,253 peaks). Baseline tools run under their standard peak-detection settings produce their own peak lists; from these compute fold ratios of the candidate's count against each baseline (article values: 1.6x vs MZmine 2, 5.4x vs xcms-CAMERA, 2.2x vs MS-DIAL). To assess why peaks detected by baseline tools but absent from the candidate list were missed, evaluate them against the candidate's isotopologue-pairing and peak-quality thresholds and record the failure rate. Aggregate counts, fold ratios, and threshold-failure rates into a comparison table (e.g. peak_count_comparison.csv).

## Related tools

- **IDSL.IPA** (Candidate pipeline under benchmark — its `IPA_workflow` function (driven by a parameter spreadsheet) performs 12C/13C isotopologue pairing, mass correction, peak smoothing, local-noise baseline development, and peak quality determination on the raw test file to produce the peak list being benchmarked.) — https://github.com/idslme/IDSL.IPA
- **MZmine 2** (Baseline peak-detection tool run on the same file to provide a reference peak list for fold-ratio computation.)
- **xcms-CAMERA** (Baseline R pipeline (chromatographic region-of-interest detection plus CAMERA grouping) run on the same file to provide a reference peak list for fold-ratio computation.)
- **MS-DIAL** (Baseline untargeted peak-detection tool run on the same file to provide a reference peak list for fold-ratio computation.)
- **R** (Execution environment for the candidate pipeline (IDSL.IPA) and the xcms-CAMERA baseline, and for aggregating the comparison table.) — https://cran.r-project.org/package=IDSL.IPA

## Examples

```
library(IDSL.IPA); IPA_workflow("IPA_parameters.xlsx")  # then run MZmine 2, xcms-CAMERA, MS-DIAL on the same 003.d file and tabulate fold ratios
```

## Evaluation signals

- Total peak counts are produced for all four tools on the identical file, with the candidate's count (e.g. 20,253) exactly matching the reported benchmark value for that file.
- Fold ratios are computed as candidate_count / baseline_count and reproduce the published ratios (1.6x vs MZmine 2, 5.4x vs xcms-CAMERA, 2.2x vs MS-DIAL) within rounding.
- Each baseline-only peak is classified as either passing or failing the candidate's isotopologue-pairing and peak-quality thresholds; the failure rate is recorded rather than unexplained.
- The comparison table (e.g. peak_count_comparison.csv) contains a row per tool with total count, plus fold-ratio and threshold-failure columns.

## Limitations

- Peak counts alone do not indicate correctness; baseline-only peaks may fail the candidate pipeline's isotopologue-pairing or peak-quality thresholds rather than being truly spurious, so the failure rate must be interpreted alongside raw counts.
- Benchmark applies to a single test file (003.d); results are not necessarily generalizable to other chromatographic methods or instruments.
- The candidate pipeline's data matrices still need downstream processing (adduct/fragment identification, statistical filtering, molecular formula annotation) — this skill covers only the peak-detection comparison stage.

## Evidence

- [results] Baseline tools named as comparators: [evidence span withheld — anchor status: not_found]
