---
name: median-fold-change-normalization
description: Use when computing median fold change normalization factors from a metabolomics
  feature intensity table, or when demonstrating median FC bias against kernel-density
  MDFC under imbalanced up-/down-regulated feature fractions.
when_to_use_negative:
- The goal is an accurate normalization factor when up- and down-regulated feature
  percentages are imbalanced — median FC is the biased baseline (1.39–1.63 vs true
  1.5); use MDFCNorm/MAFFINNorm instead
- Serial QC samples (SQC_###) are available and MS signal intensity correction or
  high-quality feature selection is wanted — use the full three-module MAFFINNorm
  pipeline rather than standalone median FC
- Input is not a feature intensity table (features × samples with a group-labelled
  first row), e.g. raw vendor spectra files — MAFFIN's MedianNorm cannot consume them
  directly
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MAFFIN
  role: Integrated R package providing MedianNorm, MDFCNorm, and intragroup-variation
    evaluation used by this skill
  repo: github.com/HuanLab/MAFFIN
- name: MedianNorm
  role: MAFFIN function that computes median fold change normalization factors — the
    biased baseline estimator benchmarked in Figure 4A
  repo: github.com/Waddlessss/MAFFIN
- name: MDFCNorm
  role: MAFFIN kernel-density MDFC function run in parallel as the accurate comparator
    (1.5001 ± 0.003 vs true dilution factor 1.5)
  repo: github.com/Waddlessss/MAFFIN
- name: R
  role: Runtime language used to generate the simulated 5000-feature tables and call
    MAFFIN normalization functions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac355
    title: 'MAFFIN: metabolomics sample normalization using maximal density fold change
      with high-quality metabolic features and corrected signal intensities'
  source_content_hash: sha256:9def46bb7e6f913ac3eb2076a34c1095d53e3e522b76a900b802d09c7662b8d9
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
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# median-fold-change-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes median fold change (median FC) normalization factors from a metabolomics feature intensity table using MAFFIN's MedianNorm, and benchmarks it against the kernel-density MDFC estimator under imbalanced up-/down-regulated feature fractions. Median FC is retained here as the biased baseline estimator.

## When to use

Use when computing median fold change normalization factors from a metabolomics feature intensity table, or when demonstrating median FC bias against kernel-density MDFC under imbalanced up-/down-regulated feature fractions.

## When NOT to use

- The goal is an accurate normalization factor when up- and down-regulated feature percentages are imbalanced — median FC is the biased baseline (1.39–1.63 vs true 1.5); use MDFCNorm/MAFFINNorm instead
- Serial QC samples (SQC_###) are available and MS signal intensity correction or high-quality feature selection is wanted — use the full three-module MAFFINNorm pipeline rather than standalone median FC
- Input is not a feature intensity table (features × samples with a group-labelled first row), e.g. raw vendor spectra files — MAFFIN's MedianNorm cannot consume them directly

## Inputs

- Feature intensity table (data frame; features in rows, samples in columns, first row holding group labels such as blank, RT, QC, SQC_###; read from .csv via read.csv)
- Simulated two-sample table per the Methods design: 5000 features, 50% unchanged, up-regulated fraction swept 5%–45% in 5% steps, true dilution factor k = 1.5
- MedianNorm/MDFCNorm parameters: IntThreshold, SampleInCol, OutputNormFactors, RunEvaluation, bwOpt (MDFC only)

## Outputs

- Named numeric vector of median FC normalization factors (one per biological sample) when OutputNormFactors=TRUE
- Normalized feature intensity table (MedianNormedTable) when output=TRUE
- figure4A_normalization_factors.csv: mean ± SD estimated normalization factors per up-regulated fraction, one column for MDFC and one for median FC (mirroring Figure 4A)
- Intragroup-variation metrics (pooled RMAD/RSD) when RunEvaluation=TRUE

## How to apply

Read the feature intensity table into R with read.csv so features are rows and samples are columns (SampleInCol), with sample group names in the first row; biological group labels must not be blank, RT, QC, or SQC_###. Call MedianNorm(FeatureTable, IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation) to obtain the median FC normalization factors — the estimator that assumes roughly balanced regulation. In parallel, call MDFCNorm(FeatureTable, IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation, bwOpt) to get the kernel-density-based maximal density fold change factors as the comparator. For the Figure 4A benchmark, simulate two-sample tables of 5000 features with 50% unchanged and the up-regulated fraction swept from 5% to 45% in 5% increments at a true dilution factor k=1.5, then aggregate mean ± SD estimated factors per fraction into figure4A_normalization_factors.csv with one column for MDFC and one for median FC. Evaluate normalization quality with intragroup variation (EvaPRMAD, pooled RMAD/RSD), since lower intragroup variation means better sample normalization. Validation: MDFC factors hold at 1.5001 ± 0.003 across all up-regulated fractions while median FC drifts between 1.39 and 1.63, exposing the median FC bias.

## Related tools

- **MAFFIN** (Integrated R package providing MedianNorm, MDFCNorm, and intragroup-variation evaluation used by this skill) — github.com/HuanLab/MAFFIN
- **MedianNorm** (MAFFIN function that computes median fold change normalization factors — the biased baseline estimator benchmarked in Figure 4A) — github.com/Waddlessss/MAFFIN
- **MDFCNorm** (MAFFIN kernel-density MDFC function run in parallel as the accurate comparator (1.5001 ± 0.003 vs true dilution factor 1.5)) — github.com/Waddlessss/MAFFIN
- **R** (Runtime language used to generate the simulated 5000-feature tables and call MAFFIN normalization functions)

## Examples

```
MedianNormedTable = MedianNorm(inputTable)
```

## Evaluation signals

- With true dilution factor k=1.5, MDFC estimates stay at 1.5001 ± 0.003 across up-regulated fractions 5%–45%, while median FC estimates span 1.39–1.63 (Figure 4A behavior)
- MedianNorm returns a named numeric normalization-factor vector with one factor per biological sample (SampleInCol orientation)
- figure4A_normalization_factors.csv has one column for MDFC and one for median FC with mean ± SD aggregated per up-regulated fraction
- RunEvaluation=TRUE yields intragroup variation (pooled RMAD/RSD); lower values confirm better sample normalization
- Input schema check: first row holds group labels, and real biological sample group names must not be blank, RT, QC, or SQC_###

## Limitations

- Median FC normalization factors are biased when the percentages of up- and down-regulated features are imbalanced (estimates ranged 1.39–1.63 against a true dilution factor of 1.5 in the simulation)
- Standalone MedianNorm performs no high-quality feature selection or serial-QC MS signal intensity correction; those biases are only addressed by the full MAFFINNorm three-module pipeline
- MAFFIN expects a feature intensity table with features in rows and samples in columns, with reserved group labels (blank, RT, QC, SQC_###) unavailable to biological samples

## Evidence
