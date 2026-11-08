---
name: precision-recall-f1-evaluation
description: Use when comparing robustness of pathway-ranking methods on metabolomics
  intensity matrices where peaks may be missing or noisy — specifically when you have
  a full-data reference ranking and subsampled reruns (e.
when_to_use_negative:
- Input is already a subsampled feature table with no full-data reference ranking
  available for comparison — precision/recall/F1 cannot be computed against a baseline.
- Peak features lack compound annotations (KEGG or ChEBI IDs), as unannotated peaks
  are excluded from pathway analysis and cannot contribute to ranking comparison.
- The goal is de novo pathway discovery rather than benchmarking robustness of existing
  ranking methods under feature loss.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: PALS
  role: Executes the pathway-ranking methods (PLAGE, ORA, GSEA) on full and subsampled
    HAT intensity matrices, producing the rankings that are scored by precision/recall/F1.
  repo: github.com/glasgowcompbio/PALS
- name: pandas
  role: Loads and manipulates the HAT intensity and annotation DataFrames used for
    subsampling and reference-ranking construction.
- name: SciPy
  role: Standardises the log-2-transformed intensity matrix (zero mean, unit variance)
    before pathway decomposition runs.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:1de73437f02091d2fc7af53fa3de8fc19ac61c21e052a208f06b1f23d52086b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# precision-recall-f1-evaluation

## Summary

Benchmark pathway-ranking methods (PLAGE/mPLAGE, ORA, GSEA) on metabolomics peak data by repeatedly subsampling annotated features and scoring each method's top-ranked pathways against a full-data reference ranking using precision, recall, and F1. This skill operationalises the paper's claim that mPLAGE is more robust to missing peaks than ORA or GSEA on real HAT Plasma and CSF datasets.

## When to use

Use when comparing robustness of pathway-ranking methods on metabolomics intensity matrices where peaks may be missing or noisy — specifically when you have a full-data reference ranking and subsampled reruns (e.g. 80% of features removed) that need precision/recall/F1 scoring against that reference.

## When NOT to use

- Input is already a subsampled feature table with no full-data reference ranking available for comparison — precision/recall/F1 cannot be computed against a baseline.
- Peak features lack compound annotations (KEGG or ChEBI IDs), as unannotated peaks are excluded from pathway analysis and cannot contribute to ranking comparison.
- The goal is de novo pathway discovery rather than benchmarking robustness of existing ranking methods under feature loss.

## Inputs

- Full-data HAT Plasma and CSF processed intensity matrices (CSV, rows = peak features with group line indicating sample factor)
- Annotation CSV mapping peak row_id to KEGG or ChEBI entity_id (e.g. C00148)
- Experimental design dictionary with comparisons (case/control) and groups
- Subsampled feature indices (20% of annotated peaks) per repetition

## Outputs

- missing_feature_benchmark_f1.csv containing precision, recall, and F1 columns for PLAGE, ORA, and GSEA on both HAT datasets
- Aggregated 500-repetition precision/recall/F1 scores per method per dataset

## How to apply

First establish a full-data reference ranking by running PALS with PLAGE on the complete HAT Plasma and CSF intensity matrices (PiMP_KEGG database, comparisons Stage_1/Control and Stage_2/Control, min_replace 5000 imputation, log-2 transformation and scipy standardisation). For each of 500 repetitions, randomly retain only 20% of the annotated peak features and rerun mPLAGE, ORA, and GSEA on the subsampled data using the same experimental design. For each repetition and method, compute precision, recall, and F1 by comparing the subsampled top-ranked pathways against the full-data reference ranking (peaks without annotations are excluded from pathway analysis). Aggregate the 500-repetition precision/recall/F1 scores per method per dataset and verify that mPLAGE achieves the higher robustness (F1 ≈ 0.48 Plasma, 0.60 CSF) relative to ORA (≈ 0.20/0.50) and GSEA (≈ 0.08/0.16), then write the aggregated benchmark metrics to missing_feature_benchmark_f1.csv.

## Related tools

- **PALS** (Executes the pathway-ranking methods (PLAGE, ORA, GSEA) on full and subsampled HAT intensity matrices, producing the rankings that are scored by precision/recall/F1.) — github.com/glasgowcompbio/PALS
- **pandas** (Loads and manipulates the HAT intensity and annotation DataFrames used for subsampling and reference-ranking construction.)
- **SciPy** (Standardises the log-2-transformed intensity matrix (zero mean, unit variance) before pathway decomposition runs.)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- Output CSV contains precision, recall, and F1 columns for all three methods (PLAGE, ORA, GSEA) on both HAT Plasma and CSF datasets.
- mPLAGE F1 on Plasma is ≈ 0.48 (vs ≈ 0.20 for ORA and ≈ 0.08 for GSEA) across 500 repetitions.
- mPLAGE F1 on CSF is ≈ 0.60 (vs ≈ 0.50 for ORA and ≈ 0.16 for GSEA) across 500 repetitions.
- Precision and recall values are computed against a full-data reference ranking established with min_replace 5000 imputation and log-2/scipy standardisation.
- Every repetition uses the same 20%-retention subsampling scheme, so scores are comparable across methods and datasets.

## Limitations

- Results depend on the choice of subsampling fraction (here 20% retained / 80% missing); other missingness rates may change the relative robustness ranking.
- Only metabolomics peak data with compound annotations are covered; peaks without annotations are dropped and cannot contribute to precision/recall.
- The benchmark relies on a single full-data reference ranking; disagreement in the reference (e.g. unstable PLAGE ranking on small datasets) propagates into all F1 estimates.

## Evidence

- [abstract] mPLAGE robustness finding: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given and reveals that mPLAGE is more robust to missing"
- [readme] full-data reference ranking run: "python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control"
- [readme] imputation and standardisation pre-processing: "Data imputation is performed to the intensity matrix when it is loaded: if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [readme] annotation requirement for pathway analysis: "In addition, users also provide a list of compound annotations assigned to peak features (peaks that do not have annotations will not be used for pathway analysis)."
