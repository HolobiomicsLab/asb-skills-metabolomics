---
name: decoy-metabolite-set-simulation
description: Use when you must quantify how robustly pathway-ranking methods (mPLAGE
  vs ORA vs GSEA) behave under missing metabolomics peaks — e.g. annotated peak-feature
  intensity matrices where you can inject controlled, randomised feature removal and
  score recovery against known true pathways.
when_to_use_negative:
- You need production pathway ranking on real (non-perturbed) data rather than a robustness
  benchmark with synthetic missingness
- The input peaks lack compound annotations entirely — PALS pathway analysis only
  uses annotated peaks, so controlled annotation-targeted removals cannot be simulated
- The comparison targets transcriptomics/proteomics Reactome queries (UniProt/ENSEMBL)
  where 'metabolite-set' decoy semantics and PiMP_KEGG mapping do not apply
edam_operation: http://edamontology.org/operation_2935
tools:
- name: PALS
  role: Provides pals.PLAGE, pals.ORA, pals.GSEA and the DataSource API used to score
    pathway p-values on each perturbed replicate
  repo: github.com/glasgowcompbio/PALS
- name: pandas
  role: Holds int_df and annotation_df DataFrames passed directly into the PALS DataSource
    for each simulated replicate
- name: SciPy
  role: Standardises the imputed, log-2 transformed intensity matrix to zero mean
    and unit variance before scoring
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:7ca575d7323b821e5a54ccd8d87d62adf229cf937686b78c0095d85d7bfffcf0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2935
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# decoy-metabolite-set-simulation

## Summary

Benchmarking skill for robustness evaluation of pathway-ranking methods: hold a decoy metabolite-set activity level fixed (100%) while randomly removing increasing fractions (20–80%) of annotated peak features, so mPLAGE, ORA, and GSEA p-values and F1 scores can be compared under controlled synthetic missingness.

## When to use

Use when you must quantify how robustly pathway-ranking methods (mPLAGE vs ORA vs GSEA) behave under missing metabolomics peaks — e.g. annotated peak-feature intensity matrices where you can inject controlled, randomised feature removal and score recovery against known true pathways.

## When NOT to use

- You need production pathway ranking on real (non-perturbed) data rather than a robustness benchmark with synthetic missingness
- The input peaks lack compound annotations entirely — PALS pathway analysis only uses annotated peaks, so controlled annotation-targeted removals cannot be simulated
- The comparison targets transcriptomics/proteomics Reactome queries (UniProt/ENSEMBL) where 'metabolite-set' decoy semantics and PiMP_KEGG mapping do not apply

## Inputs

- Peak-feature intensity matrix CSV (rows = peak features, columns = samples with group line, e.g. notebooks/test_data/HAT/int_df.csv)
- Peak-to-compound annotation matrix CSV mapping peak IDs to KEGG/ChEBI IDs (e.g. notebooks/test_data/HAT/annotation_df.csv)
- Pathway database selection (PiMP_KEGG) and experimental design with comparisons (Stage_1/Control, Stage_2/Control)
- Imputation threshold --min_replace (default 5000) and list of missingness fractions {20%, 40%, 60%, 80%} with 500 random seeds each

## Outputs

- Aggregated per-method/per-missingness table (missing_feature_benchmark_results.csv) with mean pathway p-value and p-value variance for PLAGE (mPLAGE), ORA, and GSEA
- Per-replicate pathway p-value rankings from pals.PLAGE, pals.ORA, pals.GSEA
- F1 scores against known true pathways at the fixed 100% decoy level

## How to apply

Load the intensity matrix and annotation file (e.g. notebooks/test_data/HAT/int_df.csv and annotation_df.csv) with PALS data imputation (all-zero factors replaced by --min_replace 5000, partially zero factors by the mean of non-zero samples in that factor), then log-2 transform and SciPy-standardise to zero mean and unit variance. Configure the benchmark with a pathway database (here PiMP_KEGG) and the case/control comparisons of interest (Stage_1/Control, Stage_2/Control), fixing the decoy metabolite-set level at 100%. For each missingness level in {20%, 40%, 60%, 80%}, randomly remove that fraction of annotated peak features and repeat the perturbation 500 times with distinct random seeds. At each replicate instantiate pals.PLAGE, pals.ORA, and pals.GSEA on a pals.feature_extraction.DataSource and collect pathway p-values; unannotated peaks will not enter pathway analysis, so removals target only annotated features. Aggregate per method and missingness level the mean p-value, the variance of p-values, and F1 scores against the known true pathways. Accept the benchmark when mPLAGE shows consistently lower mean p-values, smaller p-value variance, and the highest F1 as missingness increases, while ORA and GSEA degrade.

## Related tools

- **PALS** (Provides pals.PLAGE, pals.ORA, pals.GSEA and the DataSource API used to score pathway p-values on each perturbed replicate) — github.com/glasgowcompbio/PALS
- **pandas** (Holds int_df and annotation_df DataFrames passed directly into the PALS DataSource for each simulated replicate)
- **SciPy** (Standardises the imputed, log-2 transformed intensity matrix to zero mean and unit variance before scoring)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- Across increasing missingness (20→80%), PLAGE mean p-values stay consistently lower than ORA and GSEA at every level
- PLAGE p-value variance is smaller than ORA and GSEA variance at each missingness level
- PLAGE attains the highest F1 against the known true pathways at the fixed 100% decoy level
- Each aggregated statistic is stable over the 500 distinct-seed replicates per missingness level
- Preprocessing invariant: after imputation and log-2 transform, the standardised intensity matrix has zero mean and unit variance across samples

## Limitations

- Unannotated peaks are excluded — 'peaks that do not have annotations will not be used for pathway analysis' — so the simulation only perturbs the annotated feature subset
- Results depend on the PiMP_KEGG database export and the HAT example dataset; other databases or datasets may shift the robustness comparison
- Random (missing-completely-at-random style) feature removal may not reproduce real metabolomics missingness, which is often systematic or left-censored below detection

## Evidence

- [abstract] A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given"
- [intro] robustness claim intro: "The results are found to be more robust to noise and missing peaks compared to the alternatives (ORA, GSEA)."
- [methods] imputation rule: "Data imputation is performed to the intensity matrix when it is loaded"
- [readme] only annotated peaks used: "peaks that do not have annotations will not be used for pathway analysis"
- [readme] command-line invocation on HAT data: "python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control"
