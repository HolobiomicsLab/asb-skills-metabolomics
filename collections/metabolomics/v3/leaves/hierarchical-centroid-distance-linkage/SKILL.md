---
name: hierarchical-centroid-distance-linkage
description: Decision support for clustering a wide-formatted metabolomics feature
  table (samples as columns, metabolomics features as rows) to visualize group structure
  via a clustered heatmap with dendrograms.
when_to_use_negative:
- Input is already clustered, or the analysis goal is supervised classification (e.g.
  PLS-DA, random forest, SVM) rather than unsupervised structure visualization
- The data are raw vendor peak lists that still need peak picking or blank-feature
  filtering before a wide formatted table exists
- Only pairwise distance values are needed without heatmap rendering — the standalone
  distance computation may suffice
edam_operation: http://edamontology.org/operation_0558
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SECIMTools
  role: provides the hierarchical clustering tool (standalone Python or Galaxy-wrapped)
    that computes centroid-distance linkage and renders the clustered heatmap
  repo: https://github.com/SECIMTools
- name: Python
  role: runtime for the SECIMTools scripts; the suite is a set of Python applications
    available as standalone tools
  repo: https://pypi.python.org/pypi/secimtools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:88e01561068cc0e8ca2a574c73757912828a756ba2cad1c0b2ca79759680794f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_0558
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hierarchical-centroid-distance-linkage

## Summary

Apply SECIMTools hierarchical clustering to a wide-format metabolomics feature table, computing centroid-distance linkage that yields both row/column dendrograms and a pairwise distance matrix. This skill supports exploratory structure discovery in metabolomics feature data.

## When to use

Decision support for clustering a wide-formatted metabolomics feature table (samples as columns, metabolomics features as rows) to visualize group structure via a clustered heatmap with dendrograms.

## When NOT to use

- Input is already clustered, or the analysis goal is supervised classification (e.g. PLS-DA, random forest, SVM) rather than unsupervised structure visualization
- The data are raw vendor peak lists that still need peak picking or blank-feature filtering before a wide formatted table exists
- Only pairwise distance values are needed without heatmap rendering — the standalone distance computation may suffice

## Inputs

- wide formatted feature table (samples as columns, metabolomics features as rows)
- companion design file with grouping variables
- pairwise centroid-distance values computed from the linkage

## Outputs

- pairwise centroid-distance matrix as a tab-separated CSV with feature/sample identifiers as row and column labels
- hierarchical clustering heatmap plot (hierarchical_clustering_heatmap.png) with row and column dendrograms, sample coloring by design-file grouping variable

## How to apply

Load the bundled SECIMTools example wide formatted table and its companion design file (samples as columns, metabolomics features as rows). Run the SECIMTools Hierarchical Clustering tool, which computes centroid-distance linkage over the feature and sample axes; use the design-file grouping variable for sample coloring in the heatmap. Write the pairwise centroid-distance matrix as a tab-separated CSV with identifiers as row/column labels, and render the heatmap as hierarchical_clustering_heatmap.png with row and column dendrograms. SECIMTools ships this as a standalone Python tool or wrapped for Galaxy, so it can be invoked either way; the MIT-licensed code is on GitHub and installable from PyPI/bioconda.

## Related tools

- **SECIMTools** (provides the hierarchical clustering tool (standalone Python or Galaxy-wrapped) that computes centroid-distance linkage and renders the clustered heatmap) — https://github.com/SECIMTools
- **Python** (runtime for the SECIMTools scripts; the suite is a set of Python applications available as standalone tools) — https://pypi.python.org/pypi/secimtools

## Examples

```
python secimtools/hierarchicalClustering.py --input example_feature_table.csv --design example_design.csv --output_dir . --fig hierarchical_clustering_heatmap.png
```

## Evaluation signals

- The SECIMTools hierarchical clustering script runs without error on the wide formatted table and design file
- Both output files exist: the distance-matrix CSV and hierarchical_clustering_heatmap.png
- The distance-matrix CSV contains an identifier column plus numeric pairwise distance fields
- The heatmap PNG/PDF is a non-empty image file
- Row and column dendrograms are present in the rendered figure, with samples colored by the design-file grouping variable

## Limitations

- The article does not specify a SECIMTools or Galaxy version number, nor resolve the GitHub reference to a URL in the text
- Centroid-distance linkage is one choice among available linkage methods; the article does not enumerate alternatives or tuning guidance
- Untargeted metabolomics has no single platform capturing the entire metabolome, so feature content and distance scale vary across platforms

## Evidence

- [abstract] SECIMTools includes hierarchical cluster heatmap among visualization techniques alongside PCA and modularity clustering: "visualization techniques (hierarchical cluster heatmap, principal component analysis, modular modularity clustering)"
- [intro] SECIMTools runs standalone or in Galaxy: "SECIMTools project aims to develop a suite of tools for processing of metabolomics data, which can be run in a standalone mode or via Galaxy"
- [methods] SECIMTools code is on GitHub under MIT license: "SECIMTools is open source, the code, is available on GitHub using the MIT license"
- [readme] README confirms hierarchical cluster heatmap as a visualization technique: "visualization techniques (hierarchical cluster heatmap, principal component analysis, linear discriminant analysis, modular modularity clustering)"
