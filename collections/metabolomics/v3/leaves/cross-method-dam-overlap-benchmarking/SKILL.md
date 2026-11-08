---
name: cross-method-dam-overlap-benchmarking
description: Use when one raw feature table has been normalized by ≥2 methods and
  you must quantify how normalization affects differential abundance results, e.g.
  reproduce a reported DAM overlap range between robust and QC-RLSC normalization.
when_to_use_negative:
- You have only a single normalization method applied — pairwise overlap benchmarking
  needs DAM lists from at least two methods on the identical feature set
- Final published inference is the goal — p < 0.05 calls here are uncorrected for
  benchmarking comparison, and the paper notes 'multiple testing correction is essential
  for downstream statistical inference'
- You plan to rely on SERFF for normalization of a very large table — the SERFF Shiny
  interface failed to complete on the 618×8913 FAME dataset because capacity was exceeded
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Metanorm R package
  role: Implements rLOESS, rGAM, tGAM, QC-RLSC and QC-RSC normalization (batchwise,
    with QCcheck) whose outputs feed the DAM overlap comparison
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment for Metanorm normalization and the downstream Wilcoxon
    rank sum DAM testing and overlap computation
  repo: https://github.com/UGent-LIMET/Metanorm
- name: SERFF
  role: Multivariate normalization method attempted via its Shiny web interface for
    the benchmark; recorded as failing at FAME dataset scale
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:4dd2c68e679a22e5eca31a8599940a0cef1aeba99b30d6f54251c11e7792a57d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - Metanorm R package
  - R
  - SERFF
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# Cross-method DAM overlap benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark how the choice of normalization method (robust rLOESS/rGAM/tGAM vs QC-sample-based QC-RLSC/QC-RSC) changes downstream differential abundance calls by computing pairwise DAM overlap percentages on the same metabolomics feature table, as done on the 618-run × 8913-feature FAME saliva cohort.

## When to use

Use when one raw feature table has been normalized by ≥2 methods and you must quantify how normalization affects differential abundance results, e.g. reproduce a reported DAM overlap range between robust and QC-RLSC normalization.

## When NOT to use

- You have only a single normalization method applied — pairwise overlap benchmarking needs DAM lists from at least two methods on the identical feature set
- Final published inference is the goal — p < 0.05 calls here are uncorrected for benchmarking comparison, and the paper notes 'multiple testing correction is essential for downstream statistical inference'
- You plan to rely on SERFF for normalization of a very large table — the SERFF Shiny interface failed to complete on the 618×8913 FAME dataset because capacity was exceeded

## Inputs

- Raw metabolomics feature-intensity matrix (rows = compounds/features, columns = sample runs), e.g. 618 runs × 8913 metabolic features
- Batch vector assigning each sample run to a batch
- Sample-type vector distinguishing QC runs from biological samples
- Group labels for the contrast, e.g. healthy-weight (n=337) vs overweight/obesity (n=99)

## Outputs

- One normalized intensity matrix per normalization method (rLOESS, rGAM, tGAM, QC-RLSC, QC-RSC)
- Per-method DAM lists (two-sided Wilcoxon rank sum test, p < 0.05)
- Pairwise DAM overlap percentage table across methods
- Benchmark comparison against reported overlap values (robust 79–87%, QC-RLSC 55–59%) and an explicit SERFF failure record where applicable

## How to apply

Start from the raw feature-intensity matrix (compounds × sample runs) and natural-log (base e) transform all intensities before normalization. Normalize batchwise with the Metanorm R package under each candidate model — rLOESS, rGAM, tGAM (rGAM/tGAM run with the QCcheck argument active) plus QC-RLSC and QC-RSC — supplying the batch vector and the QC/sample-type vector; optionally attempt SERFF via its Shiny interface and record it if it fails at dataset scale. For each normalization, call DAMs between the two biological groups (e.g. healthy-weight n=337 vs overweight/obesity n=99) using two-sided Wilcoxon rank sum tests with significance at p < 0.05. Compute pairwise DAM overlap percentages across the resulting per-method DAM lists and compare them against the reference values — robust methods agreeing at 79–87% overlap, QC-RLSC agreeing at only 55–59% — to judge whether robust normalization recovers a consistent underlying signal.

## Related tools

- **Metanorm R package** (Implements rLOESS, rGAM, tGAM, QC-RLSC and QC-RSC normalization (batchwise, with QCcheck) whose outputs feed the DAM overlap comparison) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment for Metanorm normalization and the downstream Wilcoxon rank sum DAM testing and overlap computation) — https://github.com/UGent-LIMET/Metanorm
- **SERFF** (Multivariate normalization method attempted via its Shiny web interface for the benchmark; recorded as failing at FAME dataset scale)

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch)
```

## Evaluation signals

- Pairwise overlap percentages among robust methods (rLOESS, rGAM, tGAM) fall in the reported 79–87% range
- QC-RLSC overlap with the other methods falls in the weaker reported 55–59% range
- All methods are tested on the same ln-transformed, batchwise-normalized feature set so overlaps are computed over identical candidate DAM universes
- Each method's DAM list is produced by two-sided Wilcoxon rank sum tests with the stated group sizes (n=337 vs n=99) at p < 0.05
- The validation output contains an explicit SERFF failure record when attempted at FAME dataset scale

## Limitations

- SERFF normalization could not be obtained for the 618-run × 8913-feature FAME dataset, likely because it exceeded the Shiny server's capacity
- Differential abundance testing at p < 0.05 omits multiple testing correction, which the authors flag as essential for downstream statistical inference
- Filtering features on relative standard deviation (RSD) in QC samples can bias cross-method comparison, since differences between QC and biological samples may invalidate such filtering
- Normalization choice itself materially changes DAM results ('distinct differential abundance results'), so overlaps benchmark sensitivity rather than establish one method as ground truth

## Evidence

- [abstract] abstract: "The robust methods, with improved recovery of the underlying signal demonstrated in simulation, produced distinct differential abundance results, highlighting the impact of normalization on"
- [discussion] discussion: "Overall, the tGAM method exhibited the strongest performance across the experimental data sets, whereas performance on simulated data was comparable between rLOESS, rGAM, and tGAM"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
