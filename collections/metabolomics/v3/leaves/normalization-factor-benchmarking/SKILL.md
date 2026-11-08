---
name: normalization-factor-benchmarking
description: Use when you need to test whether a normalization-factor estimator stays
  unbiased when the fraction of up-regulated features is imbalanced, e.g. validating
  MDFC-like estimators against median FC or other MAFFIN norm methods.
when_to_use_negative:
- Input is real experimental data with unknown true normalization factor — benchmarking
  requires simulated data with a known ground-truth dilution factor (k=1.5 here)
- You need to actually normalize a dataset rather than evaluate estimator accuracy
  — use MAFFINNorm/MDFCNorm directly on the feature table instead of running the sweep
  benchmark
- Serial QC samples (SQC_###) are unavailable and intensity correction cannot be run,
  if your goal is to benchmark the full MAFFIN pipeline rather than the normalization-factor
  estimator alone
tools:
- name: MAFFIN
  role: R package supplying MDFCNorm (kernel-density maximal density fold change)
    and MedianNorm (median fold change) used as the compared normalization-factor
    estimators in the benchmark
  repo: github.com/HuanLab/MAFFIN
- name: R
  role: Environment in which the simulated feature tables are generated and the MAFFIN
    benchmark functions are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac355
    title: 'MAFFIN: metabolomics sample normalization using maximal density fold change
      with high-quality metabolic features and corrected signal intensities'
  source_content_hash: sha256:090d505b16e4cfcb297d21c3fa6ffbf4eef3448cf57ec856f26d550a6dd150cd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/HuanLab/MAFFIN
  tools:
  - MAFFIN
  - MDFCNorm
  - MedianNorm
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# normalization-factor-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark metabolomics sample-normalization factor estimators under imbalanced differential regulation by simulating a two-feature-intensity table (5000 features, true dilution factor k=1.5) and comparing MAFFIN's kernel-density MDFC estimator against median fold change across up-regulated fractions of 5–45%.

## When to use

Use when you need to test whether a normalization-factor estimator stays unbiased when the fraction of up-regulated features is imbalanced, e.g. validating MDFC-like estimators against median FC or other MAFFIN norm methods.

## When NOT to use

- Input is real experimental data with unknown true normalization factor — benchmarking requires simulated data with a known ground-truth dilution factor (k=1.5 here)
- You need to actually normalize a dataset rather than evaluate estimator accuracy — use MAFFINNorm/MDFCNorm directly on the feature table instead of running the sweep benchmark
- Serial QC samples (SQC_###) are unavailable and intensity correction cannot be run, if your goal is to benchmark the full MAFFIN pipeline rather than the normalization-factor estimator alone

## Inputs

- Simulated two-sample feature intensity table (dataframe, features in rows, samples in columns, first row of group labels) with 5000 features, 50% unchanged, up-regulated fraction swept 5–45%, true dilution factor k=1.5
- IntThreshold intensity threshold parameter for the MAFFIN norm functions
- SampleInCol flag indicating samples are in columns

## Outputs

- Per-fraction estimated normalization factors (mean and SD across replicates) for MDFC and median FC, mirroring Figure 4A
- figure4A_normalization_factors.csv
- Validation verdict: MDFC at 1.5001 ± 0.003 across all up-regulated fractions vs median FC spanning 1.39–1.63

## How to apply

In R, generate a simulated two-sample feature intensity table following the Methods design: 5000 features, 50% unchanged between groups, up-regulated feature fractions swept from 5% to 45% in 5% increments, and a true dilution factor k=1.5. For each fraction, call MAFFIN's MDFCNorm(FeatureTable, IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation, bwOpt) to compute kernel-density-based maximal density fold change factors, and MedianNorm(FeatureTable, IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation) for median fold change factors. Aggregate estimated normalization factors per up-regulated fraction as mean ± SD across replicates into a table mirroring Figure 4A with one column for MDFC and one for median FC, and write it to figure4A_normalization_factors.csv. The rationale is that median FC is biased under dilution imbalance, while MDFC's maximal-density fold change should recover the true factor regardless of imbalance direction or magnitude.

## Related tools

- **MAFFIN** (R package supplying MDFCNorm (kernel-density maximal density fold change) and MedianNorm (median fold change) used as the compared normalization-factor estimators in the benchmark) — github.com/HuanLab/MAFFIN
- **R** (Environment in which the simulated feature tables are generated and the MAFFIN benchmark functions are executed)

## Examples

```
MAFFINTable = MAFFINNorm(inputTable)
```

## Evaluation signals

- MDFC-estimated normalization factors stay at 1.5001 ± 0.003 across every up-regulated fraction from 5% to 45%
- Median FC estimates span 1.39–1.63 as the up-regulated fraction is varied, reproducing the reported bias
- Aggregate table has one column per estimator (MDFC, median FC) with one mean and SD per 5% up-regulated fraction, matching Figure 4A structure
- Each estimator call uses the documented MAFFIN signatures, including OutputNormFactors=TRUE so factors are returned, not just the normalized table

## Limitations

- Results depend on the simulation design (5000 features, 50% unchanged, k=1.5); other true dilution factors or imbalance structures were not covered by this benchmark
- Median FC is biased under imbalanced up/down regulation — expect deviation from the true factor rather than interpreting its estimates as correct
- MAFFIN expects group labels in the first row and reserves the labels blank, RT, QC, and SQC_###; simulated tables must respect this format

## Evidence

- [results] MDFC remains accurate under imbalance: [evidence span withheld — anchor status: not_found]
