---
name: metabolomics-missing-feature-subsampling
description: Use when evaluating the stability of a pathway ranking method on metabolomics
  peak data where missing peaks and noise are prevalent, and you need a quantitative
  robustness comparison against alternative methods.
when_to_use_negative:
- Input data has no missing features and the question is about pathway ranking accuracy
  rather than robustness to missingness — full-data ranking alone suffices.
- Peak features lack compound annotations (KEGG/ChEBI IDs), since unannotated peaks
  are excluded from pathway analysis and the subsampling comparison would not be meaningful.
- The goal is transcriptomics/proteomics pathway ranking with gene-level databases
  (UniProt/ENSEMBL); this skill targets metabolomics peak data.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PALS
  role: Runs mPLAGE (PLAGE), ORA, and GSEA pathway ranking on full and subsampled
    intensity/annotation data; installed as 'pip install PALS-pathway' or from the
    git repository.
  repo: https://github.com/glasgowcompbio/PALS
- name: pandas
  role: Loads and passes int_df/annotation_df DataFrames directly into PALS programmatically
    and writes the aggregated benchmark CSV.
- name: SciPy
  role: Provides the preprocessing module used to standardise the log-2 transformed
    intensity matrix to zero mean and unit variance after PALS data imputation.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:b2daa9cec9abaf2e6a6a3bb49d2f3b7d12c381e7f28bb17bc10e66329b609539
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# metabolomics-missing-feature-subsampling

## Summary

Benchmark strategy that quantifies how robust pathway ranking methods (mPLAGE, ORA, GSEA) are when a large share of metabolomic peak features is randomly removed, by comparing subsampled rankings against full-data reference rankings using precision, recall, and F1 over hundreds of repetitions.

## When to use

Use when evaluating the stability of a pathway ranking method on metabolomics peak data where missing peaks and noise are prevalent, and you need a quantitative robustness comparison against alternative methods.

## When NOT to use

- Input data has no missing features and the question is about pathway ranking accuracy rather than robustness to missingness — full-data ranking alone suffices.
- Peak features lack compound annotations (KEGG/ChEBI IDs), since unannotated peaks are excluded from pathway analysis and the subsampling comparison would not be meaningful.
- The goal is transcriptomics/proteomics pathway ranking with gene-level databases (UniProt/ENSEMBL); this skill targets metabolomics peak data.

## Inputs

- Processed intensity CSV matrix (rows = peak features, columns = samples, with a group line mapping samples to experimental factors)
- Annotation CSV matrix mapping peak row_id to KEGG or ChEBI compound IDs (columns: row_id, entity_id)
- Experimental design dictionary or comparison spec (e.g. case/control names like Stage_1/Control)
- Pathway database choice (e.g. PiMP_KEGG)
- Minimum intensity for imputation (e.g. min_replace 5000)

## Outputs

- Full-data reference pathway ranking per dataset
- Per-repetition and aggregated precision/recall/F1 per method (mPLAGE, ORA, GSEA) per dataset
- Benchmark CSV table (e.g. missing_feature_benchmark_f1.csv)

## How to apply

Start by running PALS with the PLAGE decomposition on the complete intensity matrix (e.g. HAT Plasma and CSF processed data annotated to KEGG/ChEBI compound IDs via the PiMP_KEGG database, with comparisons such as Stage_1/Control and Stage_2/Control) to establish the full-data reference pathway ranking. Then, for each repetition (500 in the paper), randomly retain only 20% of the annotated peak features and rerun mPLAGE, ORA, and GSEA on the subsampled data using the same experimental design and default min_replace 5000 imputation. For each repetition and method, compute precision, recall, and F1 by treating the subsampled top-ranked pathways as predictions against the full-data reference ranking. Aggregate metrics per method per dataset; the paper's expected result is that mPLAGE is the most robust, reaching mean F1 of about 0.48 on Plasma and 0.60 on CSF at 80% missing features, versus about 0.20/0.50 for ORA and 0.08/0.16 for GSEA. Write the aggregated precision/recall/F1 per method per dataset to a CSV benchmark table.

## Related tools

- **PALS** (Runs mPLAGE (PLAGE), ORA, and GSEA pathway ranking on full and subsampled intensity/annotation data; installed as 'pip install PALS-pathway' or from the git repository.) — https://github.com/glasgowcompbio/PALS
- **pandas** (Loads and passes int_df/annotation_df DataFrames directly into PALS programmatically and writes the aggregated benchmark CSV.)
- **SciPy** (Provides the preprocessing module used to standardise the log-2 transformed intensity matrix to zero mean and unit variance after PALS data imputation.)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- The benchmark CSV contains precision, recall, and F1 columns for all three methods (mPLAGE, ORA, GSEA) on both datasets (Plasma and CSF).
- mPLAGE F1 values reproduce the reported robustness advantage: ~0.48 (Plasma) and ~0.60 (CSF), higher than ORA (~0.20/0.50) and GSEA (~0.08/0.16) at 80% feature removal.
- The full-data reference ranking is generated with the same experimental design (comparisons, database, min_replace) as the subsampled runs.
- Each of the 500 repetitions uses a different random 20% subset of annotated peak features.

## Limitations

- Robustness conclusions depend on the chosen full-data reference ranking; the method assumes this reference is meaningful.
- The benchmark as published covers HAT Plasma and CSF data with the PiMP_KEGG database; other datasets/species may require adaptation of comparisons and database choice.
- Results are affected by the data imputation step (min_replace, mean substitution within a factor) and log-2/standardisation applied before ranking.
- Only metabolic pathways are available in offline Reactome mode; online Neo4j mode is needed for all pathways.

## Evidence

- [readme] Data imputation replaces all-zero factors by min_replace (5000) then log-2 and SciPy standardisation before ranking.: "Data imputation is performed to the intensity matrix when it is loaded: if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [abstract] PALS results are more robust to missing peaks than ORA/GSEA.: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given and reveals that mPLAGE is more robust to missing"
- [readme] Command-line run of PALS with PLAGE on HAT data with PiMP_KEGG and Stage_1/Stage_2 vs Control comparisons.: "python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control"
- [readme] PALS can be imported programmatically with pandas DataFrames and experimental design dict.: "When PALS is used programatically, pandas dataframes storing relevant data can be passed directly. Experimental design data can be passed directly as a dictionary structure in the programmatic use."
- [other] Precision/recall/F1 computed against full-data reference ranking.: [evidence span withheld — anchor status: not_found]
