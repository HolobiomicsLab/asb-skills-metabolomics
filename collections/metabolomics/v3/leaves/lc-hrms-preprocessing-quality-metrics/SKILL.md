---
name: lc-hrms-preprocessing-quality-metrics
description: Use when you have a validated mzRAPP benchmark (e.g., Benchmark.csv from
  MTBLS267) plus exported output tables from an NPP tool run and need to quantify
  peak-picking or alignment performance such as the proportion of split peaks.
when_to_use_negative:
- You only have a raw feature table with no benchmark dataset — mzRAPP requires a
  user-supplied benchmark of molecules with known retention behavior generated from
  the same mzML files.
- The data are low-resolution or profile-mode — the workflow targets LC-HRMS centroided
  mzML files where isotopologue accuracy/precision (mz precision 6 ppm, mz accuracy
  5 ppm in the example) can be assessed.
- You need direct inter-tool comparison without a shared benchmark — mzRAPP's comparison
  is mediated by matching against a benchmark, not by pairwise joins between two NPP
  outputs alone.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzRAPP
  role: 'Primary tool: imports and validates NPP outputs, joins them to the benchmark
    via match_peaks_to_benchmark/match_features_to_benchmark, and derives performance
    metrics such as split peaks'
  repo: https://github.com/YasinEl/mzRAPP
- name: R
  role: Implementation/runtime environment; mzRAPP is installed via devtools::install_github
    from GitHub and can be used without the shiny interface via R-script
  repo: https://cran.r-project.org/
- name: XCMS
  role: Source of the non-targeted pre-processing output tables (unaligned and aligned
    run 2 CSVs) that are assessed against the benchmark
- name: Skyline
  role: Recommended manual peak-curation tool from which user.rtmin/user.rtmax peak
    boundaries can be exported for the target file used in benchmark generation
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:584e9cff72338f4d30efc17f10e4bbcebf36fbaecb7bf9f35d83756746caaca3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - Skyline
  - XCMS
  - mzRAPP
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lc-hrms-preprocessing-quality-metrics

## Summary

Compute reliability metrics (e.g., split peaks, found/not-found peaks, alignment errors) for non-targeted LC-HRMS pre-processing by matching an mzRAPP benchmark of user-supplied target molecules against NPP tool outputs (XCMS, MZmine 2, MS-DIAL, etc.).

## When to use

Use when you have a validated mzRAPP benchmark (e.g., Benchmark.csv from MTBLS267) plus exported output tables from an NPP tool run and need to quantify peak-picking or alignment performance such as the proportion of split peaks.

## When NOT to use

- You only have a raw feature table with no benchmark dataset — mzRAPP requires a user-supplied benchmark of molecules with known retention behavior generated from the same mzML files.
- The data are low-resolution or profile-mode — the workflow targets LC-HRMS centroided mzML files where isotopologue accuracy/precision (mz precision 6 ppm, mz accuracy 5 ppm in the example) can be assessed.
- You need direct inter-tool comparison without a shared benchmark — mzRAPP's comparison is mediated by matching against a benchmark, not by pairwise joins between two NPP outputs alone.

## Inputs

- Benchmark.csv — mzRAPP benchmark dataset generated from target molecules with known RT boundaries and enviPat-predicted isotopologues extracted from centroided mzML files
- Centroided mzML files (e.g., 30 _POS.mzML files from MTBLS267 on MetaboLights)
- Sample-group file (SampleGroups_MTBLS267.csv) and target file (Target_File_MTBLS267.csv) for benchmark generation
- Unaligned (ungrouped) NPP output table, e.g., XCMS_unaligned_run2.csv
- Aligned (grouped) NPP output table, e.g., XCMS_aligned_run2.csv

## Outputs

- Peak Picking performance metrics, including the proportion of matches classified as split peaks
- Additional NPP performance metrics for found/not-found peaks, missing peaks/values, peak abundance quality/degenerated IR, and alignment error counting
- Written metrics table, e.g., xcms_run2_assessment.csv

## How to apply

