---
name: kernel-density-fold-change-estimation
description: Use when a feature intensity table (features in rows, samples in columns)
  needs normalization factors and you want a density-based fold-change estimator robust
  to imbalanced percentages of up- and down-regulated features.
when_to_use_negative:
- When serial QC samples are available for MS signal intensity correction and full
  high-quality feature selection — use the MAFFINNorm main function pipeline instead
  of standalone MDFCNorm on an uncorrected table
- When a large fraction of features are unchanged and the up/down-regulated balance
  is not a concern only if comparing estimators — median FC (MedianNorm) is a simpler
  alternative, though the article shows it is biased (1.39–1.63 vs true k=1.5) under
  imbalance
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MAFFIN
  role: R package providing MDFCNorm for kernel-density maximal density fold change
    normalization
  repo: https://github.com/HuanLab/MAFFIN
- name: MDFCNorm
  role: Function computing the MDFC normalization factors with parameters (FeatureTable,
    IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation, bwOpt)
  repo: https://github.com/HuanLab/MAFFIN
- name: NormByFactors
  role: Applies precomputed normalization factors back to the feature table
  repo: https://github.com/HuanLab/MAFFIN
- name: EvaPRMAD
  role: Evaluates normalization by pooled RMAD intragroup variation
  repo: https://github.com/HuanLab/MAFFIN
- name: MedianNorm
  role: Median fold-change baseline estimator used for comparison in the benchmark
  repo: https://github.com/HuanLab/MAFFIN
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac355
    title: 'MAFFIN: metabolomics sample normalization using maximal density fold change
      with high-quality metabolic features and corrected signal intensities'
  source_content_hash: sha256:a29c69f754fd2a6791e37538342f57f955532c9b06335fbb847bbb9d6aef0666
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
  - EvaPRMAD
  - MAFFIN
  - MDFCNorm
  - MedianNorm
  - NormByFactors
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# kernel-density-fold-change-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates a metabolomics sample normalization factor by fitting a Gaussian kernel density to per-feature log2 fold changes against a reference sample and taking the fold change at maximal density (MDFC), which remains accurate even when up-/down-regulated features are imbalanced where the median FC is biased.

## When to use

Use when a feature intensity table (features in rows, samples in columns) needs normalization factors and you want a density-based fold-change estimator robust to imbalanced percentages of up- and down-regulated features.

## When NOT to use

- When serial QC samples are available for MS signal intensity correction and full high-quality feature selection — use the MAFFINNorm main function pipeline instead of standalone MDFCNorm on an uncorrected table
- When a large fraction of features are unchanged and the up/down-regulated balance is not a concern only if comparing estimators — median FC (MedianNorm) is a simpler alternative, though the article shows it is biased (1.39–1.63 vs true k=1.5) under imbalance

## Inputs

- Feature intensity table (dataframe) with features in rows and samples in columns; column names are sample names and the first row holds group labels (blank, RT, QC, SQC_###)
- IntThreshold detection criterion value
- bwOpt bandwidth-selection option (SJ rule for two-sample tables)

## Outputs

- Named numeric vector of MDFC normalization factors per sample (written to mdfc_norm_factors.csv in the reconstruction workflow)
- Optional pooled RMAD normalization-evaluation output

## How to apply

Load the feature intensity table in MAFFIN's expected format (features in rows, sample names as column names, group labels in the first row) and identify the reference sample as the one with the most detected features. Compute per-feature log2 fold changes of every sample against the reference, applying the IntThreshold detection criterion as in MDFCNorm. Estimate the fold-change density with a Gaussian kernel density estimator, using the Sheather-Jones (SJ) bandwidth rule when the table contains only two samples (the bwOpt parameter of MDFCNorm). Take the fold change at the maximal density as that sample's MDFC normalization factor, assemble the named numeric vector, and optionally evaluate with pooled RMAD via EvaPRMAD. As a self-test, a synthetic two-sample table with a true dilution factor of 1.5 should recover a normalization factor consistent with the dilution relationship (MDFC gave 1.5001 ± 0.003 in the benchmark while median FC spanned 1.39–1.63).

## Related tools

- **MAFFIN** (R package providing MDFCNorm for kernel-density maximal density fold change normalization) — https://github.com/HuanLab/MAFFIN
- **MDFCNorm** (Function computing the MDFC normalization factors with parameters (FeatureTable, IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation, bwOpt)) — https://github.com/HuanLab/MAFFIN
- **NormByFactors** (Applies precomputed normalization factors back to the feature table) — https://github.com/HuanLab/MAFFIN
- **EvaPRMAD** (Evaluates normalization by pooled RMAD intragroup variation) — https://github.com/HuanLab/MAFFIN
- **MedianNorm** (Median fold-change baseline estimator used for comparison in the benchmark) — https://github.com/HuanLab/MAFFIN

## Examples

```
normFactors = MDFCNorm(FeatureTable, IntThreshold, SampleInCol=TRUE, output=FALSE, OutputNormFactors=TRUE, RunEvaluation=TRUE, bwOpt='SJ')
```

## Evaluation signals

- In a two-sample simulation with 5000 features, 50% unchanged, up-regulated fractions swept 5%–45%, and true dilution factor k=1.5, MDFC estimates should stay near 1.5001 ± 0.003 while median FC spans 1.39–1.63 (Figure 4A behavior)
- The output is a named numeric vector of normalization factors whose names match the sample columns of the input FeatureTable
- Synthetic self-test: a two-sample table with known dilution factor 1.5 recovers a normalization factor matching the expected dilution relationship
- Pooled RMAD of the normalized table (EvaPRMAD) should be lower than the original (pre-normalization) RMAD, since lower intragroup variation means better normalization

## Limitations

- The SJ bandwidth rule (bwOpt) applies specifically when the table contains only two samples; behavior with other bandwidth choices is not characterized here
- Fold-change accuracy depends on the IntThreshold detection criterion and, in the full MAFFIN workflow, on prior high-quality feature selection and MS signal intensity correction with serial QC samples; abiotic or poorly quantitating features bias the density estimate
- MAFFIN's normalization evaluation relies on group labels; groups of real biological samples must not use reserved labels (blank, RT, QC, SQC_###)

## Evidence

- [abstract] MAFFIN calculates normalization factors using maximal density fold change (MDFC) computed by a kernel density-based approach.: "MAFFIN calculates normalization factors using maximal density fold change (MDFC) computed by a kernel density-based approach."
- [methods] MDFCNorm signature: "MDFCNorm(FeatureTable, IntThreshold, SampleInCol, output, OutputNormFactors, RunEvaluation, bwOpt)"
- [results] MDFC stable under imbalance: [evidence span withheld — anchor status: not_found]
- [methods] Intragroup variation evaluation: "MAFFIN package uses intragroup variation to evaluate the normalization results. The lower intragroup variation means better sample normalization."
