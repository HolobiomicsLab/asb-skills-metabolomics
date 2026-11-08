---
name: false-discovery-rate-threshold-filtering
description: Use when comparing annotation yield between FDR-controlled metabolite
  annotation methods (e.g., METASPACE-ML vs MSM) across imaging MS datasets, or when
  confident annotations must be selected at a specified FDR cutoff before counting
  or fold-change statistics.
when_to_use_negative:
- The input is raw imaging MS data (e.g., imzML) that has not yet been annotated by
  METASPACE-ML or MSM — annotation must run first before FDR filtering applies
- The goal is to evaluate annotation precision against ground-truth identifications
  rather than to compare annotation coverage between FDR-controlled methods at fixed
  thresholds
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: METASPACE-ML
  role: Machine-learning annotation engine providing new scores and computationally-efficient
    FDR estimation; its GBDT model separates target and decoy ions for FDR-controlled
    metabolite annotation in specific imaging MS contexts
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Baseline rule-based engine whose Metabolite Signal Matching (MSM) scoring
    provides the comparison annotation counts filtered at the same FDR thresholds
  repo: https://github.com/metaspace2020/metaspace
- name: Lithops
  role: Serverless cloud computing framework used for efficient implementation and
    reprocessing of public METASPACE datasets at scale
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:b1ae108889c8a455c0e7be70ccfd6fa61969aa849cba38667ebaef2904ae1dea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - Lithops
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# false-discovery-rate-threshold-filtering

## Summary

Filters metabolite annotations from METASPACE-ML and rule-based MSM results at target False Discovery Rate thresholds (5% and 10%) to count confident annotations per dataset and quantify annotation-coverage gains of the machine-learning model over its rule-based predecessor in imaging mass spectrometry.

## When to use

Use when comparing annotation yield between FDR-controlled metabolite annotation methods (e.g., METASPACE-ML vs MSM) across imaging MS datasets, or when confident annotations must be selected at a specified FDR cutoff before counting or fold-change statistics.

## When NOT to use

- The input is raw imaging MS data (e.g., imzML) that has not yet been annotated by METASPACE-ML or MSM — annotation must run first before FDR filtering applies
- The goal is to evaluate annotation precision against ground-truth identifications rather than to compare annotation coverage between FDR-controlled methods at fixed thresholds

## Inputs

- METASPACE-ML annotation result tables for testing datasets (animal, plant, Orbitrap, FTICR contexts)
- MSM (rule-based METASPACE) annotation result tables for the same testing datasets
- FDR threshold specifications (5% and 10%)

## Outputs

- Per-dataset confident annotation counts at 5% and 10% FDR for METASPACE-ML and MSM
- Median annotation-count differences (METASPACE-ML minus MSM) at 10% FDR for animal and plant testing groups
- Median log2 fold-changes (ML/MSM annotation count ratio) at 5% FDR for animal and plant testing groups
- Summary CSV (annotation_comparison_stats.csv)

## How to apply

Load the METASPACE-ML and MSM annotation result tables for the testing datasets (animal, plant, Orbitrap, FTICR contexts) with pandas. For each dataset and each FDR threshold (5% and 10%), filter the annotation tables to confident annotations passing the FDR cutoff and count annotations per dataset for both methods. Compute per-dataset median differences in annotation counts (METASPACE-ML minus MSM) at 10% FDR — the default level — and median log2 fold-changes of the ML/MSM count ratio at 5% FDR, separately for animal and plant testing groups. The FDR estimation is computationally efficient in METASPACE-ML because it uses new scores and a machine-learning GBDT ranking model that separates target and decoy ions, replacing MSM's rule-based scoring that assigns equal weights to features. Assemble the recomputed medians into a summary table (annotation_comparison_stats.csv) and compare them against the reported benchmark values.

## Related tools

- **METASPACE-ML** (Machine-learning annotation engine providing new scores and computationally-efficient FDR estimation; its GBDT model separates target and decoy ions for FDR-controlled metabolite annotation in specific imaging MS contexts) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Baseline rule-based engine whose Metabolite Signal Matching (MSM) scoring provides the comparison annotation counts filtered at the same FDR thresholds) — https://github.com/metaspace2020/metaspace
- **Lithops** (Serverless cloud computing framework used for efficient implementation and reprocessing of public METASPACE datasets at scale)

## Examples

```
import pandas as pd; df = pd.read_csv('annotations.csv'); conf = df[(df.method=='METASPACE-ML') & (df.fdr<=0.10)]; counts = conf.groupby('dataset_id').size()
```

## Evaluation signals

- At default 10% FDR, recomputed medians reproduce approximately 20 and 70 more annotations on average for METASPACE-ML vs MSM in animal and plant testing datasets, respectively
- At 10% FDR, at least 50 (animal) and 100 (plant) more annotations in 50% of datasets
- At 5% FDR, recomputed median log-fold-changes reproduce approximately 1.64x (animal) and 1.80x (plant) for METASPACE-ML vs MSM
- Median annotation-count difference between Orbitrap and FTICR datasets at the reported FDR level reproduces approximately 70 extra median annotations
- All recomputed statistics assemble into a valid annotation_comparison_stats.csv with one row per dataset-group/statistic combination

## Limitations

- Some annotations are captured by MSM but not by METASPACE-ML due to MSM's sensitivity to its feature scores, particularly the rho_spatial score, so count differences can be negative for individual datasets
- Decoy ions show heterogeneity in similarity to target ions; the most reliable decoy generation method for FDR estimation needs further investigation
- QTOF-based datasets are underrepresented in the model due to lack of publicly available data for statistically reliable model evaluation, limiting generalization of FDR-filtered counts to that instrument class

## Evidence

- [results] At the default FDR 10%, METASPACE-ML identified approximately 20 and 70 more annotations on average (and at least 50 and 100 more annotations in 50% of datasets) for animal and plant-based testing datasets, respectively, versus MSM.: [evidence span withheld — anchor status: not_found]
- [abstract] METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation.: "incorporates new scores and computationally-efficient False Discovery Rate estimation"
- [intro] a limitation of METASPACE is its rule-based scoring system, namely Metabolite Signal Matching (MSM), which assigns equal weights to the features and lacks adaptability to data variations: "Metabolite Signal Matching (MSM), which assigns equal weights to the features and lacks adaptability to data variations"
- [intro] We demonstrate that METASPACE-ML outperforms the traditional MSM rule-based approach by delivering more annotations, especially at low FDR thresholds and for ions of low intensity.: "delivering more annotations, especially at low FDR thresholds and for ions of low intensity"
- [discussion] some annotations were captured by MSM but not by METASPACE-ML due to MSM’s sensitivity to its feature scores, particularly the rho_spatial score: "captured by MSM but not by METASPACE-ML due to MSM’s sensitivity to its feature scores, particularly the rho_spatial score"
