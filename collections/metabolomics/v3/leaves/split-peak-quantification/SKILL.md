---
name: split-peak-quantification
description: Use when you have a validated mzRAPP benchmark of target-molecule peaks
  and a tool-specific NPP output table (XCMS, MZmine 2, MS-DIAL, etc.) and need the
  proportion of benchmark peaks split into multiple detected peaks.
when_to_use_negative:
- You have only a raw feature table and no benchmark of target molecules with known
  retention-time boundaries — mzRAPP cannot classify split peaks without the benchmark
- The question concerns alignment errors across files rather than peak-picking quality
  within files (use the alignment error counting metrics instead)
- Data are not LC-HRMS peak tables in a supported NPP tool's export format (mzRAPP
  supports XCMS, XCMS3, MZmine 2/3, MS-DIAL, OpenMS, El-MAVEN, SLAW, etc.)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: mzRAPP
  role: R package performing the non-equivalent joins between benchmark and NPP outputs
    and deriving the split-peaks performance metric
  repo: https://github.com/YasinEl/mzRAPP
- name: R
  role: Implementation language; mzRAPP functions are called from R (optionally via
    the shiny interface callmzRAPP())
- name: XCMS
  role: Source of the NPP output tables (unaligned/aligned run 2 CSVs) whose split-peaks
    rate is assessed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:942fb1a420c3c42cbf3149191506323cd5c4b3df27114da324974f918b2fe118
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - XCMS
  - mzRAPP
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# split-peak-quantification

## Summary

Quantifies the split-peaks metric of peak-picking performance by matching a mzRAPP-generated benchmark (e.g. Benchmark.csv built from MTBLS267 LC-HRMS mzML files) against non-targeted pre-processing (NPP) output tables such as XCMS run 2 unaligned/aligned CSVs, using mzRAPP's non-equivalent joins.

## When to use

Use when you have a validated mzRAPP benchmark of target-molecule peaks and a tool-specific NPP output table (XCMS, MZmine 2, MS-DIAL, etc.) and need the proportion of benchmark peaks split into multiple detected peaks.

## When NOT to use

- You have only a raw feature table and no benchmark of target molecules with known retention-time boundaries — mzRAPP cannot classify split peaks without the benchmark
- The question concerns alignment errors across files rather than peak-picking quality within files (use the alignment error counting metrics instead)
- Data are not LC-HRMS peak tables in a supported NPP tool's export format (mzRAPP supports XCMS, XCMS3, MZmine 2/3, MS-DIAL, OpenMS, El-MAVEN, SLAW, etc.)

## Inputs

- Benchmark.csv benchmark file (prepared via mzRAPP benchmark generation from centroided mzML files, sample-group file and target file)
- Unaligned NPP output CSV, e.g. XCMS_unaligned_run2.csv
- Aligned NPP output CSV, e.g. XCMS_aligned_run2.csv

## Outputs

- Peak Picking performance metrics including the proportion of matches classified as split peaks (expected in the 1-3% range for the MTBLS267 XCMS run 2 example)
- xcms_run2_assessment.csv with all derived metrics

## How to apply

Load mzRAPP in R and read the prepared Benchmark.csv downloaded from the ucloud deposit. Import the unaligned NPP output with import_ungrouped_xcms (e.g. XCMS_unaligned_run2.csv) and the aligned output with import_grouped_xcms (e.g. XCMS_aligned_run2.csv), then validate both with check_nonTargeted_input. Match benchmark peaks to NPP peaks via match_peaks_to_benchmark and benchmark features to NPP features via match_features_to_benchmark; these functions perform the non-equivalent joins between benchmark and NPP tables that define which benchmark peaks were fragmented into multiple NPP peaks. Run derive_performance_metrics and extract the Peak Picking box metrics, in particular the proportion of matches classified as split peaks. A split peak arises when a single benchmark peak is matched by more than one NPP peak, so the join cardinality (1-to-many) is the key decision point; the article reports split peaks around 1-3% for the example XCMS run.

## Related tools

- **mzRAPP** (R package performing the non-equivalent joins between benchmark and NPP outputs and deriving the split-peaks performance metric) — https://github.com/YasinEl/mzRAPP
- **R** (Implementation language; mzRAPP functions are called from R (optionally via the shiny interface callmzRAPP()))
- **XCMS** (Source of the NPP output tables (unaligned/aligned run 2 CSVs) whose split-peaks rate is assessed)

## Examples

```
library(mzRAPP); u <- import_ungrouped_xcms('XCMS_unaligned_run2.csv'); g <- import_grouped_xcms('XCMS_aligned_run2.csv'); check_nonTargeted_input(u, g); match_peaks_to_benchmark(u, read.csv('Benchmark.csv')); derive_performance_metrics()
```

## Evaluation signals

- The split-peaks percentage from bootstrapped confidence intervals falls within the 1-3% range stated in the article for the MTBLS267 XCMS run 2 example
- Both NPP inputs pass check_nonTargeted_input without errors after import via import_ungrouped_xcms and import_grouped_xcms
- match_peaks_to_benchmark returns matches with expected 1-to-many cardinality: a benchmark peak matched by multiple NPP peaks is flagged as split
- All derived metrics are written to xcms_run2_assessment.csv without missing Peak Picking box metrics
- The benchmark was validated per README criteria: isotopologues with area/height >30% off predicted values or Pearson correlation < 0.85 were removed before benchmarking

## Limitations

- Only isotopologues for which the theoretically most abundant and at least one additional isotopologue are detected are included in the benchmark, limiting coverage of low-abundance peaks
- Adducts are only added to the benchmark if at least two isotopologues of the respective adduct are detected; if the main_adduct (e.g. M+H or M-H) is not detected, other adducts are not accepted
- Package updates can be problematic during installation; the README advises selecting 'None' when asked to update packages before installing mzRAPP

## Evidence

- [abstract] mzRAPP produces several performance metrics for different steps of the pre-processing workflow: "mzRAPP produces several performance metrics for different steps of the pre-processing workflow"
- [other] mzRAPP's compare_peaks performs non-equivalent joins: [evidence span withheld — anchor status: not_found]
- [readme] README: benchmark used to extract performance metrics for NPP on same mzML files: [evidence span withheld — anchor status: external_doc]
- [readme] README: use-case vignette for metrics and interpretation: [evidence span withheld — anchor status: external_doc]
- [other] methods: bootstrapped CI 1-3% validation: [evidence span withheld — anchor status: not_found]
