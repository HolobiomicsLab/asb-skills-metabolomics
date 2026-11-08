---
name: reverse-dependency-degree-computation
description: Use when you have a dependency-network edge list of R packages annotated
  by hosting repository (CRAN, Bioconductor, GitHub) and need reverse-dependency connectivity
  metrics per repository.
when_to_use_negative:
- The input is not a dependency network (e.g. it is a feature table, spectra file,
  or unstructured package list without dependency edges).
- Packages cannot be reliably annotated with a hosting repository, making per-repository
  in-degree aggregation meaningless.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3366
tools:
- name: R
  role: Analysis environment in which the reversed dependency graph is loaded and
    in-degree, density, and hub statistics are computed with igraph-compatible data
    structures and the degree() function.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:3cda57ea10ff082213b8e5bd8c8585ef72c6d80aae842f6f4d4e4cbab8987e7e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# reverse-dependency-degree-computation

## Summary

Computes per-repository reverse-dependency (in-degree) statistics for the metabolomics R package network catalogued in the metaRbolomics review, quantifying whether Bioconductor-hosted packages exhibit higher connectivity than CRAN-hosted ones due to Bioconductor's interoperability policy.

## When to use

Use when you have a dependency-network edge list of R packages annotated by hosting repository (CRAN, Bioconductor, GitHub) and need reverse-dependency connectivity metrics per repository.

## When NOT to use

- The input is not a dependency network (e.g. it is a feature table, spectra file, or unstructured package list without dependency edges).
- Packages cannot be reliably annotated with a hosting repository, making per-repository in-degree aggregation meaningless.

## Inputs

- Dependency-network edge list from task_002 (nodes = metabolomics R packages, edges = dependency relations)
- Repository annotation per package (Bioconductor, CRAN, or GitHub) as catalogued in the metaRbolomics review

## Outputs

- connectivity_summary.csv with one row per repository containing mean reverse-dependency (in-degree), network density, and highest-degree hub nodes
- Validated summary confirming non-negative connectivity values and agreement between stated and computed Bioconductor-vs-CRAN comparison

## How to apply

Load the task_002 dependency-network edge list (nodes = metabolomics R packages, edges = dependency relations) into R using igraph-compatible structures. Annotate each node with its hosting repository (Bioconductor, CRAN, or GitHub) as catalogued in the metaRbolomics review, which summarises more than two hundred metabolomics-specific packages. Reverse the graph so edges point from dependent to dependency, then call igraph's degree() to obtain in-degree (reverse-dependency) values per package, and additionally compute overall network density and identify the highest-degree hub nodes. Aggregate mean in-degree, density, and hub statistics into one row per repository and write them to connectivity_summary.csv. Rationale: the review attributes higher Bioconductor connectivity to its policy of encouraging reuse of existing material and interoperable packages, so per-repository mean in-degree is the discriminating metric to test that claim.

## Related tools

- **R** (Analysis environment in which the reversed dependency graph is loaded and in-degree, density, and hub statistics are computed with igraph-compatible data structures and the degree() function.)

## Examples

```
Rscript -e 'g <- read.graph("task_002_edges.csv", format="ncol"); deg <- degree(g, mode="in"); print(tapply(deg, repo_annotation, mean))'
```

## Evaluation signals

- connectivity_summary.csv exists and contains exactly one row per repository (Bioconductor, CRAN, GitHub).
- All connectivity (in-degree) values in the CSV are non-negative numbers.
- Computed mean in-degree per repository matches the direction of the stated finding: Bioconductor metabolomics packages show higher connectivity than CRAN ones.
- Network density is reported and lies in the valid range [0, 1].
- An LLM judge confirms the stated Bioconductor-vs-CRAN connectivity comparison matches the computed metrics.

## Limitations

- Repository annotation is taken as catalogued in the metaRbolomics review; packages hosted on multiple repositories or not catalogued may be misclassified.
- The finding is specific to the metabolomics R package corpus (>200 packages on CRAN, Bioconductor, GitHub) and should not be generalised to other software ecosystems.
- Connectivity is captured only by in-degree and density; the analysis does not capture other network properties such as transitive dependency chains or community structure.

## Evidence

- [abstract] More than two hundred metabolomics specific R packages are summarised, primarily available on CRAN, Bioconductor and GitHub: "In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub."
- [abstract] R as analysis environment for metabolomics data: "The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets."
- [abstract] Extensive overview of R packages across computational metabolomics workflow steps: "This review provides an extensive overview of existing packages in R for different steps in a typical computational metabolomics workflow, including data processing, biostatistics, metabolite"
