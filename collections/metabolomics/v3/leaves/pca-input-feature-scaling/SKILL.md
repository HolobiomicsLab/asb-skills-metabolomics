---
name: pca-input-feature-scaling
description: Use when a raw metabolite concentration/intensity table (e.g., 90 metabolites
  from integrated NMR peak areas across hESC→EB→Rosette→hNPC→Neuron stages) must be
  scaled before PCA so features contribute comparably to components.
when_to_use_negative:
- Input is already a normalized/scaled feature table — re-normalizing would distort
  the variance structure PCA relies on.
- Analysis does not involve dimensionality reduction by PCA (e.g., raw NMR peak deconvolution
  or Chenomx peak identification, which occur upstream of this scaling step).
- Data are non-numeric categorical labels (e.g., cell-stage markers CD24, ZO-1, Nestin,
  MAP2, vGLUT1 from flow cytometry) rather than quantitative metabolite intensity
  columns.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: Python
  role: Language used to implement the mean- and stddev-normalization preprocessing
    step with pandas before PCA
- name: scikit-learn
  role: Provides the PCA implementation that consumes the mean- and stddev-normalized
    dataset (PCA run alongside GLASSO from sklearn.covariance)
- name: pcaglasso
  role: MetaboLINK package whose pipeline runs PCA over the scaled dataset to visualize
    loadings, scores, and explained variance
  repo: github.com/jlichtarge/pcaGLASSO
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:359e236770e58d388e4c72dcab78096809a958f863eec8bb9efe6fa9a6a82f8e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - pandas
  - pcaglasso
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pca-input-feature-scaling

## Summary

Mean- and stddev-normalization (feature-wise mean centering and division by standard deviation) of the metabolite intensity table before PCA, as MetaboLINK performs prior to running PCA over the NMR metabolite dataset to produce 10 components.

## When to use

Use when a raw metabolite concentration/intensity table (e.g., 90 metabolites from integrated NMR peak areas across hESC→EB→Rosette→hNPC→Neuron stages) must be scaled before PCA so features contribute comparably to components.

## When NOT to use

- Input is already a normalized/scaled feature table — re-normalizing would distort the variance structure PCA relies on.
- Analysis does not involve dimensionality reduction by PCA (e.g., raw NMR peak deconvolution or Chenomx peak identification, which occur upstream of this scaling step).
- Data are non-numeric categorical labels (e.g., cell-stage markers CD24, ZO-1, Nestin, MAP2, vGLUT1 from flow cytometry) rather than quantitative metabolite intensity columns.

## Inputs

- Raw metabolite concentration/intensity table (90 metabolites from integrated NMR peak areas, Supplementary Table S1) with metabolite identifiers as rows/features
- Sample stage labels for the five neural developmental populations (hESC, EB, Rosettes, hNPC, Neurons)

## Outputs

- normalized_metabolite_table.csv — mean- and stddev-normalized matrix (metabolites as rows/features, stage samples as columns)
- Scaled input matrix ready for PCA (10 components produced downstream, first three used)

## How to apply

Load the raw metabolite concentration table with pandas (90 metabolites estimated from integrated 1H-NMR peak areas, Supplementary Table S1), retaining sample stage labels for the five cell populations (hESC, EB, Rosettes, hNPC, Neurons). Apply mean- and stddev-normalization feature-wise: subtract each metabolite column's mean and divide by its standard deviation, the preprocessing the paper specifies before PCA. Write the normalized matrix to normalized_metabolite_table.csv with metabolites as rows/features and stage samples as columns, preserving original metabolite identifiers and stage labels. Downstream, PCA is run over this mean- and stddev-normalized dataset to produce 10 components, of which the first three are used, using the PCA implementation from scikit-learn's sklearn packages. The rationale is to prevent metabolites with large raw peak-area magnitudes from dominating PCA variance during dimensionality reduction.

## Related tools

- **Python** (Language used to implement the mean- and stddev-normalization preprocessing step with pandas before PCA)
- **scikit-learn** (Provides the PCA implementation that consumes the mean- and stddev-normalized dataset (PCA run alongside GLASSO from sklearn.covariance))
- **pcaglasso** (MetaboLINK package whose pipeline runs PCA over the scaled dataset to visualize loadings, scores, and explained variance) — github.com/jlichtarge/pcaGLASSO

## Examples

```
df = pd.read_csv('metabolite_table.csv', index_col=0); df_norm = (df - df.mean()) / df.std(); df_norm.to_csv('normalized_metabolite_table.csv')
```

## Evaluation signals

- Each normalized metabolite column has mean ≈ 0 and standard deviation ≈ 1 (verify with df.mean() and df.std() on the output table).
- Output file normalized_metabolite_table.csv preserves the original 90 metabolite identifiers and the stage labels for hESC, EB, Rosettes, hNPC, and Neurons without row/column reordering.
- The number of features (metabolites) and samples (stage populations) in the normalized table matches the raw input table.
- PCA run on the scaled matrix yields 10 components, of which the first three are used — a component count different from 10 indicates the scaling or PCA call deviates from the paper's protocol.
- No NaN values are introduced by normalization (e.g., columns with zero variance would produce division-by-zero artifacts).

## Limitations

- The paper does not state whether normalization is applied per-metabolite across samples or per-sample across metabolites; feature-wise (column-wise) scaling is the conventional reading but is not explicitly disambiguated.
- No software versions, PCA parameters (e.g., solver, random state), or normalization implementation details are given in the discussion, hindering exact reproducibility.
- No accession or availability statement for the metabolomics dataset is provided, so the scaling step cannot be independently verified against the original data.

## Evidence

- [methods] We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three.: "We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three."
- [methods] We use the PCA and GLASSO implementations from scikit-learn,1 found in the sklearn.covariance packages, respectively.: "We use the PCA and GLASSO implementations from scikit-learn,1 found in the sklearn.covariance packages, respectively."
- [readme] A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks.: "A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."
