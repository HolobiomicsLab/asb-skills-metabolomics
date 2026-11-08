---
name: multi-omics-data-preprocessing
description: Use when raw DepMap/CCLE omics feature tables (CNV, mutations, methylation,
  histone PTMs, RNA, miRNA, lncRNA, proteomics, phosphoproteomics) must be cleaned
  and normalized before training per-metabolite ML models.
when_to_use_negative:
- Input is already a clean, Z-scored feature table — re-applying median collapsing
  or zero-variance removal would be redundant.
- When the goal is prediction on non-CCLE (e.g. in vivo or other cell-line-panel)
  data, where the article notes models are not directly transferrable due to differences
  between in vitro and in vivo conditions and limited feature overlap.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0622
tools:
- name: scikit-learn
  role: Implements the Random Forest Regressor used downstream of preprocessing; its
    CV utilities support the fold-wise Z-score normalization strategy.
  repo: https://github.com/sriram-lab/Metab8D
- name: BioTools.fr
  role: Converts RNA splicing ensemble IDs to common gene names during feature-name
    standardization.
- name: Python
  role: Language for the preprocessing scripts (ML_function.ipynb, preprocessing_example.ipynb)
    that perform NaN-to-zero imputation, median collapsing, and feature filtering.
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:79ef91926893cea436a86a0f2c649e39c6a66d60098ae79de9c3dda5f456e47c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - BioTools.fr
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# multi-omics-data-preprocessing

## Summary

Preprocesses raw CCLE omics feature matrices for the ten feature sets of Metab8D by imputing NaNs with zeros, collapsing repeated observations via medians, removing zero-variance features, and Z-score normalizing within each 5-fold CV split so random forest metabolite predictors can be trained.

## When to use

Use when raw DepMap/CCLE omics feature tables (CNV, mutations, methylation, histone PTMs, RNA, miRNA, lncRNA, proteomics, phosphoproteomics) must be cleaned and normalized before training per-metabolite ML models.

## When NOT to use

- Input is already a clean, Z-scored feature table — re-applying median collapsing or zero-variance removal would be redundant.
- When the goal is prediction on non-CCLE (e.g. in vivo or other cell-line-panel) data, where the article notes models are not directly transferrable due to differences between in vitro and in vivo conditions and limited feature overlap.

## Inputs

- Raw CCLE omics feature matrices from the DepMap portal for 8 biomolecular classes (CNV, mutations, DNA methylation, histone PTMs, coding RNA, RNA splicing, miRNA, lncRNA, proteomics, phosphoproteomics)
- Histone PTM data example (original and zero-imputed) from the Metab8D repo's example_datasets directory
- recon_mapping gene-name-to-BigGID translation CSV for filtering to metabolically relevant genes

## Outputs

- Preprocessed per-omics-class feature matrices ready for random forest training (feature counts such as 3,767 lncRNA, 2,957 splicing, 17,203 mutation features)
- Feature-count table per omics class for reporting

## How to apply

Load the raw CCLE omics matrices (e.g. from the Broad Institute DepMap portal) with Python/pandas and replace missing values (NaN) with zeros. Collapse repeated observations per feature by taking the median, and standardize feature names (e.g. convert RNA splicing ensemble IDs to common names via BioTools.fr). Drop features with zero variance, which carry no signal for downstream random forest models. Finally, Z-score normalize features within each 5-fold cross-validation training fold — not globally — so the normalization avoids information leakage across CV folds. The result is per-class feature matrices with reported counts such as 3,767 lncRNA, 2,957 splicing, and 17,203 mutation features; input datasets containing metabolically relevant genes are additionally filtered for genes present in the Recon3D genome-scale metabolic model.

## Related tools

- **scikit-learn** (Implements the Random Forest Regressor used downstream of preprocessing; its CV utilities support the fold-wise Z-score normalization strategy.) — https://github.com/sriram-lab/Metab8D
- **BioTools.fr** (Converts RNA splicing ensemble IDs to common gene names during feature-name standardization.)
- **Python** (Language for the preprocessing scripts (ML_function.ipynb, preprocessing_example.ipynb) that perform NaN-to-zero imputation, median collapsing, and feature filtering.) — https://github.com/sriram-lab/Metab8D

## Examples

```
jupyter notebook ML_function.ipynb  # includes histone PTM preprocessing; or: python preprocessing_example.ipynb --data example_datasets/histone_ptm.csv
```

## Evaluation signals

- Feature counts per class match the reported values (e.g. 3,767 lncRNA, 2,957 splicing, 17,203 mutation features) after preprocessing.
- No feature with zero variance remains in the final matrix.
- Z-score statistics (mean 0, SD 1) hold within each CV training fold, not across the full matrix.
- NaN-to-zero imputation is verifiable by comparing against the zero-imputed example histone PTM data in the repo's example_datasets folder.
- All features retained map to genes present in the Recon3D genome-scale metabolic model.

## Limitations

- The article notes models trained on CCLE data are not directly transferrable to other sources due to differences between in vitro and in vivo conditions, cell-line panels, and limited feature overlap.
- Input datasets containing metabolically relevant genes must first be filtered for genes present in the Recon3D genome-scale metabolic model, so the skill is scoped to metabolically relevant features.
- RNA splicing ensemble IDs require external conversion via BioTools.fr; any failure in this lookup will leave non-standardized feature names.

## Evidence
