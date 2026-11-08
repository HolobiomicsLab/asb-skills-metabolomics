---
name: refmet-metabolite-class-counting
description: Use when a metabolomics dataset already has RefMet class annotations
  and per-metabolite significance calls, and you need per-class (main/sub/super) enrichment
  p-values and enrichment scores.
when_to_use_negative:
- The goal is pathway-level rather than metabolite-class-level enrichment — use MetENP's
  path_enrichmentscore step instead
- Input metabolites lack RefMet class annotation (no main_class/sub_class/super_class),
  so contingency counts per class cannot be formed
- No significance calling has been done yet — the L and M counts depend on prior significant_met
  output
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: MetENP
  role: Provides the metclassenrichment and convert_refmet functions that perform
    the RefMet class tabulation and hypergeometric enrichment test
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: R
  role: Runtime language in which phyper (stats) and the MetENP functions are executed
- name: RefMet
  role: Source of the standardized metabolite class nomenclature (sub_class, super_class,
    main_class) used to define the classes being counted
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:7e62eef6d95bc1d7967fcc0b04002475f37741c865c15281aa048d9cc1c2a668
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - MetENP
  - R
  - RefMet
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# refmet-metabolite-class-counting

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Tabulates significantly altered metabolites and all detected metabolites by RefMet metabolite class (main_class, sub_class, super_class) and computes a per-class hypergeometric enrichment p-value and enrichment score, as implemented in MetENP's metclassenrichment step.

## When to use

Use when a metabolomics dataset already has RefMet class annotations and per-metabolite significance calls, and you need per-class (main/sub/super) enrichment p-values and enrichment scores.

## When NOT to use

- The goal is pathway-level rather than metabolite-class-level enrichment — use MetENP's path_enrichmentscore step instead
- Input metabolites lack RefMet class annotation (no main_class/sub_class/super_class), so contingency counts per class cannot be formed
- No significance calling has been done yet — the L and M counts depend on prior significant_met output

## Inputs

- Significant-metabolite table (R data frame / CSV) from the significant_met step of MetENP
- RefMet-annotated metabolite table (refmet_annotations.csv) with main_class, sub_class, and super_class per metabolite
- Full metabolite-name/concentration table for the study (e.g. from getmwstudies on Metabolomics Workbench study ST000915)

## Outputs

- met_class_enrichment.csv — one row per RefMet class with enrichment score and hypergeometric p-value
- Per-class contingency counts (N, L, k, M) underlying the test

## How to apply

Load the significant-metabolite table produced by MetENP's significant_met step together with the RefMet-annotated table (refmet_annotations.csv carrying main_class, sub_class, super_class). Build the four contingency counts per RefMet class: N = all metabolites detected in the study, L = all significant metabolites, k = all metabolites in a given class, and M = significant metabolites within that class. For each class, run R's one-sided hypergeometric test phyper(M-1, L, N-L, k, lower.tail=FALSE) to obtain the enrichment p-value, since the question is whether significant metabolites are over-represented in the class. Derive each class's enrichment score from the expected-versus-observed significant-metabolite counts and assemble the per-class results into a data frame. Write the result to met_class_enrichment.csv. Because MetENP's Imports dependencies are not auto-attached, explicitly load library(KEGGREST) and the other README-listed packages (plyr, dplyr, tidyr, tidygraph, reshape2, ggplot2, etc.) before sourcing the function files.

## Related tools

- **MetENP** (Provides the metclassenrichment and convert_refmet functions that perform the RefMet class tabulation and hypergeometric enrichment test) — https://github.com/metabolomicsworkbench/MetENP
- **R** (Runtime language in which phyper (stats) and the MetENP functions are executed)
- **RefMet** (Source of the standardized metabolite class nomenclature (sub_class, super_class, main_class) used to define the classes being counted)

## Examples

```
p_class <- phyper(M - 1, L, N - L, k, lower.tail = FALSE)
```

## Evaluation signals

- met_class_enrichment.csv exists and contains exactly one row per RefMet class with enrichment-score and p-value fields populated
- All enrichment p-values lie within [0, 1]
- Contingency invariants hold for every class: M ≤ k, M ≤ L, and k ≤ N
- The p-value for a class reproduces phyper(M-1, L, N-L, k, lower.tail=FALSE) when recomputed manually in R
- Every input metabolite row carries non-missing main_class, sub_class, and super_class (complete RefMet annotation coverage)

## Limitations

- The hypergeometric test treats each RefMet class independently; overlapping class membership between main/sub/super levels is not modeled as multiple independent tests
- Results depend entirely on the upstream significance calling step — no alternative statistical validation or quantitative benchmark results are reported for the MetENP case studies
- No software version number is stated for MetENP in the article, so reproducibility requires pinning the GitHub revision yourself

## Evidence
