---
name: metabolite-class-enrichment-analysis
description: Use when a table of significantly altered metabolites annotated with
  RefMet classes needs a per-class over-representation test (hypergeometric) and enrichment
  scoring, as in the MetENP workflow.
when_to_use_negative:
- Input is still a raw metabolite intensity table without significance calls — run
  significant_met first (with the study's NA filter, p-value, and log2FC cutoffs)
  before class enrichment.
- The goal is pathway-level rather than metabolite-class-level enrichment — use met_pathways/path_enrichmentscore
  instead.
- Metabolite names cannot be standardized to RefMet classes (e.g., non-standard nomenclature
  that convert_refmet cannot map), since class counts N, L, M, k depend on RefMet
  annotation.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0602
tools:
- name: MetENP
  role: R package providing metclassenrichment and the surrounding significant-metabolite
    workflow whose output feeds the class enrichment test
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: R
  role: Runtime for metclassenrichment; the hypergeometric test is computed with R's
    phyper
- name: RefMet
  role: Standard nomenclature source whose metabolite classes define the categories
    tested for enrichment (via convert_refmet)
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_006
  - task_001
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:74819d8d86cb5a30315851ae9a2f1e295220665e988ad4643e7b0954b5918642
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGGREST
  - MetENP
  - R
  - RefMet
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolite-class-enrichment-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

MetENP's metclassenrichment step: a hypergeometric (phyper) test that scores enrichment of significantly altered metabolites within RefMet metabolite classes, producing a per-class enrichment score and HG p-value.

## When to use

Use when a table of significantly altered metabolites annotated with RefMet classes needs a per-class over-representation test (hypergeometric) and enrichment scoring, as in the MetENP workflow.

## When NOT to use

- Input is still a raw metabolite intensity table without significance calls — run significant_met first (with the study's NA filter, p-value, and log2FC cutoffs) before class enrichment.
- The goal is pathway-level rather than metabolite-class-level enrichment — use met_pathways/path_enrichmentscore instead.
- Metabolite names cannot be standardized to RefMet classes (e.g., non-standard nomenclature that convert_refmet cannot map), since class counts N, L, M, k depend on RefMet annotation.

## Inputs

- Significant-metabolite table from MetENP significant_met (metabolite names, p-values/p_adjust, log2 fold changes, group labels)
- Full metabolite intensity/abundance table with metadata (e.g., from getmwstudies(studyid, typedata) for a Metabolomics Workbench study such as ST001308 or ST000915)
- RefMet-standardized metabolite class annotations (via convert_refmet)

## Outputs

- Per-class enrichment table (one row per RefMet class) with enrichment score and hypergeometric p-value (e.g., met_class_enrichment.csv)
- Class-level enrichment results suitable for downstream plotting (plot_met_enrichment, dotplot_met_class_path) and pathway mapping

## How to apply

First obtain the significant-metabolite table from significant_met (e.g., p<0.05 with p_adjust correction, log2FC cutoff 0.5, NA filters such as 50percent or half_of_min, depending on the study). Tabulate metabolites by RefMet class to build the contingency counts: N = all metabolites detected in the study, L = all significant metabolites, k = all metabolites in a given class, M = significant metabolites in that class. For each RefMet class, compute the one-sided enrichment p-value with R's phyper(M-1, L, N-L, k, lower.tail=FALSE). Derive the enrichment score per class as expected vs. observed significant-metabolite counts, and assemble the per-class results into a data frame (e.g., met_class_enrichment.csv). The rationale is that classes with more significant metabolites than expected by chance indicate biological categories altered in the comparison, which MetENP then feeds forward to species-specific KEGG pathway mapping.

## Related tools

- **MetENP** (R package providing metclassenrichment and the surrounding significant-metabolite workflow whose output feeds the class enrichment test) — https://github.com/metabolomicsworkbench/MetENP
- **R** (Runtime for metclassenrichment; the hypergeometric test is computed with R's phyper)
- **RefMet** (Standard nomenclature source whose metabolite classes define the categories tested for enrichment (via convert_refmet))

## Examples

```
pval <- phyper(M-1, L, N-L, k, lower.tail=FALSE)  # M = sig. metabolites in class, L = all sig., N = all detected, k = all in class
```

## Evaluation signals

- Output file (e.g., met_class_enrichment.csv) exists and contains one row per RefMet class with both enrichment-score and p-value fields present.
- All enrichment p-values lie within [0, 1].
- Contingency counts are consistent by construction: M ≤ L, M ≤ k, N ≥ k, and significant counts per class sum to L.
- Class-enrichment results reproduce case-study expectations, e.g., ST001308 significant metabolites fall in Amino acids and Fatty acids subclasses, and ST000915 yields class enrichment for 85 significant metabolites.
- Recomputing phyper(M-1, L, N-L, k, lower.tail=FALSE) independently on the class contingency table reproduces the reported per-class p-values.

## Limitations

- The test depends on prior correct standardization of metabolite names to RefMet classes; misnamed or unmapped metabolites distort the N, L, M, k counts.
- No statistical validation metrics or quantitative benchmark results are reported for the three case studies (Human, bacteria, dog) in the article.
- MetENP version and changelog information are not stated, so exact function behavior may vary across GitHub checkouts.

## Evidence
