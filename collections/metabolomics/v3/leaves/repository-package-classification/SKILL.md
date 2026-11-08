---
name: repository-package-classification
description: Use when a dependency edge list or package list of metabolomics R software
  lacks per-package repository labels and the analysis goal requires comparing CRAN-,
  Bioconductor-, and GitHub-hosted packages.
when_to_use_negative:
- The input already carries a per-package repository annotation and the goal is only
  to compute connectivity metrics.
- The package set is not metabolomics-specific R software (e.g., general CRAN packages
  outside the review's scope).
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Environment in which packages are loaded into igraph-compatible data structures
    and annotated with repository labels before computing per-repository connectivity
    statistics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:b181a600af8956094f55e51667058627f680c4c7cfc1f1547c7623d3955de475
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# repository-package-classification

## Summary

Annotates metabolomics R packages with their hosting repository (CRAN, Bioconductor, or GitHub), as catalogued in the metaRbolomics review, so that downstream network analyses can compare packages across repositories.

## When to use

Use when a dependency edge list or package list of metabolomics R software lacks per-package repository labels and the analysis goal requires comparing CRAN-, Bioconductor-, and GitHub-hosted packages.

## When NOT to use

- The input already carries a per-package repository annotation and the goal is only to compute connectivity metrics.
- The package set is not metabolomics-specific R software (e.g., general CRAN packages outside the review's scope).

## Inputs

- metabolomics R package dependency edge list (nodes = packages, edges = dependency relations) from task_002
- metaRbolomics review package catalogue mapping packages to hosting repositories (CRAN, Bioconductor, GitHub)

## Outputs

- annotated graph node table with a repository attribute per package
- per-repository summary table (e.g., connectivity_summary.csv) with one row per repository and non-negative connectivity metrics

## How to apply

Start from the package node set of the metabolomics dependency network (nodes = metabolomics R packages, edges = dependency relations) as compiled in the metaRbolomics Toolbox review, which summarises more than two hundred metabolomics-specific packages primarily available on CRAN, Bioconductor and GitHub. Assign each node exactly one hosting-repository label — CRAN, Bioconductor, or GitHub — by cross-checking the review's catalogue (maintained in the sneumann/metaRbolomics repository). Attach the label as a node attribute in an igraph-compatible data structure so it survives subsetting. The rationale, per the review's connectivity finding, is that Bioconductor's policy encourages reuse of existing, interoperable material, so repository identity is a meaningful grouping variable for reverse-dependency statistics. After annotation, aggregate per-repository metrics (e.g., mean in-degree on the reversed graph) into one row per repository and write them out (e.g., connectivity_summary.csv). Decision point: packages mirrored on multiple hosts should be labelled by their primary/canonical repository to keep counts non-overlapping.

## Related tools

- **R** (Environment in which packages are loaded into igraph-compatible data structures and annotated with repository labels before computing per-repository connectivity statistics)

## Examples

```
library(igraph); V(g)$repo <- vapply(V(g)$name, lookup_repo, ""); deg <- degree(g, mode="in"); tapply(deg, V(g)$repo, mean)
```

## Evaluation signals

- The output CSV exists with exactly one row per repository (CRAN, Bioconductor, GitHub).
- All connectivity values in the summary are non-negative numbers.
- Every package node in the edge list carries exactly one non-empty repository label.
- An LLM judge confirms the stated Bioconductor-vs-CRAN connectivity comparison matches the computed metrics (Bioconductor higher).
- Label assignment is consistent with the metaRbolomics review's repository catalogue.

## Limitations

- Repository coverage is limited to what the review catalogues; a package hosted outside CRAN, Bioconductor, and GitHub may be unlabellable.
- Packages mirrored on multiple hosts require a primary-repository decision that can shift per-repository counts.
- The review's catalogue is a snapshot of its 200+ summarised packages and may not reflect current hosting status.

## Evidence

- [abstract] Packages primarily available on CRAN, Bioconductor and GitHub: "In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub."
- [other] Repository catalogue maintained in sneumann/metaRbolomics: [evidence span withheld — anchor status: not_found]
