---
name: reverse-dependency-network-construction
description: Use when a catalogued list of R packages (e.g. metabolomics packages
  from CRAN, Bioconductor and GitHub) is available and you need to know which other
  packages depend on them, to compare connectivity across repositories.
when_to_use_negative:
- The package catalogue has not yet been built (this skill consumes the task_001 catalogue
  of metabolomics R packages, it does not produce it)
- You need forward dependencies of a single package rather than the reverse-dependency
  network over a whole package catalogue
- You are interested in GitHub-only packages with no CRAN or Bioconductor metadata,
  since edge extraction relies on CRAN available.packages() and Bioconductor metadata
  dumps
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3173
tools:
- name: R
  role: Language in which metadata fields are parsed and the igraph graph is assembled
    and exported
- name: CRAN
  role: Source of package metadata via available.packages() for the reverse-dependency
    edge extraction
- name: Bioconductor
  role: Source of biocViews/package metadata dumps for the reverse-dependency edge
    extraction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:0433b4cdc4367cea83352e7f7d7cdcbeb2e4ae1e279e6af068642d13f9194400
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - CRAN
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# reverse-dependency-network-construction

## Summary

Builds the reverse-dependency network of the metabolomics R packages catalogued in the metaRbolomics review, by parsing Depends/Imports/LinkingTo fields from CRAN and Bioconductor metadata and keeping edges that point at catalogued packages. It quantifies the relative connectivity of CRAN versus Bioconductor packages, which the review attributes to Bioconductor's reuse-friendly policy.

## When to use

Use when a catalogued list of R packages (e.g. metabolomics packages from CRAN, Bioconductor and GitHub) is available and you need to know which other packages depend on them, to compare connectivity across repositories.

## When NOT to use

- The package catalogue has not yet been built (this skill consumes the task_001 catalogue of metabolomics R packages, it does not produce it)
- You need forward dependencies of a single package rather than the reverse-dependency network over a whole package catalogue
- You are interested in GitHub-only packages with no CRAN or Bioconductor metadata, since edge extraction relies on CRAN available.packages() and Bioconductor metadata dumps

## Inputs

- Package catalogue of metabolomics R packages from CRAN, Bioconductor and GitHub (output of the catalogue task)
- CRAN package metadata from available.packages()
- Bioconductor biocViews/package metadata dumps
- Depends/Imports/LinkingTo fields of CRAN and Bioconductor packages

## Outputs

- Directed reverse-dependency graph written as a GraphML file
- Flat edge-list CSV (source package, target metabolomics package, dependency field)
- Edge count and per-repository (CRAN vs Bioconductor) connectivity comparison as reported in Figure 2

## How to apply

Start from the package catalogue of metabolomics R packages from CRAN, Bioconductor and GitHub produced in the preceding catalogue task. Fetch public package metadata with available.packages() for CRAN and the biocViews/package metadata dumps for Bioconductor, then parse the Depends/Imports/LinkingTo fields of every package and retain only edges whose dependency target appears in the catalogue. Assemble the retained edges into a directed graph with igraph (R or Python) and export it both as GraphML and as a flat edge-list CSV recording source package, target metabolomics package, and dependency field. The rationale, per the metaRbolomics review, is that Bioconductor packages show higher connectivity because the Bioconductor policy encourages package authors to reuse existing material and write packages that work better together.

## Related tools

- **R** (Language in which metadata fields are parsed and the igraph graph is assembled and exported)
- **CRAN** (Source of package metadata via available.packages() for the reverse-dependency edge extraction)
- **Bioconductor** (Source of biocViews/package metadata dumps for the reverse-dependency edge extraction)

## Examples

```
library(igraph); g <- graph_from_edgelist(as.matrix(edges[,c('source','target')]), directed=TRUE); write.graph(g, 'revdeps.graphml', format='graphml')
```

## Evaluation signals

- The GraphML file parses as valid graphml format (e.g. with igraph read.graph(format='graphml'))
- The flat edge-list CSV contains exactly the same edges as the GraphML file
- The total edge count is greater than zero
- Every edge target is a package present in the input metabolomics package catalogue
- Per-repository connectivity summary reproduces the Figure 2 pattern: Bioconductor packages show higher connectivity than CRAN packages

## Limitations

- Edges are only recoverable for packages with CRAN or Bioconductor metadata; packages hosted solely on GitHub may be missing from the dependency fields parsed
- Only Depends/Imports/LinkingTo fields are considered; Suggests and other dependency fields are excluded, so the network may undercount true reuse
- Connectivity differences are descriptive of repository policy (Bioconductor encourages reuse) rather than a causal quality measure

## Evidence

- [abstract] In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub.: "In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub."
- [results] Bioconductor packages have higher connectivity than CRAN packages: [evidence span withheld — anchor status: not_found]
