---
name: gene-identifier-standardization
description: Use when raw CCLE/DepMap omics feature matrices mix identifier namespaces
  (e.g. RNA splicing ensemble IDs, BiggIDs) and feature names must be mapped to common
  gene names before cross-omics ML training or network output.
when_to_use_negative:
- Input feature names are already in a single consistent namespace (e.g. all genes
  are already common symbols) so renaming would add no signal
- The task is training or predicting metabolite values rather than preparing the feature
  matrices — standardization is a preprocessing sub-step, not the modeling step itself
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: BioTools.fr
  role: Online service used to convert RNA splicing ensemble IDs to common names
- name: Python (pandas)
  role: Scripting environment for applying identifier mappings across the omics feature
    matrices
- name: Metab8D recon_mapping
  role: MATLAB/Python scripts and CSV mapping common gene names to BiggIDs for matching
    features to Recon3D reactions
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:090fdd9100a3a25f09bf4df058262c5f814084d42b46b3d6d23b13c7ba805389
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - BioTools.fr
  - Metab8D recon_mapping
  - Python
  - Python (pandas)
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Gene identifier standardization

## Summary

Standardizes heterogeneous omics feature identifiers (e.g. RNA splicing ensemble IDs) into consistent common gene names using BioTools.fr so that features across the eight CCLE biomolecular classes are consistently labeled before random forest modeling and network construction.

## When to use

Use when raw CCLE/DepMap omics feature matrices mix identifier namespaces (e.g. RNA splicing ensemble IDs, BiggIDs) and feature names must be mapped to common gene names before cross-omics ML training or network output.

## When NOT to use

- Input feature names are already in a single consistent namespace (e.g. all genes are already common symbols) so renaming would add no signal
- The task is training or predicting metabolite values rather than preparing the feature matrices — standardization is a preprocessing sub-step, not the modeling step itself

## Inputs

- Raw CCLE omics feature matrices for each of the eight biomolecular classes (CNV, mutations, DNA methylation, histone PTMs, coding transcripts, RNA splicing, miRNA, lncRNA, proteomics, phosphoproteomics) from the Broad Dependency Map portal
- RNA splicing feature identifiers (ensemble IDs)
- recon_mapping CSV mapping common gene names to BiggIDs

## Outputs

- Per-class feature matrices with uniformly labeled common gene names
- Standardized feature lists ready for zero-variance removal and Z-score normalization
- Feature names compatible with the Metab8D_network.xlsx top-20 feature output

## How to apply

After loading each raw CCLE omics feature matrix (from the Broad Dependency Map portal), standardize feature names to a common namespace. For the RNA splicing feature set, convert ensemble IDs to common names using BioTools.fr; for metabolic model-based filtering, translate common gene names to BiggIDs using the recon_mapping CSV from the Metab8D repo. Standardization happens alongside the NaN-to-zero imputation, median collapse of repeated observations, and zero-variance removal steps, so apply it before Z-score normalization within the 5-fold CV splits. Consistent naming is required so that top features from the Metab8D_network.xlsx output can be matched back to genes in Recon3D reactions for downstream network inference.

## Related tools

- **BioTools.fr** (Online service used to convert RNA splicing ensemble IDs to common names)
- **Python (pandas)** (Scripting environment for applying identifier mappings across the omics feature matrices)
- **Metab8D recon_mapping** (MATLAB/Python scripts and CSV mapping common gene names to BiggIDs for matching features to Recon3D reactions) — https://github.com/sriram-lab/Metab8D

## Examples

```
import pandas as pd; df = pd.read_csv('splicing_raw.csv'); df['feature'] = df['feature'].map(ensemble_to_common_from_BioToolsfr_mapping); df.to_csv('splicing_standardized.csv', index=False)
```

## Evaluation signals

- Resulting splicing feature names are common gene names rather than raw ensemble IDs
- Feature counts after preprocessing match reported values (e.g. 3,767 lncRNA, 2,957 splicing events)
- Each standardized feature name maps unambiguously to exactly one row in the per-class matrix (no duplicates after median collapse)
- Standardized names can be joined to recon_mapping BiggIDs when matching against Recon3D reactions

## Limitations

- Identifier conversion services (BioTools.fr) may not cover all ensemble IDs, leaving unmatched features after standardization
- Translation between common names and BiggIDs depends on the recon_mapping CSV staying in sync with the Recon3D model version used
- Binary mutation features were excluded from RF results, limiting usefulness of standardized mutation identifiers in the final network

## Evidence

- [discussion] Data preprocessing involved replacing missing values (NaN) with zeros, collapsing duplicate entries by median, and removing features with 0 variance.: [evidence span withheld — anchor status: not_found]
