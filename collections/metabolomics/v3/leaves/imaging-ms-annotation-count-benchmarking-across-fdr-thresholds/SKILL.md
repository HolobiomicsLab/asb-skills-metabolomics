---
name: imaging-ms-annotation-count-benchmarking-across-fdr-thresholds
description: Use when comparing two imaging-MS metabolite annotation engines on the
  same dataset set and you need per-dataset annotation counts at specified FDR cutoffs
  (e.g., 5% and 10%), summarized by sample-type or instrument-context medians.
when_to_use_negative:
- You only need to inspect or browse a single dataset's annotations interactively
  rather than benchmark two engines across a dataset collection
- Your annotations come from an engine or FDR estimation scheme other than METASPACE-ML
  vs. MSM, where the target/decoy FDR-based count comparison is not meaningful
- The dataset lacks both ML and MSM annotation runs at matched FDR levels, so paired
  per-dataset differences cannot be computed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: METASPACE-ML
  role: Machine-learning annotation engine whose FDR-controlled annotations are counted
    and benchmarked against MSM
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Rule-based MSM annotation engine providing the baseline annotation result
    tables; its python-client is used to retrieve annotations from the annotations
    database
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:ea752ff61919bdea5092a40d4065a5d2a82531b70b6daeef816f87fce5f8b601
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# imaging-ms-annotation-count-benchmarking-across-fdr-thresholds

## Summary

A benchmarking skill that quantifies how many more confident metabolite annotations the machine-learning METASPACE-ML engine delivers versus the rule-based MSM scoring, per dataset, at FDR thresholds of 10% and 5%, across animal, plant, Orbitrap, and FT-ICR testing contexts.

## When to use

Use when comparing two imaging-MS metabolite annotation engines on the same dataset set and you need per-dataset annotation counts at specified FDR cutoffs (e.g., 5% and 10%), summarized by sample-type or instrument-context medians.

## When NOT to use

- You only need to inspect or browse a single dataset's annotations interactively rather than benchmark two engines across a dataset collection
- Your annotations come from an engine or FDR estimation scheme other than METASPACE-ML vs. MSM, where the target/decoy FDR-based count comparison is not meaningful
- The dataset lacks both ML and MSM annotation runs at matched FDR levels, so paired per-dataset differences cannot be computed

## Inputs

- METASPACE-ML annotation result tables for animal and plant testing datasets (including Orbitrap and FT-ICR instrument contexts)
- MSM (rule-based METASPACE) annotation result tables for the same testing datasets
- FDR threshold specification (5% and 10%) per annotation

## Outputs

- Per-dataset annotation counts for METASPACE-ML and MSM at 5% and 10% FDR
- Per-group median annotation-count differences (METASPACE-ML minus MSM) at 10% FDR
- Per-group median log2 fold-change of annotation counts at 5% FDR
- annotation_comparison_stats.csv summary table of all recomputed statistics

## How to apply

Load the METASPACE-ML and MSM annotation result tables for each testing dataset (animal, plant, Orbitrap, FT-ICR) into pandas. For each dataset and each FDR threshold (5% and 10%), filter the annotation tables to confident (FDR-controlled) annotations and count annotations per dataset for both methods. Compute the per-dataset median difference in annotation counts (METASPACE-ML minus MSM) at 10% FDR, the median log2 fold-change of METASPACE-ML/MSM counts at 5% FDR, and the median annotation-count difference between Orbitrap and FT-ICR contexts. Assemble the recomputed statistics into a summary table (annotation_comparison_stats.csv) and accept them if they reproduce the reported benchmarks (~20 and ~70 extra median annotations at 10% FDR for animal and plant datasets; ~1.64x and ~1.80x median LFC at 5% FDR; ~70 extra median annotations for Orbitrap versus FT-ICR).

## Related tools

- **METASPACE-ML** (Machine-learning annotation engine whose FDR-controlled annotations are counted and benchmarked against MSM) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Rule-based MSM annotation engine providing the baseline annotation result tables; its python-client is used to retrieve annotations from the annotations database) — https://github.com/metaspace2020/metaspace

## Examples

```
import pandas as pd; ml, msm = pd.read_csv('metaspace_ml.csv'), pd.read_csv('msm.csv'); med = (ml.groupby('dataset')['n10'].median() - msm.groupby('dataset')['n10'].median())
```

## Evaluation signals

- Median METASPACE-ML minus MSM annotation-count difference at 10% FDR is approximately 20 (animal) and 70 (plant) annotations
- Median log2 fold-change of METASPACE-ML/MSM counts at 5% FDR is approximately 1.64x (animal) and 1.80x (plant)
- Median annotation-count difference for Orbitrap versus FT-ICR datasets is approximately 70 extra annotations
- annotation_comparison_stats.csv contains one row per dataset-group statistic with matching column schema (method, FDR level, group, median statistic)
- Counts at each FDR threshold are computed only from annotations passing the corresponding confident/FDR filter, so counts at 5% FDR are a subset of counts at 10% FDR per method

## Limitations

- MSM can capture annotations missed by METASPACE-ML due to MSM's sensitivity to its feature scores, particularly the rho_spatial score, so METASPACE-ML does not strictly dominate per dataset
- Training data selection is biased towards public datasets represented in METASPACE (from 47 labs), which may limit generalization of the comparison
- QTOF-based datasets are underrepresented in the model due to lack of publicly available data for statistically reliable evaluation

## Evidence

- [intro] ML outperforms MSM on annotations: "We demonstrate that METASPACE-ML outperforms the traditional MSM rule-based approach by delivering more annotations, especially at low FDR thresholds and for ions of low intensity."
- [abstract] FDR estimation and new scores: "METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation."
- [discussion] MSM captures annotations ML misses: "some annotations were captured by MSM but not by METASPACE-ML due to MSM’s sensitivity to its feature scores, particularly the rho_spatial score"
- [readme] Python client for annotations database: [evidence span withheld — anchor status: external_doc]
