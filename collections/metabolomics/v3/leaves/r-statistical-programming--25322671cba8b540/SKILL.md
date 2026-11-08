---
name: r-statistical-programming
description: Use when reconstructing or reproducing MetENP's R-based metabolite/pathway
  enrichment workflow, e.g. running hypergeometric metabolite class enrichment or
  sourcing MetENP analysis scripts in an R session.
when_to_use_negative:
- The analysis target is pathway (KEGG) enrichment scoring rather than metabolite
  class enrichment — use path_enrichmentscore.R instead
- The working environment is not R (no R runtime and no willingness to install the
  KEGGREST/KEGGgraph/pathview Bioconductor dependencies) — MetENPweb on Metabolomics
  Workbench is the alternative interface
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetENP
  role: R package whose scripts implement the hypergeometric metabolite class enrichment
    and the surrounding enrichment workflow
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: R
  role: Programming language/runtime in which the enrichment statistics (e.g. phyper
    hypergeometric test) and the pipeline scripts are executed
- name: KEGGREST
  role: Bioconductor dependency required for MetENP's KEGG pathway mapping steps in
    R
  repo: https://github.com/Bioconductor/KEGGREST
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:6e97bb2e57683b41db1e2a283ef7ad3b6300a208f978a1679c6cbd55be4a7425
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGGREST
  - MetENP
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# r-statistical-programming

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Implements the MetENP metabolomics enrichment pipeline in R, using R statistical functions such as the one-sided hypergeometric test (phyper) for metabolite class enrichment and scripting individual analysis steps as sourceable R files.

## When to use

Use when reconstructing or reproducing MetENP's R-based metabolite/pathway enrichment workflow, e.g. running hypergeometric metabolite class enrichment or sourcing MetENP analysis scripts in an R session.

## When NOT to use

- The analysis target is pathway (KEGG) enrichment scoring rather than metabolite class enrichment — use path_enrichmentscore.R instead
- The working environment is not R (no R runtime and no willingness to install the KEGGREST/KEGGgraph/pathview Bioconductor dependencies) — MetENPweb on Metabolomics Workbench is the alternative interface

## Inputs

- Significant-metabolite table (R data frame / CSV) with RefMet-standardized metabolite classes produced by the significant_met / convert_refmet steps
- Metabolite names/abundances plus metadata from a Metabolomics Workbench study or custom dataset

## Outputs

- Per-RefMet-class enrichment table (R data frame / met_class_enrichment.csv) with enrichment score and hypergeometric p-value for each metabolite class
- Downstream MetENP objects for pathway enrichment score, gene/reaction/enzyme information

## How to apply

Work inside R, either with the MetENP package installed via devtools (after BiocManager installation of KEGGREST, KEGGgraph, and pathview) or by sourcing the individual function scripts (e.g. significant_met.R, anova_ana.R, convert_refmet.R, metclassenrichment.R) from the MetENP repository. For the metabolite class enrichment step, tabulate the significant-metabolite table by RefMet class to obtain the contingency counts — N = all metabolites detected in the study, L = all significant metabolites, M = significant metabolites in a class, k = all metabolites in that class — and compute the one-sided enrichment p-value with R's phyper(M-1, L, N-L, k, lower.tail=FALSE). Derive each class's enrichment score from expected vs. observed significant-metabolite counts and assemble per-class results into an R data frame. Write the class-level results (e.g. met_class_enrichment.csv) and validate that every p-value lies within [0, 1].

## Related tools

- **MetENP** (R package whose scripts implement the hypergeometric metabolite class enrichment and the surrounding enrichment workflow) — https://github.com/metabolomicsworkbench/MetENP
- **R** (Programming language/runtime in which the enrichment statistics (e.g. phyper hypergeometric test) and the pipeline scripts are executed)
- **KEGGREST** (Bioconductor dependency required for MetENP's KEGG pathway mapping steps in R) — https://github.com/Bioconductor/KEGGREST

## Examples

```
source('path/metclassenrichment.R')
```

## Evaluation signals

- met_class_enrichment.csv exists and contains one row per RefMet class with enrichment-score and p-value fields present
- All hypergeometric p-values in the output lie within [0, 1]
- The enrichment p-values reproduce R's phyper(M-1, L, N-L, k, lower.tail=FALSE) given the tabulated N, L, M, k counts
- The R session installs/loads the required Bioconductor dependencies (KEGGREST, KEGGgraph, pathview) without error
- Sourcing each MetENP function script (e.g. metclassenrichment.R, significant_met.R) defines the corresponding function in the R environment

## Limitations

- No explicit CRAN/Bioconductor install command for MetENP itself is documented — installation is via devtools from the GitHub clone or by sourcing functions directly, which the README notes is needed when users have difficulty downloading the package
- Users without admin privileges must install R packages in a user area (e.g. /home/username/.local/R)
- The paper's discussion omits version numbers, license, and changelog details for MetENP, complicating reproducibility pinning

## Evidence
