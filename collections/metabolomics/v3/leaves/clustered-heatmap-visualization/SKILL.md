---
name: clustered-heatmap-visualization
description: Use when a wide-formatted metabolomics feature table (samples as columns,
  metabolomics features as rows) needs exploratory co-clustering of samples and features
  — a hierarchical cluster heatmap with dendrograms plus a pairwise centroid-distance
  matrix.
when_to_use_negative:
- Input is raw instrument output still needing peak picking / feature identification
  — SECIMTools starts from a feature table, so preprocessing belongs upstream (e.g.
  Galaxy-M or Workflow4Metabolomics)
- The goal is significance testing or classification (ANOVA, t-test, Kruskal-Wallis,
  PLS-DA, random forest, SVM) rather than exploratory clustering — use the corresponding
  SECIMTools statistical or classification tools instead
- Missing values are present — run the Imputation tool (group mean, group median,
  or KNN) first, since distances cannot be computed over gaps
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3318
tools:
- name: SECIMTools
  role: Provides the Hierarchical Clustering tool that computes centroid-distance
    linkage over feature/sample axes and renders the heatmap with row and column dendrograms
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: Optional execution environment — the SECIMTools heatmap tool runs standalone
    or wrapped for use in Galaxy workflows
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:b58eb1f73837c6a0c1678c34f2d48056bbb6b78e1e3588321a29c7552f5b12e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# clustered-heatmap-visualization

## Summary

Renders SECIMTools' hierarchical cluster heatmap: centroid-distance linkage orders the features and samples of a wide-formatted metabolomics feature table into a two-dendrogram heatmap, alongside a tab-separated pairwise centroid-distance matrix CSV.

## When to use

Use when a wide-formatted metabolomics feature table (samples as columns, metabolomics features as rows) needs exploratory co-clustering of samples and features — a hierarchical cluster heatmap with dendrograms plus a pairwise centroid-distance matrix.

## When NOT to use

- Input is raw instrument output still needing peak picking / feature identification — SECIMTools starts from a feature table, so preprocessing belongs upstream (e.g. Galaxy-M or Workflow4Metabolomics)
- The goal is significance testing or classification (ANOVA, t-test, Kruskal-Wallis, PLS-DA, random forest, SVM) rather than exploratory clustering — use the corresponding SECIMTools statistical or classification tools instead
- Missing values are present — run the Imputation tool (group mean, group median, or KNN) first, since distances cannot be computed over gaps

## Inputs

- Wide-formatted metabolomics feature table (samples as columns, metabolomics features as rows)
- Companion design file carrying the grouping variable used for sample coloring

## Outputs

- Tab-separated CSV of the pairwise centroid-distance matrix, with feature/sample identifiers as row and column labels
- Hierarchical clustering heatmap image (hierarchical_clustering_heatmap.png) with row and column dendrograms

## How to apply

Load the wide-formatted feature table — samples as columns, metabolomics features as rows — together with its companion design file holding the grouping variable. Pass the table into the SECIMTools Hierarchical Clustering tool, which computes centroid-distance linkage over both the feature and sample axes; this ordered linkage feeds both the dendrograms and the pairwise distances. Write the centroid-distance matrix as a tab-separated CSV with feature/sample identifiers as row and column labels, exploiting SECIMTools' standard data formats so the CSV can be linked into further tools. Render the heatmap with row and column dendrograms, using the design-file grouping variable for sample coloring, and save it as hierarchical_clustering_heatmap.png. Run standalone via the secimtools PyPI package or wrapped for Galaxy; the code is MIT-licensed on GitHub.

## Related tools

- **SECIMTools** (Provides the Hierarchical Clustering tool that computes centroid-distance linkage over feature/sample axes and renders the heatmap with row and column dendrograms) — https://github.com/SECIMTools
- **Galaxy** (Optional execution environment — the SECIMTools heatmap tool runs standalone or wrapped for use in Galaxy workflows)

## Examples

```
pip install secimtools
```

## Evaluation signals

- The clustering script runs end-to-end without error
- Both outputs exist: the distance-matrix CSV and the heatmap PNG/PDF
- The distance-matrix CSV contains an identifier column plus numeric pairwise distance fields
- The heatmap PNG/PDF is a non-empty image showing both row and column dendrograms
- Row/column labels of the distance matrix match the feature/sample identifiers of the wide-formatted input table

## Limitations

- The README documents only installation (PyPI, bioconda, Galaxy packaging) and motivation — no CLI flags, defaults, or heatmap parameters, so exact settings must come from the tool's own help
- No SECIMTools version is pinned in the article's discussion (the GitHub reference is not resolved to a URL in the text), so reproduce against a fixed release
- Sample coloring depends on the design-file grouping variable; without a design file the heatmap carries no group annotation

## Evidence

- [abstract] visualization techniques (hierarchical cluster heatmap, principal component analysis, modular modularity clustering): "visualization techniques (hierarchical cluster heatmap, principal component analysis, modular modularity clustering)"
- [abstract] Standard data formats and utilities allow arbitrary linkages between tools for novel workflow designs: "Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs."
- [methods] SECIMTools open source, MIT license, GitHub: "SECIMTools is open source, the code, is available on GitHub using the MIT license"
- [readme] Standalone Python tools, also wrapped for Galaxy: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
