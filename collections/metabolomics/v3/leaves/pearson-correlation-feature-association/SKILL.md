---
name: pearson-correlation-feature-association
description: Use when an untargeted lipidomics feature table has few biological groups
  (e.g. resistant vs parent strains, ~4 replicates each) and you need a feature-level
  association ranking to complement multivariate PCA/PLS-DA cluster separation.
when_to_use_negative:
- Input is raw HILIC-IM-MS spectra (mzML) rather than an annotated feature table —
  identification/matching, not correlation, is needed
- The goal is lipid CCS or HILIC retention time prediction — that is the prediction/identification
  functionality, not the stats module
- Only two features or fewer than a handful of replicates exist, where Pearson ranks
  are unstable and uninformative
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LiPydomics
  role: Provides the 'stats' module functions that compute Pearson correlations on
    a lipydomics Dataset, alongside 'data' and 'plotting' modules used in the same
    analysis flow
  repo: https://github.com/dylanhross/lipydomics
- name: SciPy
  role: Backend supplying the correlation functions used by lipydomics' stats module
    for the Dap2-vs-Par comparison
- name: scikit-learn
  role: Backend for the multivariate (PCA/PLS-DA) analyses run alongside the Pearson
    correlations on the same Dataset
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:a7c9bc4fa57c228242f95b65786d4215e57437bcf358a9f34b9f18158a4d2dc2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dylanhross/lipydomics
  repo_url: https://github.com/dylanhross/lipydomics
  tools:
  - LiPydomics
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# pearson-correlation-feature-association

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Feature-wise Pearson correlation of an annotated lipidomics feature table against group labels (e.g. daptomycin-resistant Dap2 vs parent Par MRSA strains), computed with the lipydomics 'stats' module backed by SciPy correlation functions, to rank which of the ~3647 features associate with antimicrobial resistance.

## When to use

Use when an untargeted lipidomics feature table has few biological groups (e.g. resistant vs parent strains, ~4 replicates each) and you need a feature-level association ranking to complement multivariate PCA/PLS-DA cluster separation.

## When NOT to use

- Input is raw HILIC-IM-MS spectra (mzML) rather than an annotated feature table — identification/matching, not correlation, is needed
- The goal is lipid CCS or HILIC retention time prediction — that is the prediction/identification functionality, not the stats module
- Only two features or fewer than a handful of replicates exist, where Pearson ranks are unstable and uninformative

## Inputs

- Annotated lipidomics feature table (CSV, features x samples, e.g. 3647 features with four replicates per strain)
- Sample-group labels (e.g. Par, Dap2, Dal2, Van4) defining the comparison pair
- lipydomics Dataset instance created via the 'data' module

## Outputs

- Per-feature Pearson correlation values associating each feature with the group comparison (Dap2 vs Par)
- Heat map of feature associations / fold changes (plotting module)
- Exported statistics table, e.g. fold_change_heatmap_table.csv

## How to apply

Load the annotated feature table (e.g. 3647 features, four replicates each of Par, Dap2, Dal2, Van4) into a lipydomics Dataset via the 'data' module. From the 'stats' module, compute Pearson correlations comparing Dap2 vs Par; under the hood these calls use SciPy correlation functions. Pair the correlation ranking with Log2(fold-change) of Dap2 relative to Par for all features and heat maps from the 'plotting' module, exporting statistics as fold_change_heatmap_table.csv. Rationale: the article notes that high-dimensional lipidomics data (many features, fewer samples) are hard to interpret, so per-feature Pearson correlation provides a directly interpretable univariate companion to PCA/PLS-DA projections. In the MRSA reanalysis, this whole flow (PCA, PLS-DA, Pearson, log2 fold-change) reproduced the published Dap2-vs-Par pattern in only 19 lines of Python on minimally processed data.

## Related tools

- **LiPydomics** (Provides the 'stats' module functions that compute Pearson correlations on a lipydomics Dataset, alongside 'data' and 'plotting' modules used in the same analysis flow) — https://github.com/dylanhross/lipydomics
- **SciPy** (Backend supplying the correlation functions used by lipydomics' stats module for the Dap2-vs-Par comparison)
- **scikit-learn** (Backend for the multivariate (PCA/PLS-DA) analyses run alongside the Pearson correlations on the same Dataset)

## Examples

```
from lipydomics.data import Dataset; from lipydomics import stats; ds = Dataset('MRSA_feature_table.csv'); stats.pearson_correlations(ds, 'Dap2', 'Par')
```

## Evaluation signals

- The correlation computation covers all 3647 features of the annotated MRSA table, not a filtered subset
- Resulting Dap2-vs-Par pattern in fold_change_heatmap_table.csv reproduces the reported decreased DGDG, increased LysylPG, and increased 15–21 carbon fatty acids
- Correlation ranking is consistent with the PCA (~90% variance in three components) and PLS-DA strain-cluster separation computed on the same Dataset
- Entire reanalysis (PCA, PLS-DA, Pearson, log2 fold-change) reproducibly runs in ~19 lines of Python on minimally processed data
- Exported CSV and PNG figures are generated with defined names (fold_change_heatmap_table.csv) for diff-based reproducibility checks

## Limitations

- Pearson correlation is a univariate association measure; it must be paired with PCA/PLS-DA because high-dimensional data (many features, fewer samples) are otherwise difficult to interpret
- Requires a minimally processed, already-annotated feature table; raw spectra must be processed upstream before this skill applies
- The demonstration used S. aureus (MRSA) strains resistant to various antimicrobials with four replicates per group; results with other designs are not validated here

## Evidence

- [abstract] which enables performing statistical and multivariate analyses ("stats" module): [evidence span withheld — anchor status: not_found]
