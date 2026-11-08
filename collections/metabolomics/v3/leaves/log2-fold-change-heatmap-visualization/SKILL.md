---
name: log2-fold-change-heatmap-visualization
description: Use when an annotated lipidomics feature table has two strain/condition
  groups to compare and you need per-feature log2 fold-change statistics and a heat
  map to spot lipid class-level abundance patterns.
when_to_use_negative:
- The input is unprocessed vendor spectra or an imzML/centroided file rather than
  an already annotated feature table — lipydomics operates on datasets loaded into
  its Dataset structure, not raw acquisitions
- The comparison requires univariate significance testing or multivariate separation
  (PCA/PLS-DA) rather than per-feature fold-change visualization — use the corresponding
  stats-module workflows instead
- Lipid species identification from m/z, CCS, and retention time is the goal — use
  the identification module, not fold-change/plotting
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LiPydomics
  role: stats module computes Log2(fold-change) on a Dataset; plotting module generates
    the heat map from computed statistics
  repo: https://github.com/dylanhross/lipydomics
- name: Python
  role: language in which the scripted fold-change/heatmap analysis is executed
  repo: https://github.com/dylanhross/lipydomics
- name: SciPy
  role: backend scientific-computing functions used by lipydomics stats computations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:134984201c9227913c496b52c8b84854e261ec1fd59d494af66bba4d634fe9a9
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# log2-fold-change-heatmap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute Log2(fold-change) of every feature between two conditions (e.g., Dap2 vs Par MRSA strains) directly on a lipydomics Dataset and render the result as a heat map plus an exported CSV table, revealing coordinated lipid class shifts such as decreased DGDG and increased LysylPG.

## When to use

Use when an annotated lipidomics feature table has two strain/condition groups to compare and you need per-feature log2 fold-change statistics and a heat map to spot lipid class-level abundance patterns.

## When NOT to use

- The input is unprocessed vendor spectra or an imzML/centroided file rather than an already annotated feature table — lipydomics operates on datasets loaded into its Dataset structure, not raw acquisitions
- The comparison requires univariate significance testing or multivariate separation (PCA/PLS-DA) rather than per-feature fold-change visualization — use the corresponding stats-module workflows instead
- Lipid species identification from m/z, CCS, and retention time is the goal — use the identification module, not fold-change/plotting

## Inputs

- Annotated lipidomics feature table with replicates per condition group (e.g., 3647 features, four replicates each of Par, Dap2, Dal2, Van4 MRSA strains)
- lipydomics Dataset instance holding the feature table and metadata

## Outputs

- Log2(fold-change) values for all features (Dap2 relative to Par)
- Heat map figure exported as PNG
- fold_change_heatmap_table.csv containing the fold-change statistics

## How to apply

Load the minimally processed annotated feature table (e.g., 3647 MRSA features with four replicates each of Par, Dap2, Dal2, Van4) into a lipydomics Dataset via the data module. Use the lipydomics stats module, backed by SciPy/scikit-learn, to compute Log2(fold-change) of Dap2 relative to Par for all features. Pass the computed statistics to the lipydomics plotting module to generate heat maps that use the computed stats or the data itself. Export the figures as PNG and the fold-change values as fold_change_heatmap_table.csv. The rationale is that dimensionality (many features, fewer samples) makes raw feature tables hard to interpret, so fold-change plus heat map visualization surfaces coordinated lipid class patterns.

## Related tools

- **LiPydomics** (stats module computes Log2(fold-change) on a Dataset; plotting module generates the heat map from computed statistics) — https://github.com/dylanhross/lipydomics
- **Python** (language in which the scripted fold-change/heatmap analysis is executed) — https://github.com/dylanhross/lipydomics
- **SciPy** (backend scientific-computing functions used by lipydomics stats computations)

## Examples

```
python -c "from lipydomics import data, stats, plotting; ds = data.Dataset('data.h5')"
```

## Evaluation signals

- fold_change_heatmap_table.csv reproduces the reported Dap2-vs-Par pattern of decreased DGDG, increased LysylPG, and increased 15–21 carbon fatty acids
- Log2(fold-change) is computed for all 3647 features, not a subset, using Dap2 relative to Par
- Heat map PNG renders from stats stored on the lipydomics Dataset (not re-derived ad hoc), matching the computed statistics
- Fold-change directionality is consistent with any companion PLS-DA / Pearson correlation results computed on the same Dataset

## Limitations

- Fold-change analysis alone does not establish statistical significance — pair it with Pearson correlations or PLS-DA from the stats module for inferential support
- Interpretation is constrained by feature-table dimensionality (many features, fewer samples), which makes raw results hard to read without visualization
- Computed statistics are stored on the Dataset instance; users must persist/export them (e.g., to fold_change_heatmap_table.csv) or results are not reproducible from the script alone

## Evidence

- [abstract] which enables performing statistical and multivariate analyses ("stats" module), generating informative plots ("plotting" module): [evidence span withheld — anchor status: not_found]
