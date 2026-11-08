---
name: missing-feature-perturbation-benchmarking
description: Use when comparing pathway-ranking methods (PLAGE/mPLAGE, ORA, GSEA)
  for robustness to missing peaks in metabolomics feature tables, by synthetically
  injecting known missingness levels (20/40/60/80%) over many seeded replicates.
when_to_use_negative:
- The goal is a single one-shot pathway ranking on an existing dataset, not a robustness
  comparison — plain PALS (PLAGE/ORA/GSEA) output suffices without perturbation replicates.
- Missing features arise naturally in the measured data and only imputation is needed
  — the PALS loading-time imputation (min_replace / mean-of-non-zero) handles this
  without the synthetic removal loop.
- Peak features lack compound annotations entirely — peaks without annotations are
  excluded from pathway analysis, so the perturbation benchmark would have no annotated
  features to remove.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: PALS
  role: Runs the three ranking methods (pals.PLAGE, pals.ORA, pals.GSEA) via the DataSource
    API on each perturbed replicate; also performs loading-time imputation, log-2
    transformation, and SciPy standardisation
  repo: https://github.com/glasgowcompbio/PALS
- name: pandas
  role: Holds the intensity and annotation DataFrames passed programmatically to the
    PALS DataSource
- name: SciPy
  role: Preprocessing module used to standardise the intensity matrix to zero mean
    and unit variance across samples
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:9ea62a7d4b69e29df9688b8d124af8c8e592e641be1e34a8fa7916be3a6d7faf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# missing-feature-perturbation-benchmarking

## Summary

A controlled perturbation benchmark that randomly removes 20–80% of annotated peak features from a metabolomics intensity matrix across hundreds of seeded replicates to quantify the robustness of pathway-ranking methods (mPLAGE/PLAGE vs ORA vs GSEA) by aggregated p-value distributions and F1 scores.

## When to use

Use when comparing pathway-ranking methods (PLAGE/mPLAGE, ORA, GSEA) for robustness to missing peaks in metabolomics feature tables, by synthetically injecting known missingness levels (20/40/60/80%) over many seeded replicates.

## When NOT to use

- The goal is a single one-shot pathway ranking on an existing dataset, not a robustness comparison — plain PALS (PLAGE/ORA/GSEA) output suffices without perturbation replicates.
- Missing features arise naturally in the measured data and only imputation is needed — the PALS loading-time imputation (min_replace / mean-of-non-zero) handles this without the synthetic removal loop.
- Peak features lack compound annotations entirely — peaks without annotations are excluded from pathway analysis, so the perturbation benchmark would have no annotated features to remove.

## Inputs

- Peak-feature intensity matrix CSV (rows = peak features, columns = samples with group line), e.g. notebooks/test_data/HAT/int_df.csv
- Peak-to-compound annotation table (peak ID, KEGG/ChEBI entity ID), e.g. notebooks/test_data/HAT/annotation_df.csv
- Pathway database choice (PiMP_KEGG exported from PiMP)
- Experimental design with comparisons (case/control, e.g. Stage_1/Control, Stage_2/Control)
- Random seeds for replicate perturbations (500 distinct seeds per missingness level)

## Outputs

- Aggregated per-method/per-missingness table of mean p-value and p-value variance, e.g. missing_feature_benchmark_results.csv
- Per-replicate pathway p-value rankings from pals.PLAGE, pals.ORA, and pals.GSEA
- F1 scores of each method against known true pathways at the fixed decoy level

## How to apply

Load the peak intensity matrix (e.g. notebooks/test_data/HAT/int_df.csv) and annotation file into pandas DataFrames, applying PALS imputation rules — all-zero factors replaced by --min_replace 5000, partially-zero factors by the mean of non-zero samples in that factor — followed by log-2 transformation and SciPy zero-mean/unit-variance standardisation. Configure the pathway database (PiMP_KEGG) and case/control comparisons (Stage_1/Control, Stage_2/Control) with a fixed decoy metabolite-set level (100%) so only missingness varies. For each missingness level in {20%, 40%, 60%, 80%}, randomly remove that fraction of annotated peak features and repeat the perturbation 500 times with distinct random seeds. At each replicate, run the three ranking methods via the PALS library (pals.PLAGE, pals.ORA, pals.GSEA through the DataSource API) and collect pathway p-values, plus F1 scores against known true pathways. Aggregate per-method mean p-value and p-value variance per missingness level and verify that mPLAGE yields consistently lower mean p-values with smaller variance as missingness grows, while ORA and GSEA degrade.

## Related tools

- **PALS** (Runs the three ranking methods (pals.PLAGE, pals.ORA, pals.GSEA) via the DataSource API on each perturbed replicate; also performs loading-time imputation, log-2 transformation, and SciPy standardisation) — https://github.com/glasgowcompbio/PALS
- **pandas** (Holds the intensity and annotation DataFrames passed programmatically to the PALS DataSource)
- **SciPy** (Preprocessing module used to standardise the intensity matrix to zero mean and unit variance across samples)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control
```

## Evaluation signals

- At every missingness level (20%, 40%, 60%, 80%), mPLAGE shows a lower mean pathway p-value and smaller p-value variance than ORA and GSEA.
- mPLAGE achieves the highest F1 score against known true pathways across increasing missingness at the fixed 100% decoy level.
- Each missingness level is backed by 500 replicate runs with distinct random seeds; re-running with a fixed seed reproduces the same perturbation sets.
- Output table schema contains one aggregated row per (method, missingness level) with mean p-value, variance, and F1 columns.
- Imputation invariants hold on the unperturbed matrix: all-zero factors replaced by min_replace=5000, partially-zero factors by the mean of non-zero samples, matrix log-2 transformed and standardised to zero mean/unit variance.

## Limitations

- Robustness conclusions rest on a synthetic decoy design (decoy metabolite-set level fixed at 100%) rather than naturally missing peaks; results may not transfer to other perturbation designs.
- No software version numbers are stated for PALS or its dependencies, so exact reproducibility is uncertain.
- Offline database mode only covers metabolic pathways (e.g. downloaded KEGG/PiMP_KEGG), and peaks without compound annotations are excluded from pathway analysis, shrinking the perturbable feature pool.
