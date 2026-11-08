---
name: instrument-context-stratified-comparison-orbitrap-fticr
description: Use when comparing annotation counts between METASPACE-ML and MSM on
  imaging mass spectrometry datasets grouped by instrument (Orbitrap, FT-ICR) or sample
  type (animal, plant) at fixed FDR cutoffs of 5% and 10%.
when_to_use_negative:
- When the comparison goal is annotation precision or target-decoy score distributions
  rather than annotation-count coverage differences
- When datasets do not have defined instrument context (Orbitrap/FT-ICR) or sample-context
  labels (animal/plant), since stratified medians cannot be computed
- When evaluating QTOF-based datasets, which the article reports as underrepresented
  in the model due to lack of publicly available data
tools:
- name: METASPACE-ML
  role: Machine-learning annotation engine whose FDR-controlled annotations are the
    treatment arm being counted per dataset and FDR threshold
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Platform hosting the MSM rule-based annotation engine and the annotations
    database from which result tables are retrieved for the baseline arm
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:4157be21d35dfdd101957b8bcd77aff64cb8de1873082a3b0780e72e91e06923
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Instrument-context-stratified comparison of METASPACE-ML vs MSM annotation coverage (Orbitrap vs FT-ICR)

## Summary

Recompute and benchmark annotation-coverage gains of machine-learning METASPACE-ML over the rule-based Metabolite Signal Matching (MSM) scoring engine, stratified by sample context (animal/plant) and instrument context (Orbitrap vs FT-ICR), at 5% and 10% FDR thresholds.

## When to use

Use when comparing annotation counts between METASPACE-ML and MSM on imaging mass spectrometry datasets grouped by instrument (Orbitrap, FT-ICR) or sample type (animal, plant) at fixed FDR cutoffs of 5% and 10%.

## When NOT to use

- When the comparison goal is annotation precision or target-decoy score distributions rather than annotation-count coverage differences
- When datasets do not have defined instrument context (Orbitrap/FT-ICR) or sample-context labels (animal/plant), since stratified medians cannot be computed
- When evaluating QTOF-based datasets, which the article reports as underrepresented in the model due to lack of publicly available data

## Inputs

- METASPACE-ML annotation result tables for animal and plant testing datasets
- MSM (rule-based) annotation result tables for the same testing datasets
- Instrument-context labels (Orbitrap vs FT-ICR) per dataset
- FDR threshold definitions (5% and 10%) used to define confident annotations

## Outputs

- annotation_comparison_stats.csv with per-group median annotation-count differences and median log2 fold-changes
- Per-dataset annotation counts for METASPACE-ML and MSM at 5% and 10% FDR

## How to apply

Load the METASPACE-ML and MSM annotation result tables for the testing datasets (animal, plant, Orbitrap, FT-ICR contexts) with pandas. For each dataset and each FDR threshold (5% and 10%), filter to confident annotations and count annotations per dataset for both methods. Compute per-dataset median differences (METASPACE-ML minus MSM) at 10% FDR for animal and plant groups, and per-dataset median log2 fold-changes (ML/MSM ratio) at 5% FDR. Additionally compute the median difference in annotation counts between Orbitrap and FT-ICR datasets at the reported FDR level. Accept the recomputation if it reproduces the reported benchmarks: ~20 and ~70 extra median annotations at 10% FDR (animal, plant), ~1.64x and ~1.80x median LFC at 5% FDR, and ~70 extra median annotations for Orbitrap versus FT-ICR. Save all recomputed statistics to annotation_comparison_stats.csv.

## Related tools

- **METASPACE-ML** (Machine-learning annotation engine whose FDR-controlled annotations are the treatment arm being counted per dataset and FDR threshold) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Platform hosting the MSM rule-based annotation engine and the annotations database from which result tables are retrieved for the baseline arm) — https://github.com/metaspace2020/metaspace

## Examples

```
counts = ml[ml.fdr<=0.10].groupby('dataset_id').size() - msm[msm.fdr<=0.10].groupby('dataset_id').size(); stats = counts.groupby('context').median()
```

## Evaluation signals

- Median annotation-count difference at 10% FDR is ~20 for animal and ~70 for plant testing datasets (METASPACE-ML minus MSM)
- Median log2 fold-change at 5% FDR reproduces approximately 1.64x (animal) and 1.80x (plant)
- Median Orbitrap-versus-FT-ICR annotation-count difference at the reported FDR level is approximately 70 annotations
- Output file annotation_comparison_stats.csv contains one row per stratification group with difference and LFC columns populated for both FDR thresholds
- A non-trivial minority of datasets show MSM-only annotations, consistent with the reported MSM sensitivity to the rho_spatial feature score

## Limitations

- Some annotations are captured by MSM but not by METASPACE-ML due to MSM's sensitivity to its feature scores, particularly the rho_spatial score, so gains are not uniformly positive per dataset
- QTOF-based datasets are underrepresented in the model due to lack of publicly available data needed for statistically reliable model evaluation
- Training data selection is biased towards public datasets represented in METASPACE (from 47 labs), which may limit generalization of the stratified benchmarks

## Evidence

- [results] At the default FDR 10%, METASPACE-ML identified approximately 20 and 70 more annotations on average: [evidence span withheld — anchor status: not_found]
- [intro] We demonstrate that METASPACE-ML outperforms the traditional MSM rule-based approach: "We demonstrate that METASPACE-ML outperforms the traditional MSM rule-based approach by delivering more annotations, especially at low FDR thresholds and for ions of low intensity."
- [discussion] MSM's sensitivity to rho_spatial score causes MSM-only annotations: "some annotations were captured by MSM but not by METASPACE-ML due to MSM’s sensitivity to its feature scores, particularly the rho_spatial score"
- [readme] METASPACE platform hosts annotation engine and knowledgebase: "The METASPACE platform hosts an engine for
 metabolite annotation of imaging mass spectrometry data as well as a
 spatial metabolite knowledgebase"
