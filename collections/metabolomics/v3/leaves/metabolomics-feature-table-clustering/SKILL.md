---
name: metabolomics-feature-table-clustering
description: Use when a wide formatted metabolomics feature table (samples as columns,
  features as rows) plus a companion design file need unsupervised hierarchical clustering
  with dendrograms and pairwise distance output.
when_to_use_negative:
- Input is a peak-picking-stage instrument file rather than an already-built wide
  feature table — run preprocessing (blank feature filtering, imputation) first
- The goal is supervised analysis (PLS-DA, random forest, SVM) rather than unsupervised
  visualization — use SECIMTools' statistical/classification tools instead
- Sample-level distance comparison within groups is the goal — the Standardized Euclidean
  Distance (SED) tool is the article's dedicated option for that
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: SECIMTools
  role: Provides the hierarchical cluster heatmap tool that computes centroid-distance
    linkage over feature/sample axes and renders the dendrogram-annotated heatmap
  repo: https://github.com/SECIMTools
- name: Python
  role: Language of the SECIMTools package; tools run as Python applications in standalone
    mode
  repo: https://pypi.python.org/pypi/secimtools
- name: Galaxy
  role: Alternative execution mode — SECIMTools tools are wrapped for use in Galaxy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:855471020d23511970224b7b7dd1bd3d08ffe4cdacbc3ac5237b56e30a4b456a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metabolomics-feature-table-clustering

## Summary

Hierarchical cluster heatmap analysis of a wide-format metabolomics feature table (samples as columns, features as rows) using SECIMTools, which computes centroid-distance linkage and renders row/column dendrograms with design-file-based sample coloring.

## When to use

Use when a wide formatted metabolomics feature table (samples as columns, features as rows) plus a companion design file need unsupervised hierarchical clustering with dendrograms and pairwise distance output.

## When NOT to use

- Input is a peak-picking-stage instrument file rather than an already-built wide feature table — run preprocessing (blank feature filtering, imputation) first
- The goal is supervised analysis (PLS-DA, random forest, SVM) rather than unsupervised visualization — use SECIMTools' statistical/classification tools instead
- Sample-level distance comparison within groups is the goal — the Standardized Euclidean Distance (SED) tool is the article's dedicated option for that

## Inputs

- Wide formatted feature table (samples as columns, metabolomics features as rows)
- Companion design file containing a sample grouping variable for coloring

## Outputs

- Hierarchical clustering heatmap image (e.g. hierarchical_clustering_heatmap.png/PDF) with row and column dendrograms
- Pairwise centroid-distance matrix as tab-separated CSV with feature/sample identifiers as row and column labels

## How to apply

Load the wide formatted table and its companion design file, then pass the table into the SECIMTools Hierarchical Clustering tool, which computes centroid-distance linkage over the feature (and sample) axes. Run the clustering to obtain the ordered linkage used for both the heatmap dendrograms and the pairwise distance values. Write the pairwise centroid-distance matrix as a tab-separated CSV with feature/sample identifiers as row and column labels, then render the heatmap with row and column dendrograms, using the design-file grouping variable for sample coloring. Save the figure (e.g. hierarchical_clustering_heatmap.png/PDF). SECIMTools is a set of Python applications usable standalone or wrapped in Galaxy, so the same tool can be invoked either way.

## Related tools

- **SECIMTools** (Provides the hierarchical cluster heatmap tool that computes centroid-distance linkage over feature/sample axes and renders the dendrogram-annotated heatmap) — https://github.com/SECIMTools
- **Python** (Language of the SECIMTools package; tools run as Python applications in standalone mode) — https://pypi.python.org/pypi/secimtools
- **Galaxy** (Alternative execution mode — SECIMTools tools are wrapped for use in Galaxy)

## Evaluation signals

- The clustering script runs without error
- Both output files (heatmap image and distance-matrix CSV) exist after the run
- The distance-matrix CSV contains an identifier column plus numeric pairwise distance fields
- The heatmap PNG/PDF is a non-empty image and includes row and column dendrograms
- Row/column labels of the distance matrix match the feature and sample identifiers from the input wide table

## Limitations

- The README and article give no version number for SECIMTools and no CLI parameter/threshold details for the clustering tool; invocation must be derived from the installed package
- Linkage type is fixed by the tool (centroid-distance linkage); no alternative linkage options are described in the provided context
- No release/package version is resolved in the discussion text (GitHub reference not resolved to a URL in the article body)

## Evidence

- [abstract] visualization techniques (hierarchical cluster heatmap, principal component analysis, modular modularity clustering): "visualization techniques (hierarchical cluster heatmap, principal component analysis, modular modularity clustering)"
- [readme] The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy.: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
- [methods] SECIMTools is open source, the code, is available on GitHub using the MIT license: "SECIMTools is open source, the code, is available on GitHub using the MIT license"
