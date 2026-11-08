---
name: npp-performance-metric-derivation
description: Use when unaligned and aligned NPP output tables (e.g. XCMS_unaligned_run*.csv,
  MZmine_aligned.csv) exist for LC-HRMS mzML files and you need to quantify peak-picking
  and alignment reliability against a validated benchmark.
when_to_use_negative:
- Input is already a reliability assessment or metric table rather than raw unaligned/aligned
  NPP peak output from a supported tool (XCMS, MZmine 2, MS-DIAL, etc.)
- No benchmark dataset exists for the same mzML files and only benchmark generation
  (not assessment) is the goal
- Data are profile-mode vendor files not converted to centroided mzML — mzRAPP requires
  centroided mzML input
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzRAPP
  role: primary R tool that matches the benchmark to NPP output and derives performance
    metrics with bootstrapped CIs
  repo: https://github.com/YasinEl/mzRAPP
- name: XCMS
  role: source of the unaligned/aligned NPP peak tables being assessed (e.g. run 1
    and run 3 outputs)
- name: MZmine 2
  role: alternative NPP tool whose 30 unaligned csv files plus MZmine_aligned.csv
    are matched against the benchmark
- name: R
  role: implementation language; mzRAPP is installed via devtools::install_github
    and run with library(mzRAPP)
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:60ad52f873f286ee7681934be8c7b18ea638f511313db80e93a3332d2751f8fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - MZmine 2
  - R
  - XCMS
  - mzRAPP
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# npp-performance-metric-derivation

## Summary

Derive reliability metrics for non-targeted pre-processing (NPP) outputs (XCMS, MZmine 2, MS-DIAL) by matching their unaligned/aligned peak tables against an mzRAPP-generated benchmark (e.g. from MTBLS267), quantifying missed peaks, split peaks, degenerated isotopologue ratios, and alignment errors with bootstrapped confidence intervals.

## When to use

Use when unaligned and aligned NPP output tables (e.g. XCMS_unaligned_run*.csv, MZmine_aligned.csv) exist for LC-HRMS mzML files and you need to quantify peak-picking and alignment reliability against a validated benchmark.

## When NOT to use

- Input is already a reliability assessment or metric table rather than raw unaligned/aligned NPP peak output from a supported tool (XCMS, MZmine 2, MS-DIAL, etc.)
- No benchmark dataset exists for the same mzML files and only benchmark generation (not assessment) is the goal
- Data are profile-mode vendor files not converted to centroided mzML — mzRAPP requires centroided mzML input

## Inputs

- Benchmark.csv (mzRAPP-generated benchmark table with isotopologue peaks)
- unaligned NPP output tables (e.g. XCMS_unaligned_run1.csv, 30 MZmine 2 unaligned csv files)
- aligned NPP output tables (e.g. XCMS_aligned_run1.csv, MZmine_aligned.csv)
- centroided mzML files underlying the NPP run (e.g. MTBLS267 *_POS.mzML)

## Outputs

- performance metrics table (e.g. xcms_run1_npp_assessment.csv, mzmine2_npp_assessment_metrics.csv)
- post-alignment detection rate with bootstrapped 95% CI
- proportion of degenerated isotopologue ratios
- alignment error/peaks-lost counts and rate
- split-peak proportion and missing-peak classification
- sunburst plots of found/not-found peaks and alignment

## How to apply

Load the benchmark (Benchmark.csv) plus the NPP tool's unaligned and aligned output tables into mzRAPP (via callmzRAPP() GUI or R functions), first validating them with check_nonTargeted_input() and importing with tool-specific importers (import_ungrouped_xcms()/import_grouped_xcms(), import_ungrouped_mzmine/import_grouped_mzmine). Match benchmark peaks to NPP peaks with match_peaks_to_benchmark() (and match_peaks_to_benchmark_split() for split peaks), then match NPP features and run assess_alignment()/count_alignment_errors() to count peaks lost during alignment. Quantify isotopologue-ratio degeneration with check_IR_biases() and classify missed peaks with check_missing_peaks()/classify_false_negative(), then summarize per-stage metrics with derive_performance_metrics() using alpha = 0.95 bootstrapped confidence intervals. Optionally produce diagnostic sunburst plots (plot_sunburst_peaks, plot_sunburst_alignment) and the peak-abundance-quality line plot, and export the assembled metrics table to csv.

## Related tools

- **mzRAPP** (primary R tool that matches the benchmark to NPP output and derives performance metrics with bootstrapped CIs) — https://github.com/YasinEl/mzRAPP
- **XCMS** (source of the unaligned/aligned NPP peak tables being assessed (e.g. run 1 and run 3 outputs))
- **MZmine 2** (alternative NPP tool whose 30 unaligned csv files plus MZmine_aligned.csv are matched against the benchmark)
- **R** (implementation language; mzRAPP is installed via devtools::install_github and run with library(mzRAPP))

## Examples

```
library(mzRAPP); bm <- import_benchmark('Benchmark.csv'); pp <- import_ungrouped_xcms('XCMS_unaligned_run1.csv'); pf <- import_grouped_xcms('XCMS_aligned_run1.csv', algo='XCMS'); derive_performance_metrics(match_peaks_to_benchmark(bm, pp), pf)
```

## Evaluation signals

- XCMS run 1 metrics reproduce reported values: 2660/2870 benchmark peaks found, up to 12% split peaks, 28-53% degenerated IR, 19-35% peaks lost in alignment, 83-94% post-alignment detection (bootstrap randomness tolerated)
- XCMS run 3 metrics fall in reported ranges: 93-99% post-alignment detection, 3-20% degenerated IR, 4-10% alignment-lost peaks
- MZmine 2 run falls in reported ranges: 82-92% post-alignment detection, 1-9% degenerated IR, ~7% alignment errors
- check_nonTargeted_input() passes validation on all supplied NPP tables before matching
- Output metrics csv is assembled with post-alignment detection, degenerated IR proportion, alignment error rate, and missing-value classification columns

## Limitations

- Bootstrapped confidence intervals introduce an element of randomness, so exact metric values vary slightly between runs
- Assessment quality depends on benchmark size — the more molecules with known retention behavior supplied by the user, the better
- Only isotopologues with at least two detected isotopologues and predicted abundances within 30% (Pearson correlation >= 0.85 vs highest isotopologue) enter the benchmark, limiting coverage
- Isotopologue peaks failing peak-shape/abundance criteria are removed, so low-abundance peaks may be excluded from assessment

## Evidence

- [other] By matching the BM against supplied NPP output tables (e.g., XCMS unaligned/aligned results), mzRAPP derives performance metrics covering missed peaks, peak abundance quality, nature of missed peaks, and alignment error counts, with bootstrapped confidence intervals (alpha = 0.95).: [evidence span withheld — anchor status: not_found]
- [readme] Isotopologue peaks with an area or height which is more than 30% off the predicted value or a Pearson Correlation coef < 0.85 (as compared to the highest isotopologue) are removed.: [evidence span withheld — anchor status: not_found]
- [readme] The resulting benchmark dataset is used to extract different performance metrics for NPP performed on the same mzML files.: [evidence span withheld — anchor status: external_doc]
- [abstract] mzRAPP produces several performance metrics for different steps of the pre-processing workflow: "mzRAPP produces several performance metrics for different steps of the pre-processing workflow"
- [readme] In order to generate a benchmark you need to provide your centroided mzML files.: [evidence span withheld — anchor status: external_doc]