In R, load mzRAPP and read the prepared Benchmark.csv benchmark downloaded from the ucloud deposit. Import the NPP tool's unaligned output with import_ungrouped_xcms (for XCMS run 2: XCMS_unaligned_run2.csv), the aligned output with import_grouped_xcms (XCMS_aligned_run2.csv), and generate_options for XCMS; validate both inputs with check_nonTargeted_input. Match benchmark peaks to NPP peaks via match_peaks_to_benchmark and benchmark features to NPP features via match_features_to_benchmark — mzRAPP performs non-equivalent joins that supply the matching step needed for peak-picking metrics. Then run derive_performance_metrics and extract the Peak Picking box metrics, in particular the proportion of matches classified as split peaks. Benchmarks are built from target molecules with known molecular composition (SumForm_c) and retention-time boundaries, from which enviPat-predicted isotopologues are extracted from centroided mzML files; an NPP run is judged reliable when the bootstrapped split-peaks percentage falls within the 1–3% range stated in the article.

## Related tools

- **mzRAPP** (Primary tool: imports and validates NPP outputs, joins them to the benchmark via match_peaks_to_benchmark/match_features_to_benchmark, and derives performance metrics such as split peaks) — https://github.com/YasinEl/mzRAPP
- **R** (Implementation/runtime environment; mzRAPP is installed via devtools::install_github from GitHub and can be used without the shiny interface via R-script) — https://cran.r-project.org/
- **XCMS** (Source of the non-targeted pre-processing output tables (unaligned and aligned run 2 CSVs) that are assessed against the benchmark)
- **Skyline** (Recommended manual peak-curation tool from which user.rtmin/user.rtmax peak boundaries can be exported for the target file used in benchmark generation) — https://skyline.ms/project/home/software/Skyline/begin.view

## Examples

```
library(mzRAPP); bmk <- read.csv("Benchmark.csv"); xc_un <- import_ungrouped_xcms("XCMS_unaligned_run2.csv"); xc_al <- import_grouped_xcms("XCMS_aligned_run2.csv"); check_nonTargeted_input(bmk, xc_un, xc_al); m <- match_peaks_to_benchmark(bmk, xc_un, xc_al, generate_options("XCMS")); derive_performance_metrics(m)
```

## Evaluation signals

- The reported split-peaks percentage from bootstrapped confidence intervals falls within the 1–3% range stated in the article.
- check_nonTargeted_input completes on both the imported unaligned and aligned NPP tables without errors.
- match_peaks_to_benchmark and match_features_to_benchmark produce matched peak/feature tables with non-equivalent joins populated for both XCMS run 2 outputs.
- derive_performance_metrics returns a Peak Picking box containing the split-peaks proportion alongside found/not-found and alignment error metrics.
- All computed metrics are written to the output file (e.g., xcms_run2_assessment.csv) with a defined value for the split-peaks metric.

## Limitations

- Only isotopologues whose theoretically most abundant plus at least one additional isotopologue are detected make it into the benchmark; peaks with area/height more than 30% off the predicted value or Pearson correlation < 0.85 versus the highest isotopologue are removed, so coverage depends on target-molecule quality.
- If the main_adduct of a molecule is not detected, other adducts will not be accepted, reducing benchmark coverage for some compounds.
- mzRAPP is designed for LC-HRMS reliability assessment and supports five of the most commonly used pre-processing tools; outputs from unsupported tools require adapter/import functions.

## Evidence

- [abstract] which generates and validates a benchmark from user-supplied information and later utilizes it for reliability assessment of data pre-processing: "which generates and validates a benchmark from user-supplied information and later utilizes it for reliability assessment of data pre-processing"
- [abstract] mzRAPP produces several performance metrics for different steps of the pre-processing workflow: "mzRAPP produces several performance metrics for different steps of the pre-processing workflow"
- [readme] The resulting benchmark dataset is used to extract different performance metrics for NPP performed on the same mzML files.: [evidence span withheld — anchor status: external_doc]
- [methods] Download the already prepared benchmark "Benchmark.csv" if you do not want to do the benchmark generation part of the example: "Download the already prepared benchmark "Benchmark.csv" if you do not want to do the benchmark generation part of the example"
- [readme] Isotopologue peaks with an area or height which is more than 30% off the predicted value or a Pearson Correlation coef < 0.85 (as compared to the highest isotopologue) are removed.: [evidence span withheld — anchor status: not_found]
