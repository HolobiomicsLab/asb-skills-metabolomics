---
name: sample-group-scatter-plot-visualization
description: Use when a wide-format metabolomics feature table with a sample-design
  file (treatment groups) needs per-sample visualization in PC space to reveal group
  clustering or separation.
when_to_use_negative:
- The goal is per-feature (loading) visualization or explained-variance assessment
  — a scree plot or loadings table is the right output, not a sample scatter plot.
- The input is not a feature table with a design file defining groups, so there is
  no grouping variable to color samples by.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SECIMTools
  role: Provides the PCA tool whose SVD-based projections and group-colored 2D/3D
    scatter plots this skill reproduces
  repo: https://github.com/SECIMTools
- name: Python
  role: Standalone implementation language for loading data with pandas, running SVD,
    and rendering the scatter plots
- name: Galaxy
  role: Web platform through which the SECIMTools PCA visualization tool can also
    be run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:ac018a7e83f736095d3053674b86425735aec3532b9dd8b7bb71fc4ee14b0a8f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sample-group-scatter-plot-visualization

## Summary

Generate 2D and 3D scatter plots of samples projected onto the first principal components, with points colored by design-file group (e.g., treatment group), as implemented in the SECIMTools PCA visualization tool for metabolomics data.

## When to use

Use when a wide-format metabolomics feature table with a sample-design file (treatment groups) needs per-sample visualization in PC space to reveal group clustering or separation.

## When NOT to use

- The goal is per-feature (loading) visualization or explained-variance assessment — a scree plot or loadings table is the right output, not a sample scatter plot.
- The input is not a feature table with a design file defining groups, so there is no grouping variable to color samples by.

## Inputs

- Wide-format metabolomics feature table (samples as columns, features as rows with unique feature identifiers), e.g. CSV
- Design file mapping sample identifiers to sample characteristics such as treatment group
- Per-sample PCA scores matrix (sample identifier, group, PC1..PCn columns)

## Outputs

- 2D scatter plot (PC1 vs PC2) with samples colored by design-file group, as PNG or PDF
- 3D scatter plot (PC1-PC3) with samples colored by design-file group, as PNG or PDF

## How to apply

Load the wide-format dataset (samples as columns, metabolomics features as rows with unique feature identifiers) and the design file relating samples to sample characteristics such as treatment group, using pandas. Center and optionally scale the feature matrix, then decompose it via singular value decomposition (SVD) as the SECIMTools PCA tool specifies; the resulting principal components are orthogonal and ordered in descending order of explained variability. Project samples onto PC1..PCn, then plot the first three principal components as 2D and 3D scatter plots with samples colored by the design-file group, saving the figures as PNG or PDF. Correctness is judged by comparison against the reference plots in Additional file 2, Figures S19 and S20.

## Related tools

- **SECIMTools** (Provides the PCA tool whose SVD-based projections and group-colored 2D/3D scatter plots this skill reproduces) — https://github.com/SECIMTools
- **Python** (Standalone implementation language for loading data with pandas, running SVD, and rendering the scatter plots)
- **Galaxy** (Web platform through which the SECIMTools PCA visualization tool can also be run)

## Examples

```
import pandas as pd; data = pd.read_csv('features.csv', index_col=0); design = pd.read_csv('design.csv'); U, S, Vt = np.linalg.svd(scale(data, with_mean=True), full_matrices=False)
```

## Evaluation signals

- The scores CSV contains the sample identifier, the design-file group, and PC1-PC3 columns.
- A plot file (PNG or PDF) exists after the run.
- Principal components are orthogonal and ordered in descending order of explained variability, verified against a companion scree plot of explained variance.
- Plot colors correspond one-to-one to design-file groups, matching the reference plots in Additional file 2, Figures S19 and S20.
- The script runs end-to-end on the bundled example wide-format dataset without errors.

## Limitations

- Only the first three principal components are visualized, so group separation occurring on later components is not shown.
- The visualization depends on prior centering and (optional) scaling of the feature matrix; unscaled heterogeneous feature magnitudes can distort the projection.
- Missing values in the feature table must be handled (e.g. by imputation) before decomposition, as SVD requires a complete matrix.

## Evidence

- [abstract] SECIMTools includes visualization techniques such as hierarchical cluster heatmap, PCA, and modularity clustering: "visualization techniques (hierarchical cluster heatmap, principal component analysis, modular modularity clustering)"
- [readme] The suite is available standalone and wrapped for Galaxy, and includes PCA among visualization techniques.: "The suite includes a comprehensive set of quality
control metrics (retention time window evaluation and various peak evaluation
tools), visualization techniques (hierarchical cluster heatmap,"
- [abstract] Standard data formats allow linking tools, which supports chaining the scores CSV into the plotting step.: "Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs."
