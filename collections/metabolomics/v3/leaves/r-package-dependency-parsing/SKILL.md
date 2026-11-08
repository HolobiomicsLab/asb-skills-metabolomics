---
name: r-package-dependency-parsing
description: Use when a catalogue of R metabolomics packages (from CRAN, Bioconductor
  and GitHub) is available and you need to quantify how other packages depend on them,
  e.g. to reproduce a reverse-dependency network figure.
when_to_use_negative:
- The input is a single package's DESCRIPTION file with no surrounding catalogue to
  restrict which dependency targets count
- The goal is forward dependencies of the metabolomics packages themselves rather
  than reverse dependencies (packages that depend on them)
- The packages are not R packages distributed via CRAN/Bioconductor (e.g. Python PyPI
  ecosystem metadata)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Environment for fetching available.packages() metadata, parsing Depends/Imports/LinkingTo
    fields, and building the graph
- name: CRAN
  role: Source of public package metadata via available.packages()
- name: Bioconductor
  role: Source of package metadata via biocViews/package metadata dumps for reverse-dependency
    edges
- name: igraph
  role: Assembles the retained edges into a directed graph and writes it out as GraphML
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:5e2d5da133b3ed68a2c999ec5b5666af8bbfda1b0c21dd79056ebdd627f4d8f5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - CRAN
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# R package dependency parsing

## Summary

Parses Depends/Imports/LinkingTo fields from CRAN and Bioconductor package metadata to build the reverse-dependency network of the metabolomics R packages catalogued in the metaRbolomics review, revealing the relative connectivity of CRAN versus Bioconductor packages.

## When to use

Use when a catalogue of R metabolomics packages (from CRAN, Bioconductor and GitHub) is available and you need to quantify how other packages depend on them, e.g. to reproduce a reverse-dependency network figure.

## When NOT to use

- The input is a single package's DESCRIPTION file with no surrounding catalogue to restrict which dependency targets count
- The goal is forward dependencies of the metabolomics packages themselves rather than reverse dependencies (packages that depend on them)
- The packages are not R packages distributed via CRAN/Bioconductor (e.g. Python PyPI ecosystem metadata)

## Inputs

- Metabolomics R package catalogue (CRAN, Bioconductor and GitHub packages, e.g. from task_001)
- CRAN package metadata from available.packages()
- Bioconductor package metadata dumps / biocViews listings
- Depends, Imports and LinkingTo description fields of each package

## Outputs

- Directed reverse-dependency graph as a GraphML file
- Flat edge-list CSV with columns: source package, target metabolomics package, dependency field
- Reverse-dependency edge count (e.g. Figure 2 statistic)

## How to apply

Start from the package catalogue produced in the preceding step (the list of more than two hundred metabolomics R packages primarily available on CRAN, Bioconductor and GitHub). Fetch public metadata with available.packages() for CRAN and the biocViews/package metadata dumps for Bioconductor. Parse the Depends/Imports/LinkingTo fields of every CRAN and Bioconductor package and retain only edges whose dependency target is a package in the catalogue, yielding the reverse-dependency edge list. Assemble the edges into a directed graph (e.g. with igraph in R or Python) and export it as GraphML plus a flat edge-list CSV (source package, target metabolomics package, dependency field). The rationale, grounded in the review's finding, is that Bioconductor packages show higher connectivity because Bioconductor policy encourages authors to reuse existing material and write packages that work better together.

## Related tools

- **R** (Environment for fetching available.packages() metadata, parsing Depends/Imports/LinkingTo fields, and building the graph)
- **CRAN** (Source of public package metadata via available.packages())
- **Bioconductor** (Source of package metadata via biocViews/package metadata dumps for reverse-dependency edges)
- **igraph** (Assembles the retained edges into a directed graph and writes it out as GraphML)

## Examples

```
pkgs <- available.packages(repos="https://cloud.r-project.org"); g <- graph_from_edgelist(edges, directed=TRUE); write.graph(g, "revdep.graphml", format="graphml")
```

## Evaluation signals

- GraphML file parses as valid GraphML format (e.g. read back with igraph::read_graph)
- CSV edge list contains exactly the same edges as the GraphML graph
- Edge count is greater than zero
- Every retained edge's target appears in the metabolomics package catalogue
- Retained edges only originate from the Depends, Imports or LinkingTo fields

## Limitations

- GitHub-only packages lack systematic Depends/Imports metadata dumps, so their reverse-dependency edges may be under-represented
- Connectivity differences (e.g. Bioconductor's higher connectivity) partly reflect Bioconductor's reuse-oriented policy, not purely technical dependency structure

## Evidence

- [abstract] Two hundred metabolomics packages on CRAN, Bioconductor and GitHub: "In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub."
- [abstract] R as processing environment: "The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets."
- [other] metaRbolomics repository: [evidence span withheld — anchor status: not_found]
