---
name: metabolite-concentration-table-normalization
description: Use when a raw metabolite intensity/concentration table (e.g., 90 metabolites
  quantified by Chenomx from 1H-NMR spectra of hESC, EB, rosette, hNPC, and neuron
  samples) must be scaled before PCA.
when_to_use_negative:
- The input is already z-score normalized or the downstream method (e.g., Pattern
  Hunter in MetaboAnalyst) requires raw concentration units rather than scaled features.
- The goal is raw spectra preprocessing (TopSpin/Chenomx peak identification) rather
  than scaling an already-quantified metabolite table.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: pandas
  role: Loads the raw metabolite table and applies feature-wise mean centering and
    standard-deviation scaling
- name: Python
  role: Execution environment for the normalization script feeding the MetaboLINK
    pipeline
  repo: github.com/jlichtarge/pcaGLASSO
- name: MetaboLINK (PCA-GLASSO algorithm)
  role: Downstream consumer of the normalized dataset; its PCA step requires the mean-
    and stddev-normalized input
  repo: github.com/jlichtarge/pcaGLASSO
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:52ef9985a1b640d6bed0871d06a9223b668ab94a4d190712e8f1f1b7a8f2538b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboLINK (PCA-GLASSO algorithm)
  - Python
  - matplotlib
  - pandas
  - pcaglasso
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metabolite-concentration-table-normalization

## Summary

Feature-wise mean- and stddev-normalization (z-score scaling) of the 90-metabolite NMR concentration table from hESC-derived neural populations, applied as the mandatory preprocessing step before MetaboLINK's PCA-GLASSO analysis.

## When to use

Use when a raw metabolite intensity/concentration table (e.g., 90 metabolites quantified by Chenomx from 1H-NMR spectra of hESC, EB, rosette, hNPC, and neuron samples) must be scaled before PCA.

## When NOT to use

- The input is already z-score normalized or the downstream method (e.g., Pattern Hunter in MetaboAnalyst) requires raw concentration units rather than scaled features.
- The goal is raw spectra preprocessing (TopSpin/Chenomx peak identification) rather than scaling an already-quantified metabolite table.

## Inputs

- Raw metabolite concentration table (n = 90 metabolites from Supplementary Table S1, quantified by Chenomx from 1H-NMR spectra acquired on a Bruker Avance 800 MHz spectrometer)
- Sample stage labels for the five cell populations (hESC, EB, Rosettes, hNPC, Neurons)

## Outputs

- Mean- and stddev-normalized metabolite matrix written to normalized_metabolite_table.csv with metabolites as rows/features and stage samples as columns
- Preserved metabolite identifiers and stage labels for downstream PCA-GLASSO (MetaboLINK) analysis

## How to apply

Load the raw metabolite concentration table (90 metabolites estimated from integrated NMR peak areas; 3–8 biological replicates per stage) with pandas, retaining stage labels for the five cell populations. Apply feature-wise mean centering followed by division by each metabolite's standard deviation, matching the paper's preprocessing in which 'PCA [is run] over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three.' Preserve metabolite identifiers and stage labels, then write the normalized matrix (metabolites as rows/features, stage samples as columns) to normalized_metabolite_table.csv. This scaling is a prerequisite for the downstream MetaboLINK pca_bundle PCA and GLASSO network steps, which assume comparable variance across metabolite features.

## Related tools

- **pandas** (Loads the raw metabolite table and applies feature-wise mean centering and standard-deviation scaling)
- **Python** (Execution environment for the normalization script feeding the MetaboLINK pipeline) — github.com/jlichtarge/pcaGLASSO
- **MetaboLINK (PCA-GLASSO algorithm)** (Downstream consumer of the normalized dataset; its PCA step requires the mean- and stddev-normalized input) — github.com/jlichtarge/pcaGLASSO

## Examples

```
df_norm = (df - df.mean()) / df.std(); df_norm.to_csv('normalized_metabolite_table.csv')
```

## Evaluation signals

- Each normalized metabolite column has mean ≈ 0 and standard deviation ≈ 1 (within floating-point tolerance).
- Row and column counts match the input table (90 metabolites; 3–8 replicates per population across hESC, EB, rosettes, hNPC, neurons).
- Original metabolite identifiers and stage labels are unchanged relative to the raw table.
- normalized_metabolite_table.csv parses cleanly in pandas and downstream PCA (10 components, first three retained) runs without scaling artifacts dominating loadings.
- Subsequent PCA score plots separate the five cell-type groups, consistent with the paper's reported result.

## Limitations

- The paper states no software versions or explicit implementation parameters for the normalization or downstream PCA/GLASSO steps.
- Cross-validation of GLASSO alpha is unstable on the small pairwise datasets (per-transition subsetting), so scaling choices interact with downstream regularization settings.
- No accession or availability statement is given for the metabolomics dataset itself, limiting direct reproduction of the exact input table.

## Evidence

- [methods] PCA over mean- and stddev-normalized dataset: "We ran PCA over the mean- and stddev-normalized dataset to produce 10 components, of which we use the first three."
- [methods] scikit-learn implementations: "We use the PCA and GLASSO implementations from scikit-learn,1 found in the sklearn.covariance packages, respectively."
- [readme] pcaglasso package description: "A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."
