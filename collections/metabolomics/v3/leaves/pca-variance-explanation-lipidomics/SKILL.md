---
name: pca-variance-explanation-lipidomics
description: Use when an annotated lipidomics feature table (e.g., 3647 features across
  replicate groups like Par, Dap2, Dal2, Van4) must be reduced to a few components
  to test whether strain or condition clusters separate before downstream PLS-DA or
  fold-change analyses.
when_to_use_negative:
- The data are raw, unannotated mass-spectrometric peaks rather than an annotated
  feature table — lipid feature identification is a prerequisite
- The goal is lipid species identification at different confidence levels rather than
  dimensionality reduction; use the lipydomics identification module instead
- A supervised discrimination result is already required — PCA is unsupervised, so
  PLS-DA (also in the stats module) is the appropriate choice when class labels drive
  the comparison
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0157
tools:
- name: LiPydomics
  role: Provides the data module to hold the feature table and the stats module to
    run three-component PCA with a scikit-learn backend, plus plotting to export projection
    PNGs
  repo: https://github.com/dylanhross/lipydomics
- name: scikit-learn
  role: Backend that performs the PCA computation inside the lipydomics stats module
- name: Python
  role: Language in which the scripted PCA analysis (19 lines for the full MRSA reanalysis)
    is executed reproducibly
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:370212e152197a35afeee94751240130ccc87279d19f456c5a98d084dcf2fb5b
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# PCA variance-explanation assessment for lipidomics feature tables

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute principal component analysis on an annotated lipidomics feature table using the LiPydomics stats module (scikit-learn backend) and judge whether a small number of components (e.g., three) captures ~90% of the variance while separating sample groups such as antimicrobial-resistant MRSA strains.

## When to use

Use when an annotated lipidomics feature table (e.g., 3647 features across replicate groups like Par, Dap2, Dal2, Van4) must be reduced to a few components to test whether strain or condition clusters separate before downstream PLS-DA or fold-change analyses.

## When NOT to use

- The data are raw, unannotated mass-spectrometric peaks rather than an annotated feature table — lipid feature identification is a prerequisite
- The goal is lipid species identification at different confidence levels rather than dimensionality reduction; use the lipydomics identification module instead
- A supervised discrimination result is already required — PCA is unsupervised, so PLS-DA (also in the stats module) is the appropriate choice when class labels drive the comparison

## Inputs

- Annotated lipidomics feature table (e.g., 3647 features with HILIC-IM-MS intensities for four replicates each of Par, Dap2, Dal2, Van4 MRSA strains), loadable as a lipydomics Dataset

## Outputs

- PCA projection figure (exported as PNG) showing strain cluster separation
- Per-component explained-variance values (target: ~90% cumulative across three components)
- PCA scores/projections persisted on the lipydomics Dataset object

## How to apply

Load the annotated feature table into a lipydomics Dataset via the data module, then compute a three-component PCA using the lipydomics stats module, which is backed by scikit-learn. Check that the retained components capture approximately 90% of the total variance — this is the acceptance threshold used in the MRSA reanalysis — and project the samples (four replicates per strain) to verify that Par, Dap2, Dal2, and Van4 form visually separated clusters. If ~90% variance is not reached in three components, the feature space is not dominated by the grouping of interest and multivariate separation claims should be treated cautiously. Export the PCA projection figures as PNG images so the cluster separation and variance-capture values can be audited alongside subsequent PLS-DA and Log2(fold-change) analyses.

## Related tools

- **LiPydomics** (Provides the data module to hold the feature table and the stats module to run three-component PCA with a scikit-learn backend, plus plotting to export projection PNGs) — https://github.com/dylanhross/lipydomics
- **scikit-learn** (Backend that performs the PCA computation inside the lipydomics stats module)
- **Python** (Language in which the scripted PCA analysis (19 lines for the full MRSA reanalysis) is executed reproducibly)

## Examples

```
from lipydomics.data import Dataset; from lipydomics.stats import pca; dset = Dataset('mrsa_dap2.pkl'); pca(dset, 3)
```

## Evaluation signals

- The three retained components cumulatively capture approximately 90% of the variance in the feature table
- The PCA projection shows separation of the Par, Dap2, Dal2, and Van4 strain clusters (four replicates each)
- Projection figures are exported as PNG images for the audited analysis
- The PCA scores are stored on the lipydomics Dataset instance so downstream PLS-DA and Log2(fold-change) analyses reuse the same object
- Reanalysis is reproducible with minimally processed data in a short scripted analysis (the MRSA study used 19 lines of Python)

## Limitations

- PCA variance capture and cluster separation are descriptive; they do not by themselves establish differential lipids — validated by the follow-up pattern check of decreased DGDG, increased LysylPG, and increased 15–21 carbon fatty acids in Dap2 vs Par
- The high dimensionality of lipidomics data (many features, fewer samples) makes interpretation difficult, so component count and separation must be reported explicitly
- Results depend on the feature table being minimally but adequately processed beforehand; LiPydomics assumes such an annotated table as input

## Evidence

- [abstract] Stats module for multivariate analyses: [evidence span withheld — anchor status: not_found]
