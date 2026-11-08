---
name: pathway-ranking-robustness-benchmarking
description: Use when evaluating how stable pathway rankings from a metabolomics tool
  remain when a large fraction of peak features is randomly removed, and you need
  a quantitative robustness metric (F1) to compare competing ranking methods.
when_to_use_negative:
- The dataset is complete (no missing peaks) and the goal is simply to rank pathways,
  not to compare method robustness — run the ranking method directly instead.
- Peak features lack compound annotations (unannotated peaks are excluded from pathway
  analysis), so subsampling of annotated features would not reflect pathway coverage.
- The pathway database or annotations cannot be mapped to a supported ID space (KEGG/ChEBI),
  as ORA/GSEA/PLAGE in PALS require compound-level ID matching.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PALS
  role: Provides the PLAGE/mPLAGE, ORA and GSEA pathway ranking methods and the DataSource
    loading of intensity and annotation dataframes used for benchmarking
  repo: https://github.com/glasgowcompbio/PALS
- name: pandas
  role: Passes intensity and annotation DataFrames directly to PALS when used programmatically
    for the benchmark loop
- name: SciPy
  role: Standardises the intensity matrix (zero mean, unit variance) via its preprocessing
    module before ranking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:60f1858784137f4ecb22de5f7d23ac0cdb3c9079539b7afcb1b60909807661d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# Pathway-ranking robustness benchmarking

## Summary

Benchmarks pathway-ranking methods (mPLAGE/PLAGE, ORA, GSEA) under data loss by comparing top-ranked pathway sets from subsampled data against full-data reference rankings, using precision/recall/F1 to quantify which method is most robust to missing features.

## When to use

Use when evaluating how stable pathway rankings from a metabolomics tool remain when a large fraction of peak features is randomly removed, and you need a quantitative robustness metric (F1) to compare competing ranking methods.

## When NOT to use

- The dataset is complete (no missing peaks) and the goal is simply to rank pathways, not to compare method robustness — run the ranking method directly instead.
- Peak features lack compound annotations (unannotated peaks are excluded from pathway analysis), so subsampling of annotated features would not reflect pathway coverage.
- The pathway database or annotations cannot be mapped to a supported ID space (KEGG/ChEBI), as ORA/GSEA/PLAGE in PALS require compound-level ID matching.

## Inputs

- Intensity CSV matrix (peak features × samples, with group/sample-factor row)
- Annotation CSV mapping peak IDs to compound database IDs (KEGG or ChEBI)
- Experimental design (comparisons, e.g. case/control pairs)
- Pathway database choice (e.g. PiMP_KEGG, ChEBI, COMPOUND)

## Outputs

- Aggregated precision/recall/F1 benchmark CSV per method per dataset (e.g. missing_feature_benchmark_f1.csv)
- Per-repetition subsampled pathway rankings for each method

## How to apply

First establish a full-data reference: run the ranking method of interest (e.g. PALS PLAGE with PiMP_KEGG database on HAT Plasma/CSF intensity matrices and KEGG/ChEBI annotation CSVs) on the complete data, applying the tool's default imputation (min_replace 5000), log-2 transformation, and Scipy zero-mean/unit-variance standardisation. Then, for each of many repetitions (the article used 500), randomly subsample a fixed fraction of the annotated peak features (e.g. retain only 20%, i.e. 80% missing) and rerun all competing methods (mPLAGE, ORA, GSEA) on the subsampled intensity and annotation matrices with the same experimental design (e.g. Stage_1/Control and Stage_2/Control comparisons). For each repetition, compute precision, recall, and F1 by comparing the subsampled top-ranked pathways against the full-data reference ranking. Aggregate scores per method per dataset and rank methods by robustness; the article found mPLAGE most robust (F1 ≈ 0.48 Plasma, 0.60 CSF) versus ORA (≈0.20/0.50) and GSEA (≈0.08/0.16).

## Related tools

- **PALS** (Provides the PLAGE/mPLAGE, ORA and GSEA pathway ranking methods and the DataSource loading of intensity and annotation dataframes used for benchmarking) — https://github.com/glasgowcompbio/PALS
- **pandas** (Passes intensity and annotation DataFrames directly to PALS when used programmatically for the benchmark loop)
- **SciPy** (Standardises the intensity matrix (zero mean, unit variance) via its preprocessing module before ranking)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control
```

## Evaluation signals

- Output CSV contains precision, recall, and F1 columns for all benchmarked methods on both datasets (Plasma and CSF).
- mPLAGE F1 reproduces the reported robustness advantage over ORA and GSEA (≈0.48 vs ≈0.20 vs ≈0.08 on Plasma; ≈0.60 vs ≈0.50 vs ≈0.16 on CSF).
- Every repetition uses the same experimental design and database (e.g. PiMP_KEGG, Stage_1/Control and Stage_2/Control) as the full-data reference.
- Subsampled runs retain a fixed feature fraction (e.g. 20% of annotated peaks) across repetitions, verified by consistent feature counts.
- Full-data reference rankings are computed with the same imputation (min_replace 5000), log-2 transformation, and Scipy standardisation applied in subsampled runs.

## Limitations

- Robustness F1 depends on the subsampling fraction and repetition count; results at a different missingness level may not transfer.
- Peaks without annotations cannot be used for pathway analysis, so the effective pathway coverage shrinks as features are removed.
- Multiple peak IDs may map to multiple compound IDs and vice versa, complicating pathway membership for ORA/GSEA enrichment counts.
- Reported F1 values are specific to the HAT Plasma/CSF datasets and the PiMP_KEGG database used in the paper.

## Evidence

- [abstract] mPLAGE is more robust to missing features and noisy data than ORA and GSEA: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given and reveals that mPLAGE is more robust to missing"
- [other] Repetition-based F1 vs full-data reference: [evidence span withheld — anchor status: not_found]
- [methods] Default imputation and preprocessing in PALS: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance"
- [other] Subsampling fraction (80% missing): [evidence span withheld — anchor status: not_found]
- [readme] mPLAGE vs ORA vs GSEA robustness claim (README): "The results are found to be more robust to noise and missing peaks compared to the alternatives (ORA, GSEA)."
- [readme] Imputation parameters from README: "Data imputation is performed to the intensity matrix when it is loaded: if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [readme] Annotation requirement (peaks must map to compound IDs): "users also provide a list of compound annotations assigned to peak features (peaks that do not have annotations will not be used for pathway analysis)"
