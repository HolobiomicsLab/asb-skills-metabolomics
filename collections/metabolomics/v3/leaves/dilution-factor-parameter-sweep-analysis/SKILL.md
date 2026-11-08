---
name: dilution-factor-parameter-sweep-analysis
description: 'Use when validating a normalization estimator under dilution: a two-group
  feature-intensity table with a known true dilution factor (e.g. k=1.5) and varying
  imbalanced fractions of up-regulated features.'
when_to_use_negative:
- Real experimental data with unknown ground-truth dilution factor — the sweep validation
  requires a known true k to judge estimator accuracy
- Datasets where the goal is normalization of a single study rather than benchmarking
  estimator bias under feature-imbalance variation
- Feature tables lacking a two-group structure, since the dilution fold-change estimation
  needs two-sample fold changes
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MAFFIN
  role: R package providing MDFCNorm (maximal density fold change normalization) and
    MedianNorm (median fold change normalization) used to compute normalization factors
    at each up-regulated fraction in the sweep
  repo: https://github.com/HuanLab/MAFFIN
- name: R
  role: Environment for generating the simulated feature-intensity table, installing
    MAFFIN via devtools, running the parameter sweep, and writing the CSV output
- name: devtools
  role: Used to install the MAFFIN package before running the sweep (install.packages('devtools');
    devtools::install_github('Waddlessss/MAFFIN'))
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac355
    title: 'MAFFIN: metabolomics sample normalization using maximal density fold change
      with high-quality metabolic features and corrected signal intensities'
  source_content_hash: sha256:d9624ef974198f28cc794153b5dafa6e22c5fbff66e41a20eaf5add878aabc2a
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
  - devtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# dilution-factor-parameter-sweep-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sweeps imbalanced up-regulated feature fractions in a simulated two-sample metabolomics feature-intensity table with a known true dilution factor, comparing MAFFIN's MDFC normalization-factor estimates against median fold change (MedianNorm) to quantify bias.

## When to use

Use when validating a normalization estimator under dilution: a two-group feature-intensity table with a known true dilution factor (e.g. k=1.5) and varying imbalanced fractions of up-regulated features.

## When NOT to use

- Real experimental data with unknown ground-truth dilution factor — the sweep validation requires a known true k to judge estimator accuracy
- Datasets where the goal is normalization of a single study rather than benchmarking estimator bias under feature-imbalance variation
- Feature tables lacking a two-group structure, since the dilution fold-change estimation needs two-sample fold changes

## Inputs

- Simulated two-sample feature intensity table (dataframe, features in rows, samples in columns, first row = group labels) matching the Methods design: 5000 features, 50% unchanged, up-regulated fractions 5%-45% in 5% steps, true dilution factor k=1.5
- IntThreshold parameter value for MAFFIN feature-intensity filtering
- Replicate structure for aggregating normalization-factor estimates per up-regulated fraction

## Outputs

- Per-fraction table of estimated normalization factors (mean and SD across replicates) with one MDFC column and one median-FC column, mirroring Figure 4A
- figure4A_normalization_factors.csv
- Optionally RunEvaluation outputs (e.g. intragroup variation) for each normalization call

## How to apply

In R with the MAFFIN package, generate the simulated two-sample feature-intensity table per the Methods design: 5000 features, 50% unchanged between groups, up-regulated feature fractions swept from 5% to 45% in 5% increments, and a true dilution factor of k=1.5. For each fraction, run MDFCNorm(FeatureTable, IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation, bwOpt) to obtain kernel-density-based maximal density fold change factors, and MedianNorm(FeatureTable, IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation) for median fold change factors. Aggregate mean and standard deviation of estimated factors per fraction across replicates into a table mirroring Figure 4A (one MDFC column, one median-FC column), then write to figure4A_normalization_factors.csv. The rationale is that imbalanced percentages of up- and down-regulated features bias the median FC, so sweeping the imbalance reveals estimator bias while a correct MDFC estimate should track the true dilution factor.

## Related tools

- **MAFFIN** (R package providing MDFCNorm (maximal density fold change normalization) and MedianNorm (median fold change normalization) used to compute normalization factors at each up-regulated fraction in the sweep) — https://github.com/HuanLab/MAFFIN
- **R** (Environment for generating the simulated feature-intensity table, installing MAFFIN via devtools, running the parameter sweep, and writing the CSV output)
- **devtools** (Used to install the MAFFIN package before running the sweep (install.packages('devtools'); devtools::install_github('Waddlessss/MAFFIN')))

## Examples

```
devtools::install_github("Waddlessss/MAFFIN"); factors = MDFCNorm(FeatureTable, IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation, bwOpt)
```

## Evaluation signals

- MDFC estimates remain stable at 1.5001 ± 0.003 across all up-regulated fractions from 5% to 45%
- Median FC estimates span approximately 1.39-1.63 across the sweep, confirming the expected bias pattern of Figure 4A
- Aggregated table contains mean and standard deviation per fraction for both MDFC and median-FC columns
- figure4A_normalization_factors.csv is produced with one row per up-regulated fraction
- MAFFIN's intragroup-variation evaluation (EvaPRMAD/EvaRSD style outputs from RunEvaluation=TRUE) can be attached per call to sanity-check normalization quality

## Limitations

- The benchmark depends on the specific simulation design (5000 features, 50% unchanged, k=1.5); results may not generalize to other numbers of features or other true dilution factors
- The sweep only tests imbalanced up-regulated fractions from 5% to 45% in 5% increments; extreme imbalances outside this range are untested
- MDFCNorm relies on kernel density estimation with an optional bandwidth optimization (bwOpt), so bandwidth choice could affect factor estimates

## Evidence

- [methods] MAFFIN is an integrated R package for metabolomics sample normalization.: "MAFFIN is an integrated R package for metabolomics sample normalization."
- [methods] MDFCNorm signature: "MDFCNorm(FeatureTable, IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation, bwOpt)"
- [methods] MedianNorm signature: "MedianNorm(FeatureTable, IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation)"
- [readme] Intragroup variation evaluation: "MAFFIN package uses intragroup variation to evaluate the normalization results. The lower intragroup variation means better sample normalization."
- [results] Figure 4A finding: [evidence span withheld — anchor status: not_found]
