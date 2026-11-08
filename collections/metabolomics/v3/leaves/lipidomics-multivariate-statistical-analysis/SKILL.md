---
name: lipidomics-multivariate-statistical-analysis
description: Use when you have an annotated lipidomics feature table (e.g. thousands
  of m/z features across replicate strain/treatment groups, such as Par, Dap2, Dal2,
  Van4) and need PCA/PLS-DA clustering plus per-feature fold-change statistics to
  compare lipidomes.
when_to_use_negative:
- The goal is lipid identification or CCS/retention-time prediction rather than statistical
  comparison — the 'identification' module and predicted lipid database (145,388 entries)
  serve that purpose instead
- Input is raw instrument data (not an annotated feature table) or you need to build
  the feature table itself from vendor/centroided files
- The analysis target is not lipidomics (the package and its stats/plotting modules
  are specialized to HILIC-IM-MS lipidomics data)
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LiPydomics
  role: 'Primary package: its ''data'' module stores the lipidomics Dataset with metadata
    and computed statistics; ''stats'' module computes PCA, PLS-DA, Pearson correlations,
    and Log2 fold-change; ''plotting'' module generates projection figures and heat
    maps; ''interactive'' module offers a text-based interface'
  repo: https://github.com/dylanhross/lipydomics
- name: Python
  role: Execution environment for the lipydomics API or interactive module
- name: scikit-learn
  role: Backend behind the lipydomics stats module for multivariate analyses such
    as PCA and PLS-DA
- name: SciPy
  role: Provides the correlation functions used by the lipydomics stats module for
    Pearson correlation between strain groups
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:e79224c7c27615b5de7c64598c4ad21da937ccc860d9bacc8c93c2f260099f5d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lipidomics-multivariate-statistical-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply the lipydomics 'stats' and 'plotting' modules to perform multivariate analyses (PCA, PLS-DA) and univariate comparisons (Pearson correlation, Log2 fold-change) on HILIC-IM-MS lipidomics feature tables, as demonstrated by reproducing antimicrobial-resistance lipid changes between S. aureus/MRSA strains.

## When to use

Use when you have an annotated lipidomics feature table (e.g. thousands of m/z features across replicate strain/treatment groups, such as Par, Dap2, Dal2, Van4) and need PCA/PLS-DA clustering plus per-feature fold-change statistics to compare lipidomes.

## When NOT to use

- The goal is lipid identification or CCS/retention-time prediction rather than statistical comparison — the 'identification' module and predicted lipid database (145,388 entries) serve that purpose instead
- Input is raw instrument data (not an annotated feature table) or you need to build the feature table itself from vendor/centroided files
- The analysis target is not lipidomics (the package and its stats/plotting modules are specialized to HILIC-IM-MS lipidomics data)

## Inputs

- Annotated lipidomics feature table (HILIC-IM-MS data; e.g. 3647 features with replicate columns per strain, minimally processed)
- Condition/group metadata for replicates (e.g. Par, Dap2, Dal2, Van4 MRSA strain groups)

## Outputs

- PCA/PLS-DA projection figures (PNG)
- Per-feature Log2(fold-change) statistics and Pearson correlations comparing the strain contrast
- Heat map of lipid class changes exported as CSV (e.g. fold_change_heatmap_table.csv)

## How to apply

Load the annotated feature table (e.g. 3647 features with four replicates per condition) into a lipydomics Dataset via the 'data' module. Compute multi-component PCA using the 'stats' module (scikit-learn backend) and check that the chosen number of components captures a high fraction of the variance (in the MRSA demonstration, three components captured approximately 90%) with visible separation of strain clusters. Compute PLS-DA projections and Pearson correlations (SciPy correlation functions) for the strain contrast of interest (Dap2 vs Par). Compute Log2(fold-change) of the resistant strain relative to the parent for all features and render heat maps with the 'plotting' module. Export projection figures as PNG images and the statistics as a CSV table (e.g. fold_change_heatmap_table.csv) so the analysis is reproducible; the README notes the package is designed for 'standard lipidomics analysis protocols... in an efficient and reproducible fashion'.

## Related tools

- **LiPydomics** (Primary package: its 'data' module stores the lipidomics Dataset with metadata and computed statistics; 'stats' module computes PCA, PLS-DA, Pearson correlations, and Log2 fold-change; 'plotting' module generates projection figures and heat maps; 'interactive' module offers a text-based interface) — https://github.com/dylanhross/lipydomics
- **Python** (Execution environment for the lipydomics API or interactive module)
- **scikit-learn** (Backend behind the lipydomics stats module for multivariate analyses such as PCA and PLS-DA)
- **SciPy** (Provides the correlation functions used by the lipydomics stats module for Pearson correlation between strain groups)

## Examples

```
from lipydomics.data import Dataset; from lipydomics.stats import compute_pca, compute_plsda, compute_log2fc; ds = Dataset('mrsa_features.csv'); compute_pca(ds, 3)
```

## Evaluation signals

- PCA component variance: chosen components capture approximately 90% of variance (three components in the MRSA demonstration) with clear separation of strain clusters
- Heat map table reproduces the reported Dap2-vs-Par pattern: decreased DGDG, increased LysylPG, and increased 15–21 carbon fatty acids
- Outputs are exported as expected artifacts: PNG projection figures and fold_change_heatmap_table.csv
- Analysis remains compact and reproducible on minimally processed data (the MRSA reanalysis was done in only 19 lines of Python)
- Dataset object retains associated metadata and computed statistics, consistent with the 'data' module design

## Limitations

- HILIC-IM-MS-based analysis generally supports Level 3 lipid identifications (lipid class/subclass and fatty acid sum composition), limiting the depth of feature annotation
- The stats module depends on scikit-learn and SciPy backends, so results inherit those implementations' conventions
- The package is extensible, but the README redirects bug reports and questions to the maintainers (<maintainer e-mail, see the repository README> or GitHub issues); statistical module results should be sanity-checked against expected strain patterns

## Evidence

- [abstract] LiPydomics enables stats, plotting, identification, and interactive analysis modules for HILIC-IM-MS lipidomics data.: [evidence span withheld — anchor status: not_found]
