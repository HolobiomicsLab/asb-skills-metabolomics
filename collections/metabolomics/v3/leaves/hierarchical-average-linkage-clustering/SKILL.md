---
name: hierarchical-average-linkage-clustering
description: Use when m ≥ 3 lipidomes have been placed in a common structural space
  and you need a global lipidome-level relationship model (dendrogram) from their
  pairwise Hausdorff distances.
when_to_use_negative:
- The goal is feature analysis of study variables (e.g. which lipids distinguish BMI
  or condition) — use the feature analysis/SFS module instead of dendrogram clustering.
- The input is raw mass spectrometry spectra rather than an imported lipid abundance
  table (csv/xlsx flat or pivot, or mzTab-M); LipidSpace consumes processed lipidome
  tables with Goslin-parsed lipid names.
- Only a single lipidome is analyzed — no pairwise lipidome distances exist, so no
  dendrogram can be built.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipidSpace
  role: Performs the pairwise Hausdorff distance computation over lipidome models
    and the agglomerative hierarchical clustering (unweighted average default; single/complete
    selectable) with dendrogram rendering and PDF export.
  repo: https://github.com/lifs-tools/lipidspace
- name: lambda-lanczos
  role: 'Upstream step: computes the first n'' < n principal components of the lipid
    distance matrix, defining the structural space in which lipidome Hausdorff distances
    are measured before clustering.'
  repo: https://github.com/mrcdr/lambda-lanczos
- name: cppgoslin
  role: Parses and standardizes lipid names on import so that structural comparison
    and the resulting distance matrix are computed over consistently represented lipid
    species.
  repo: https://github.com/lifs-tools/cppgoslin
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:233386a169ed68bbf378aea309a4458408aa0532fd6c8351c914125bccec7caf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - LipidSpace
  - cppgoslin
  - lambda-lanczos
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hierarchical-average-linkage-clustering

## Summary

Agglomerative hierarchical clustering of lipidomes using the LipidSpace-derived symmetric lipidome distance (Hausdorff) matrix, rendered as a dendrogram that expresses the global structural relationship of all input lipidomes.

## When to use

Use when m ≥ 3 lipidomes have been placed in a common structural space and you need a global lipidome-level relationship model (dendrogram) from their pairwise Hausdorff distances.

## When NOT to use

- The goal is feature analysis of study variables (e.g. which lipids distinguish BMI or condition) — use the feature analysis/SFS module instead of dendrogram clustering.
- The input is raw mass spectrometry spectra rather than an imported lipid abundance table (csv/xlsx flat or pivot, or mzTab-M); LipidSpace consumes processed lipidome tables with Goslin-parsed lipid names.
- Only a single lipidome is analyzed — no pairwise lipidome distances exist, so no dendrogram can be built.

## Inputs

- m lipidome models with lipid abundances normalized to the variance of the first principal component (from the shared lipid structural space)
- m × m symmetric LipidomeDistanceMatrix of pairwise Hausdorff distances (zero diagonal) computed over the union set of lipid species
- multi-lipidome dataset, e.g. csv/xlsx pivot or flat tables or mzTab-M (example: examples/Rest/Plasma-Singapore-Short.json)

## Outputs

- interactive dendrogram plot encoding the global lipidome distance model
- dendrogram exported in PDF format
- LipidomeDistanceMatrix values (e.g. 1.13403 between the first two lipidomes of the example dataset)

## How to apply

After LipidSpace builds each per-lipidome model — PCA on the n × n lipid distance matrix computed with the Lanczos method (default 7 principal components, openBLAS-accelerated), abundances normalized to the variance of the first PC — compute all pairwise Hausdorff distances between the m lipidomes over the union lipid set, yielding an m × m symmetric matrix with zero diagonal. Apply agglomerative hierarchical clustering to this distance matrix with unweighted average linkage (default); single linkage and complete linkage are selectable alternatives via the menu Analysis → Clustering strategy. Optionally restrict to a subcluster by right-clicking a dendrogram branch and choosing 'Select these lipidomes in sample selection', then restart the analysis. Render the result as an interactive dendrogram and export it in PDF format as the global lipidome distance model.

## Related tools

- **LipidSpace** (Performs the pairwise Hausdorff distance computation over lipidome models and the agglomerative hierarchical clustering (unweighted average default; single/complete selectable) with dendrogram rendering and PDF export.) — https://github.com/lifs-tools/lipidspace
- **lambda-lanczos** (Upstream step: computes the first n' < n principal components of the lipid distance matrix, defining the structural space in which lipidome Hausdorff distances are measured before clustering.) — https://github.com/mrcdr/lambda-lanczos
- **cppgoslin** (Parses and standardizes lipid names on import so that structural comparison and the resulting distance matrix are computed over consistently represented lipid species.) — https://github.com/lifs-tools/cppgoslin

## Examples

```
curl -X POST --data-binary @examples/Rest/Plasma-Singapore-Short.json http://localhost:8080/lipidspace/v1/pca
```

## Evaluation signals

- The LipidomeDistanceMatrix is symmetric with dist(A,B) = dist(B,A) and a zero diagonal.
- Matrix values match the reference REST response for the example dataset, e.g. 1.13403 between the first two lipidomes.
- The dendrogram topology changes appropriately when switching linkage in Analysis → Clustering strategy (single, unweighted average, complete).
- The dendrogram exports successfully as a PDF and branch selection ('Select these lipidomes in sample selection') reproduces the corresponding subset for reanalysis.

## Limitations

- Analysis state cannot be saved or loaded ('At the moment it is not possible to store or load an analysis.'), so dendrogram settings must be reconfigured per session.
- The unbound distance metric mode gives more accurate results but reduces the visibility of the lipid spaces since distances may become very big.
- Ignoring lipid sn-positions (all-combination fatty acyl comparison) is more accurate but decreases performance.

## Evidence

- [methods] the abundances for a lipidome are normalized concerning the variance of the first PC of its model and added to the model. Applying pairwise distance computations on m lipidomes: "the abundances for a lipidome are normalized concerning the variance of the first PC of its model and added to the model. Applying pairwise distance computations on m lipidomes"
- [abstract] A graph-based comparison of lipid structures is the basis for calculating structural space models and subsequently computing lipidome similarities.: "A graph-based comparison of lipid structures is the basis for calculating structural space models and subsequently computing lipidome similarities."
- [readme] The user can switch between single linkage, unweighted average, and complete linkage clustering in the menu → Analysis → Clustering strategy.: "The user can switch between single linkage, unweighted average, and complete linkage clustering in the menu → Analysis → Clustering strategy."
- [readme] Go to the dendrogram view and right-click on any branch which you are interested. In the context menu choose 'Select these lipidomes in sample selection': "Go to the dendrogram view and right-click on any branch which you are interested. In the context menu choose 'Select these lipidomes in sample selection'"
