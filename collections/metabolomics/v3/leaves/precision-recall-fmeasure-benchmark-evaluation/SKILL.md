---
name: precision-recall-fmeasure-benchmark-evaluation
description: Use when an LC-HRMS feature matching or alignment result must be validated
  against a ground-truth reference feature list, and TP/FN counts plus precision,
  recall and F-measure need to be reproduced or compared to a published benchmark
  (e.g., Table 3 values).
when_to_use_negative:
- Data exhibits m/z or retention time shifts requiring correction — the benchmark
  protocol applies only to the unmodified reference-vs-reference comparison without
  shift correction
- No ground-truth reference feature list exists, so TP/FN cannot be counted against
  known matches
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetMatch
  role: Executes the benchmark matching/alignment of the unmodified Phe reference
    chromatogram against the 1482-feature reference list, producing the matched/unmatched
    feature assignments that are scored
  repo: github.com/S-T-K/MetMatch
- name: Java
  role: Runtime environment required to run MetMatch (JRE 1.8 or newer) for the benchmark
    matching run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:98a52734a2b94004e5fec9ec173bf6a1990db67b36f74173833a411e3a11de32
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - MetMatch
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# precision-recall-fmeasure-benchmark-evaluation

## Summary

Benchmark evaluation protocol in which a matching/alignment result is scored against a known ground-truth reference by counting true positives and false negatives and computing precision, recall and F-measure. In the MetMatch paper this validates the unmodified Phe reference chromatogram matched against a 1482-feature reference list (1446 matched, F-measure 0.988).

## When to use

Use when an LC-HRMS feature matching or alignment result must be validated against a ground-truth reference feature list, and TP/FN counts plus precision, recall and F-measure need to be reproduced or compared to a published benchmark (e.g., Table 3 values).

## When NOT to use

- Data exhibits m/z or retention time shifts requiring correction — the benchmark protocol applies only to the unmodified reference-vs-reference comparison without shift correction
- No ground-truth reference feature list exists, so TP/FN cannot be counted against known matches

## Inputs

- Ground-truth reference feature list with m/z and retention time entries (1482 features for the Phe case study)
- Unmodified reference LC-HRMS chromatogram (Phe reference chromatogram from the wheat phenylalanine-derived metabolite experiments)
- MetMatch matching/alignment output combining peak areas, m/z values and retention times

## Outputs

- True-positive and false-negative counts from the benchmark match (expected: 1446 TP, 36 FN)
- Precision, recall and F-measure values (expected F-measure 0.988)
- Benchmark results table (e.g., phe_benchmark_table3.csv) for comparison against Table 3

## How to apply

Load the ground-truth reference chromatogram and reference feature list (in the MetMatch Phe case study, 1482 features compiled per supplementary Sections 2.1/2.2) into the alignment tool and run the matching pipeline without shift correction, since reference-vs-reference matching serves as the noise-free benchmark. From the resulting combined data matrix of matched peak areas, m/z values and retention times, count true positives (reference features correctly matched) and false negatives (reference features that could not be matched). Compute precision = TP/(TP+FP), recall = TP/(TP+FN) and F-measure = 2*precision*recall/(precision+recall). Write the counts and metrics to a results file (e.g., phe_benchmark_table3.csv) and diff them against the published benchmark entries — here 1446 of 1482 features matched (36 unmatched), F-measure 0.988. Any deviation signals a pipeline or input error rather than a genuine performance difference, because the reference-vs-reference comparison is deterministic.

## Related tools

- **MetMatch** (Executes the benchmark matching/alignment of the unmodified Phe reference chromatogram against the 1482-feature reference list, producing the matched/unmatched feature assignments that are scored) — github.com/S-T-K/MetMatch
- **Java** (Runtime environment required to run MetMatch (JRE 1.8 or newer) for the benchmark matching run)

## Evaluation signals

- True-positive count equals 1446 and false-negative count equals 36 of the 1482 reference features
- Computed precision, recall and F-measure reproduce the Table 3 benchmark entries, with F-measure 0.988
- phe_benchmark_table3.csv values match the published Table 3 row for the unmodified reference-vs-reference comparison exactly (deterministic match)
- Every one of the 1482 reference features is classified as either matched or unmatched with no double assignments

## Limitations

- The benchmark reflects a noise-free reference-vs-reference comparison; performance on shifted real data (where MetMatch corrects m/z and retention time shifts) will be lower
- MetMatch requires Java runtime (JRE) version 1.8 or newer
- The software is distributed without warranty (GNU GPL), and the discussion section content was not available in the source document, limiting documented caveats

## Evidence

- [abstract] Corresponding ions and metabolites are matched and combined into a single data matrix: "Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix."
- [intro] MetMatch provides semi-automated comparison of datasets: "MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments."
- [intro] MetMatch corrects shifts relative to a reference chromatogram: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [readme] Java runtime requirement: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
