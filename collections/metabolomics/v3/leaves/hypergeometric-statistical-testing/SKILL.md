---
name: hypergeometric-statistical-testing
description: Use when a metabolomics study has already called significant metabolites
  and needs per-RefMet-class over-representation p-values and enrichment scores (expected
  vs. observed significant counts per class).
when_to_use_negative:
- The goal is pathway-level (KEGG) rather than metabolite-class enrichment — use MetENP's
  path_enrichmentscore step instead
- Metabolites have not yet been mapped to RefMet standardized nomenclature, so class
  membership counts (M and k) cannot be tabulated
- Significant metabolites have not yet been determined — run significant_met first
  rather than testing enrichment on all metabolites
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MetENP
  role: Provides the metclassenrichment.R function that tabulates RefMet classes and
    runs the hypergeometric test; installed from the Metabolomics Workbench GitHub
    repository
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: R
  role: Execution environment; the hypergeometric test is performed via R's phyper
    with lower.tail=FALSE
- name: RefMet
  role: Supplies the standardized metabolite nomenclature/classes that define the
    categories being tested for enrichment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:343da60cfb69d40eef97f1b1223f6c73cf17320394c3b3c2eb48f3cbbd057a2e
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - MetENP
  - R
  - RefMet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hypergeometric-statistical-testing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Statistical core of MetENP's metclassenrichment step: a one-sided hypergeometric test (R phyper) that scores whether significantly altered metabolites are over-represented in a RefMet metabolite class, given the counts of all vs. significant metabolites per class.

## When to use

Use when a metabolomics study has already called significant metabolites and needs per-RefMet-class over-representation p-values and enrichment scores (expected vs. observed significant counts per class).

## When NOT to use

- The goal is pathway-level (KEGG) rather than metabolite-class enrichment — use MetENP's path_enrichmentscore step instead
- Metabolites have not yet been mapped to RefMet standardized nomenclature, so class membership counts (M and k) cannot be tabulated
- Significant metabolites have not yet been determined — run significant_met first rather than testing enrichment on all metabolites

## Inputs

- Significant-metabolite table (R data frame / CSV) produced by the significant_met reconstruction, with RefMet standardized metabolite class annotations
- Full metabolite detection table for the study (to define N, all metabolites detected)
- RefMet class assignment per metabolite (via convert_refmet)

## Outputs

- met_class_enrichment.csv: one row per RefMet metabolite class with hypergeometric p-value and enrichment score (expected vs. observed significant-metabolite counts)
- Per-class data frame of enrichment results suitable for MetENP plotting functions

## How to apply

First tabulate metabolites by RefMet class to build the four contingency counts: N = all metabolites detected in the study, L = all significant metabolites, k = all metabolites in a given metabolite class, and M = significant metabolites within that class. For each RefMet class, compute the one-sided over-representation p-value with R's hypergeometric test phyper(M-1, L, N-L, k, lower.tail=FALSE), i.e. testing whether the class contains more significant metabolites than expected by chance. Derive the enrichment score for each class as the relationship between expected and observed significant-metabolite counts, and assemble per-class p-values and scores into a data frame. In the MetENP pipeline this is implemented in metclassenrichment.R, sourced after significant_met/convert_refmet have produced the RefMet-annotated significant-metabolite table. Write the class-level results as met_class_enrichment.csv for downstream plotting (e.g. metcountplot, plot_met_enrichment).

## Related tools

- **MetENP** (Provides the metclassenrichment.R function that tabulates RefMet classes and runs the hypergeometric test; installed from the Metabolomics Workbench GitHub repository) — https://github.com/metabolomicsworkbench/MetENP
- **R** (Execution environment; the hypergeometric test is performed via R's phyper with lower.tail=FALSE)
- **RefMet** (Supplies the standardized metabolite nomenclature/classes that define the categories being tested for enrichment)

## Examples

```
source('path/metclassenrichment.R')
```

## Evaluation signals

- met_class_enrichment.csv exists and contains one row per RefMet metabolite class
- Each row carries both an enrichment-score field and a p-value field
- All reported p-values lie within [0, 1]
- Each p-value can be reproduced by phyper(M-1, L, N-L, k, lower.tail=FALSE) from the study-level counts N (all metabolites), L (all significant metabolites), k (class metabolites), M (significant metabolites in the class)
- Enrichment scores reflect expected vs. observed significant-metabolite counts per class (observed > expected implies over-representation)

## Limitations

- The test treats 'all metabolites detected in the study' as the background (N), so results depend on detection coverage and class annotation completeness
- It is a one-sided over-representation test of RefMet classes; it does not itself assess KEGG pathways or species-specific pathway enrichment
- No software version, license statement, or quantitative benchmark metrics for the class-enrichment step are provided in the article text

## Evidence
