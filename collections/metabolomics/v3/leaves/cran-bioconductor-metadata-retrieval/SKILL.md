---
name: cran-bioconductor-metadata-retrieval
description: Use when you need to enumerate R packages and their Depends/Imports/LinkingTo
  fields from CRAN and Bioconductor to compute reverse-dependency or co-dependency
  structure for a curated package catalogue.
when_to_use_negative:
- The target packages are not R packages hosted on CRAN or Bioconductor, so available.packages()
  or biocViews metadata cannot describe them
- You need the forward-dependency graph (what a metabolomics package itself depends
  on) rather than the reverse-dependency network of packages that depend on the catalogue
- The analysis goal is runtime functional interaction rather than declared Depends/Imports/LinkingTo
  relationships
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Environment for calling available.packages(), parsing Depends/Imports/LinkingTo
    fields, and assembling the reverse-dependency graph with igraph
- name: CRAN
  role: Source of public package metadata fetched via available.packages()
- name: Bioconductor
  role: Source of package metadata via biocViews/package metadata dumps
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:aa50668ffc1377217663bc20c8c13b63d3dd96b16a07cf4b691d52e3e9eedfce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - CRAN
  - R
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cran-bioconductor-metadata-retrieval

## Summary

Retrieves public package metadata from CRAN and Bioconductor to build the reverse-dependency edge list for metabolomics R packages, in order to quantify relative connectivity of CRAN versus Bioconductor packages as in Figure 2 of the metaRbolomics review.

## When to use

Use when you need to enumerate R packages and their Depends/Imports/LinkingTo fields from CRAN and Bioconductor to compute reverse-dependency or co-dependency structure for a curated package catalogue.

## When NOT to use

- The target packages are not R packages hosted on CRAN or Bioconductor, so available.packages() or biocViews metadata cannot describe them
- You need the forward-dependency graph (what a metabolomics package itself depends on) rather than the reverse-dependency network of packages that depend on the catalogue
- The analysis goal is runtime functional interaction rather than declared Depends/Imports/LinkingTo relationships

## Inputs

- Metabolomics R package catalogue from the metaRbolomics review (CRAN, Bioconductor, GitHub packages)
- CRAN package metadata table returned by available.packages()
- Bioconductor biocViews/package metadata dumps
- Depends/Imports/LinkingTo dependency fields of all CRAN and Bioconductor packages

## Outputs

- Directed reverse-dependency graph exported as GraphML
- Flat edge-list CSV (source package, target metabolomics package, dependency field)
- Connectivity comparison of CRAN versus Bioconductor packages

## How to apply

Start from the metabolomics package catalogue produced by the review (more than two hundred packages from CRAN, Bioconductor and GitHub). Fetch CRAN metadata with available.packages() and Bioconductor metadata from the biocViews/package metadata dumps. Parse the Depends, Imports and LinkingTo fields of every CRAN and Bioconductor package and retain only edges whose dependency target is a package in the catalogue, yielding the reverse-dependency edge list reported in Figure 2. Assemble the retained edges into a directed graph (e.g. with igraph in R or Python) and export both a GraphML file and a flat edge-list CSV (source package, target metabolomics package, dependency field). The rationale, grounded in the review, is that Bioconductor's policy encourages reuse of existing material and packages that work better together, which the reverse-dependency network operationalises as higher connectivity for Bioconductor packages than CRAN packages.

## Related tools

- **R** (Environment for calling available.packages(), parsing Depends/Imports/LinkingTo fields, and assembling the reverse-dependency graph with igraph)
- **CRAN** (Source of public package metadata fetched via available.packages())
- **Bioconductor** (Source of package metadata via biocViews/package metadata dumps)

## Examples

```
ap <- available.packages(repos='https://cloud.r-project.org'); g <- graph_from_edgelist(edges, directed=TRUE); write_graph(g, 'revdeps.graphml', format='graphml')
```

## Evaluation signals

- The GraphML file parses as valid graphml format
- The edge-list CSV contains exactly the same edges as the GraphML file
- The reverse-dependency edge count is greater than zero
- Every retained edge's dependency target resolves to a package in the metabolomics catalogue
- The computed connectivity comparison reproduces the reported direction: Bioconductor packages show higher connectivity than CRAN packages

## Limitations

- Only declared static dependencies (Depends/Imports/LinkingTo) are captured; runtime or suggested package interactions are not represented
- GitHub-hosted packages are outside CRAN/Bioconductor metadata dumps, so their reverse dependencies may be under-counted
- Mirror snapshot dates of available.packages() and the Bioconductor dumps affect which dependency edges exist at retrieval time

## Evidence

- [abstract] In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub.: "In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub."
- [other] Bioconductor packages have higher connectivity than CRAN packages due to reuse policy: [evidence span withheld — anchor status: not_found]
- [other] Parsing Depends/Imports/LinkingTo fields to retain edges to catalogue packages: [evidence span withheld — anchor status: not_found]
- [other] Validation of GraphML and CSV edge consistency: [evidence span withheld — anchor status: not_found]
