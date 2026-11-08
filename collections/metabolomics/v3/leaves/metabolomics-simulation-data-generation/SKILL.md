---
name: metabolomics-simulation-data-generation
description: Use when benchmarking sample-normalization estimators on metabolomics
  data where ground truth is needed — e.g., testing whether MDFC stays unbiased under
  imbalanced up/down-regulation while median FC is expected to be biased.
when_to_use_negative:
- The input is a real experimental feature intensity table — normalize it directly
  with MAFFINNorm/MedianNorm rather than simulating data first.
- The goal is evaluating normalization on real studies where serial QC samples (SQC_###)
  exist for MS signal intensity correction — simulation does not exercise the IntCorrection
  module.
- You need to compare more normalization methods (SumNorm, QuantileNorm, PQNNorm)
  without a controlled imbalance sweep or known ground-truth factor.
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Language in which the simulated feature intensity tables are generated and
    the benchmark is executed
  repo: https://github.com/HuanLab/MAFFIN
- name: MAFFIN
  role: R package supplying the normalization functions (MDFCNorm, MedianNorm) applied
    to the simulated tables and the input format (feature intensity dataframe) the
    simulation must mimic
  repo: https://github.com/HuanLab/MAFFIN
- name: MDFCNorm
  role: Computes kernel-density-based maximal density fold change normalization factors
    on each simulated table; expected to return ~1.5 regardless of imbalance fraction
  repo: https://github.com/Waddlessss/MAFFIN
- name: MedianNorm
  role: Computes median fold change normalization factors as the biased baseline estimator
    on each simulated table
  repo: https://github.com/Waddlessss/MAFFIN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac355
    title: 'MAFFIN: metabolomics sample normalization using maximal density fold change
      with high-quality metabolic features and corrected signal intensities'
  source_content_hash: sha256:322009042ea7aee3ed37a91479b6f012220582d748169abe5728c4c6a82f1ba0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/HuanLab/MAFFIN
  repo_url: https://github.com/HuanLab/MAFFIN
  tools:
  - MAFFIN
  - MDFCNorm
  - MedianNorm
  - R
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolomics-simulation-data-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate simulated metabolomics feature intensity tables with a controlled imbalance of up- and down-regulated features and a known true dilution factor, so that normalization estimators (e.g., MAFFIN's MDFC vs. median fold change) can be benchmarked against ground truth.

## When to use

Use when benchmarking sample-normalization estimators on metabolomics data where ground truth is needed — e.g., testing whether MDFC stays unbiased under imbalanced up/down-regulation while median FC is expected to be biased.

## When NOT to use

- The input is a real experimental feature intensity table — normalize it directly with MAFFINNorm/MedianNorm rather than simulating data first.
- The goal is evaluating normalization on real studies where serial QC samples (SQC_###) exist for MS signal intensity correction — simulation does not exercise the IntCorrection module.
- You need to compare more normalization methods (SumNorm, QuantileNorm, PQNNorm) without a controlled imbalance sweep or known ground-truth factor.

## Inputs

- Simulated two-sample feature intensity table in R (dataframe; features in rows, samples in columns; first row = group labels, formatted like MAFFIN TestingData)
- Simulation design parameters: 5000 features, 50% unchanged, up-regulated fraction swept 5%-45% in 5% steps, true dilution factor k=1.5

## Outputs

- Per-fraction mean and standard deviation of estimated normalization factors for MDFC and median FC, aggregated across replicates
- figure4A_normalization_factors.csv mirroring Figure 4A behavior
- Optionally MAFFIN-style outputs: named numeric vector of normalization factors and intragroup variation (pooled RMAD) evaluation per method

## How to apply

Build the simulated two-sample feature table in R following the Methods design: 5000 features, 50% of features unchanged between groups, the fraction of up-regulated features swept from 5% to 45% in 5% increments, and a true dilution factor of k=1.5. Format the table as a feature intensity dataframe with features in rows and samples in columns, with sample group names in the first row, matching MAFFIN's expected input (as in TestingData). For each up-regulated fraction, compute kernel-density-based maximal density fold change factors with MDFCNorm(FeatureTable, IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation, bwOpt) and median fold change factors with MedianNorm(FeatureTable, IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation). Aggregate mean and standard deviation of estimated normalization factors per fraction across replicates into a two-column (MDFC, median FC) table mirroring Figure 4A, then write it to figure4A_normalization_factors.csv. The rationale: a known dilution factor plus controlled feature-imbalance fractions isolates the bias of median-based normalization that MAFFIN's MDFC is designed to avoid.

## Related tools

- **R** (Language in which the simulated feature intensity tables are generated and the benchmark is executed) — https://github.com/HuanLab/MAFFIN
- **MAFFIN** (R package supplying the normalization functions (MDFCNorm, MedianNorm) applied to the simulated tables and the input format (feature intensity dataframe) the simulation must mimic) — https://github.com/HuanLab/MAFFIN
- **MDFCNorm** (Computes kernel-density-based maximal density fold change normalization factors on each simulated table; expected to return ~1.5 regardless of imbalance fraction) — https://github.com/Waddlessss/MAFFIN
- **MedianNorm** (Computes median fold change normalization factors as the biased baseline estimator on each simulated table) — https://github.com/Waddlessss/MAFFIN

## Examples

```
MedianNormedTable = MedianNorm(inputTable); mdfcFactors = MDFCNorm(inputTable, IntThreshold, SampleInCol=TRUE, OutputNormFactors=TRUE)
```

## Evaluation signals

- MDFC normalization factor estimates remain stable at 1.5001 ± 0.003 across all up-regulated fractions (5%-45%).
- Median FC estimates span 1.39-1.63 across the same fractions, confirming the expected imbalance-induced bias.
- Output table mirrors Figure 4A: one column for MDFC, one for median FC, one row per up-regulated fraction with mean ± SD across replicates.
- Simulated table structure matches MAFFIN input requirements: dataframe, features in rows, samples in columns, group names in first row (5000 features, 50% unchanged).
- Optional cross-check: MAFFIN's intragroup variation evaluation (lower pooled RMAD means better normalization) can be run via RunEvaluation=TRUE.

## Limitations

- Simulation design is fixed to the Methods parameters (5000 features, 50% unchanged, dilution factor 1.5); extrapolation to other feature counts or factors is untested in this benchmark.
- Simulated tables lack serial QC samples, so the MS signal intensity correction module of MAFFIN is not exercised by this benchmark.
- No changelog found in the source material.

## Evidence
