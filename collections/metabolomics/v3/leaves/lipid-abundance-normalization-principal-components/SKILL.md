---
name: lipid-abundance-normalization-principal-components
description: Use when comparing multiple heterogeneous lipidomes where raw abundances
  must be added to a structural (PCA-based) lipid space model without low-abundant
  lipid species dominating the structural comparison.
when_to_use_negative:
- When only qualitative presence/absence structural overlap is of interest and quantitative
  influence should be excluded — abundance addition can be switched off in LipidSpace
- When analyzing a single lipidome with no cross-lipidome comparison, since the normalization
  targets relating multiple lipid space models to each other
- When lipid names cannot be standardized (unparseable shorthand notation), since
  Goslin-based parsing must succeed before model construction
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: LipidSpace
  role: Standalone workstation application that builds the lipid space model per lipidome,
    normalizes abundances to first-PC variance, and adds them to the model (quantities
    included by default)
  repo: https://github.com/lifs-tools/lipidspace
- name: lambda-lanczos
  role: Lanczos method implementation used to compute only the first n' < n principal
    components of the lipid distance matrix
  repo: https://github.com/mrcdr/lambda-lanczos
- name: cppgoslin
  role: Parses and standardizes lipid names from heterogeneous lipidomics tables prior
    to model construction
  repo: https://github.com/lifs-tools/cppgoslin
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:bda8993b5fa17c2763e3e13a8a8dbad52371dc937e89da5ee28a3003c4652b23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - LipidSpace
  - cppgoslin
  - lambda-lanczos
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lipid-abundance-normalization-principal-components

## Summary

Normalizes lipidome abundances with respect to the variance of the first principal component of each lipidome's lipid space model before integrating them into a shared structural space, so that quantitative information rather than lipid presence/absence drives lipidome comparisons.

## When to use

Use when comparing multiple heterogeneous lipidomes where raw abundances must be added to a structural (PCA-based) lipid space model without low-abundant lipid species dominating the structural comparison.

## When NOT to use

- When only qualitative presence/absence structural overlap is of interest and quantitative influence should be excluded — abundance addition can be switched off in LipidSpace
- When analyzing a single lipidome with no cross-lipidome comparison, since the normalization targets relating multiple lipid space models to each other
- When lipid names cannot be standardized (unparseable shorthand notation), since Goslin-based parsing must succeed before model construction

## Inputs

- Multi-lipidomics dataset as csv/xlsx flat or pivot tables or mzTab-M files (lipid names parsed/standardized via Goslin)
- Per-lipidome abundance matrix (lipid species x samples)
- Lipid structural distance matrix from graph-based (MCS/Jaccard) comparisons

## Outputs

- Per-lipidome lipid space model with abundance values normalized to the variance of the first PC
- Global lipid space model connecting all lipidomes for downstream Hausdorff distance computation

## How to apply

After computing pairwise lipid-lipid structural distances (maximum common subgraph via LaWeCSE plus Jaccard index), perform a PCA on the n x n distance matrix using the Lanczos method (default 7 principal components, openBLAS-accelerated) to build the lipid space model for one lipidome. Normalize each lipidome's lipid abundances concerning the variance of the first principal component of its model before adding them to the model; this quantity-inclusion is on by default and can be switched off. Rationale: without this normalization, structural comparison models are heavily influenced by low abundant lipid species that do or do not appear across samples. The augmented models then feed into pairwise Hausdorff distance computations across the m lipidomes.

## Related tools

- **LipidSpace** (Standalone workstation application that builds the lipid space model per lipidome, normalizes abundances to first-PC variance, and adds them to the model (quantities included by default)) — https://github.com/lifs-tools/lipidspace
- **lambda-lanczos** (Lanczos method implementation used to compute only the first n' < n principal components of the lipid distance matrix) — https://github.com/mrcdr/lambda-lanczos
- **cppgoslin** (Parses and standardizes lipid names from heterogeneous lipidomics tables prior to model construction) — https://github.com/lifs-tools/cppgoslin

## Examples

```
LipidSpace REST: POST examples/Rest/Plasma-Singapore-Short.json to /lipidspace/v1/pca with quantities enabled (default), yielding the lipid space models whose first-PC variance normalizes abundances
```

## Evaluation signals

- The first principal component of each lipidome's PCA (default 7 PCs via Lanczos on the n x n lipid distance matrix) yields a finite positive variance used as the normalization denominator
- Lipidome distance matrix must be symmetric with dist(A,B) = dist(B,A) and a zero diagonal (reference value 1.13403 between the first two lipidomes of the bundled example dataset)
- LipidSpace model distances remain bounded in [0, 1] when the default bound distance metric is used
- Comparing runs with quantities on vs off should shift Hausdorff-derived lipidome similarities, confirming abundances were actually incorporated
- Reanalysis of reference datasets (e.g. the bundled Plasma-Singapore-Short example) reproduces the reference REST response values

## Limitations

- Quantitative information added without normalization would heavily influence the structural comparison model via low abundant lipid species; normalization to first-PC variance is required
- Comparing all combinations of fatty acyl chains when sn-positions are unspecified (e.g. PC 18:0_16:1) is more accurate but decreases performance
- LipidSpace cannot currently store or load an analysis, so the normalization workflow must be rerun per session

## Evidence

- [methods] abundance normalization to first PC: "the abundances for a lipidome are normalized concerning the variance of the first PC of its model and added to the model. Applying pairwise distance computations on m lipidomes"
- [methods] PCA via Lanczos on distance matrix: "we perform a principal component analysis (PCA) on the complete matrix. We decided to use an implementation of the Lanczos method17"
- [intro] rationale for quantitative integration: [evidence span withheld — anchor status: not_found]
- [readme] distance metric bounded default: "the distance is a value that ranges between 0 (both lipids are identical) and 1. However, other distance measures suggest an unbound distance ranging from 0 to infinity"
- [readme] sn-position comparison mode: "when the sn-position is not specified as for instance in PC 18:0_16:1, a mode can be activated to compare all combinations of FA comparisons"
