---
name: f1-evaluation-of-pathway-ranking
description: Use when comparing pathway ranking methods on metabolomics intensity
  matrices and you need quantitative robustness evidence (F1, p-value spread) under
  synthetic missing-feature perturbations rather than a single qualitative run.
when_to_use_negative:
- Input has no annotated peaks (peaks without compound annotations are excluded from
  pathway analysis, so robustness scoring would be meaningless)
- You need pathway rankings on a single unperturbed dataset rather than a robustness
  comparison across missing-feature levels
- The pathway database contains no ground-truth active pathways, so F1 against known
  true pathways cannot be computed
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3820
tools:
- name: PALS
  role: Python library providing pals.PLAGE, pals.ORA, pals.GSEA classes and the DataSource
    API used to run all three ranking methods on each perturbed replicate
  repo: https://github.com/glasgowcompbio/PALS
- name: pandas
  role: Holds int_df and annotation_df DataFrames passed directly to DataSource and
    aggregates benchmark statistics
- name: SciPy
  role: Preprocessing module used to standardise the intensity matrix to zero mean
    and unit variance before scoring
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:3bfa081daa8dcb871beb8a75c86f61cf764f2eae7f0f4228b64240c21de1b2ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3820
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# f1-evaluation-of-pathway-ranking

## Summary

Benchmark metabolite-set ranking methods (mPLAGE, ORA, GSEA) under controlled feature loss by perturbing annotated peak features and scoring each method with mean/variance of pathway p-values plus F1 scores against known true pathways, demonstrating mPLAGE's superior robustness.

## When to use

Use when comparing pathway ranking methods on metabolomics intensity matrices and you need quantitative robustness evidence (F1, p-value spread) under synthetic missing-feature perturbations rather than a single qualitative run.

## When NOT to use

- Input has no annotated peaks (peaks without compound annotations are excluded from pathway analysis, so robustness scoring would be meaningless)
- You need pathway rankings on a single unperturbed dataset rather than a robustness comparison across missing-feature levels
- The pathway database contains no ground-truth active pathways, so F1 against known true pathways cannot be computed

## Inputs

- Peak intensity matrix CSV (rows = peak features, columns = samples, with group/factor information)
- Annotation CSV mapping peak IDs to KEGG/ChEBI compound IDs
- Known set of true (active) pathways used as F1 ground truth
- Experimental design dictionary specifying case/control comparisons (e.g. Stage_1/Control, Stage_2/Control)
- List of missingness fractions (20%, 40%, 60%, 80%) and decoy-set level (e.g. 100%)

## Outputs

- Per-method/per-missingness table of mean p-value, p-value variance, and F1 score (e.g. missing_feature_benchmark_results.csv)
- Per-replicate pathway p-value rankings from pals.PLAGE, pals.ORA, and pals.GSEA

## How to apply

Load the peak intensity matrix and annotation CSVs (e.g. notebooks/test_data/HAT/int_df.csv and annotation_df.csv) through PALS, applying data imputation (all-zero factors replaced by --min_replace 5000, partially zero factors by the mean of non-zero samples), log-2 transformation, and SciPy zero-mean/unit-variance standardisation. Configure the benchmark with the PiMP_KEGG database, comparisons such as Stage_1/Control and Stage_2/Control, and fix the decoy metabolite-set level (e.g. 100%). For each missingness level (20%, 40%, 60%, 80%), randomly remove that fraction of annotated peak features and repeat 500 times with distinct seeds, at each replicate running pals.PLAGE, pals.ORA, and pals.GSEA via the DataSource API to collect pathway p-values. Aggregate per method and missingness level the mean and variance of p-values, and compute F1 scores against the known true pathways at the fixed decoy level. Decision rule: accept the benchmark when PLAGE (mPLAGE) shows consistently lower mean p-values, smaller p-value variance, and the highest F1 as missingness increases, while ORA and GSEA degrade; write the aggregated table (e.g. missing_feature_benchmark_results.csv).

## Related tools

- **PALS** (Python library providing pals.PLAGE, pals.ORA, pals.GSEA classes and the DataSource API used to run all three ranking methods on each perturbed replicate) — https://github.com/glasgowcompbio/PALS
- **pandas** (Holds int_df and annotation_df DataFrames passed directly to DataSource and aggregates benchmark statistics)
- **SciPy** (Preprocessing module used to standardise the intensity matrix to zero mean and unit variance before scoring)

## Examples

```
from pals.PLAGE import PLAGE; from pals.GSEA import GSEA; from pals.ORA import ORA; from pals.feature_extraction import DataSource; ds = DataSource(int_df, annotation_df, experimental_design, 'PiMP_KEGG', min_replace=5000); df = PLAGE(ds).get_pathway_df()
```

## Evaluation signals

- Across all missingness levels (20-80%), mPLAGE returns consistently lower mean pathway p-values than ORA and GSEA
- mPLAGE p-value variance stays smaller than ORA/GSEA variance at every missingness level
- mPLAGE attains the highest F1 score against the known true pathways at the fixed 100% decoy level
- Imputation check: all-zero factors replaced by min_replace=5000, partially zero factors replaced by factor-wise mean of non-zero samples, then log-2 transformed and standardised to zero mean/unit variance
- Each missingness condition is replicated with 500 distinct random seeds before aggregation

## Limitations

- Findings are grounded in synthetic missing-feature experiments with a fixed decoy metabolite-set level (100%); other decoy levels may shift the comparison
- Unannotated peaks are silently dropped from pathway analysis, so heavily unannotated datasets reduce effective pathway coverage (F_coverage = tot_ds_F / unq_pw_F)
- Robustness conclusions depend on random removal of features; results should be checked across many seeds (500 replicates used in the benchmark)

## Evidence

- [readme] The results are found to be more robust to noise and missing peaks compared to the alternatives (ORA, GSEA).: "The results are found to be more robust to noise and missing peaks compared to the alternatives (ORA, GSEA)."
- [abstract] comparison of mPLAGE with ORA and GSEA: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given"
- [readme] imputation and standardisation details: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance"
- [readme] unannotated peaks excluded: "users also provide a list of compound annotations assigned to peak features (peaks that do not have annotations will not be used for pathway analysis)"
