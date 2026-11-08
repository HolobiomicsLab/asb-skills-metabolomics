---
name: fisher-exact-test-statistics
description: Use when a user supplies a list of metabolites and/or genes of interest
  and needs pathway overrepresentation p-values combined across analyte types, with
  BH and Holm multiple-testing adjustment.
when_to_use_negative:
- The analysis goal is clustering of enriched pathways rather than overrepresentation
  testing (use findCluster / compute_overlap_matrix instead).
- Only a single analyte type is present and no pathway-size filtering is needed —
  the combined Fisher step is unnecessary and runFisherTest alone suffices.
- The input is a pathway-level annotation table rather than a user analyte list; no
  overrepresentation test is applicable.
edam_operation: http://edamontology.org/operation_0371
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3314
tools:
- name: RaMP
  role: provides the R implementation of Fisher's exact test, Fisher's combined probability
    test, BH and Holm adjustment, and pathway enrichment queries
  repo: https://github.com/ncats/RaMP-DB/
- name: MySQL
  role: hosts the RaMP database that runFisherTest and runCombinedFisherTest query
    for pathway counts and background analyte totals
- name: RMySQL
  role: R package used by connectToRaPM to open the RaMP MySQL connection for pathway
    and analyte queries
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:60343c5d9e443cfa2e460c5465f928da3f3357523cd2eb560987dffb17a87810
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncats/RaMP-DB
  tools:
  - MySQL
  - R
  - RMySQL
  - RaMP
  edam_operation: http://edamontology.org/operation_0371
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# fisher-exact-test-statistics

## Summary

Computes pathway overrepresentation p-values for a supplied metabolite and/or gene list using Fisher's exact test on each pathway's 2x2 contingency table within RaMP, then combines metabolite and gene p-values with Fisher's combined probability test and adjusts them with Benjamini-Hochberg and Holm procedures.

## When to use

Use when a user supplies a list of metabolites and/or genes of interest and needs pathway overrepresentation p-values combined across analyte types, with BH and Holm multiple-testing adjustment.

## When NOT to use

- The analysis goal is clustering of enriched pathways rather than overrepresentation testing (use findCluster / compute_overlap_matrix instead).
- Only a single analyte type is present and no pathway-size filtering is needed — the combined Fisher step is unnecessary and runFisherTest alone suffices.
- The input is a pathway-level annotation table rather than a user analyte list; no overrepresentation test is applicable.

## Inputs

- list of metabolites and/or genes of interest (analytes)
- RaMP pathway annotations (pathwaydf) for the analytes
- total counts of metabolites and genes in the background (total_metabolites, total_genes)
- analyte type specification (metabolite, gene, or both)
- RaMP MySQL database connection parameters (dbname, username, conpass, host)

## Outputs

- Fisher's-method enrichment table with per-pathway raw p-values, metabolite p-values (Pm), gene p-values (Pg), and combined p-values (Pcomb)
- BH-FDR-adjusted and Holm-adjusted p-values for each pathway
- filtered results table (fishers_results.csv) after applying p_holmadj_cutoff and p_fdradj_cutoff

## How to apply

Map the analyte list to RaMP pathway identifiers via getPathwayFromAnalyte, then compute pathway overrepresentation p-values separately for metabolites and genes with runFisherTest, applying Fisher's exact test to each pathway's 2x2 contingency table and filtering pathways by the min_analyte size threshold. For pathways annotated with both genes and metabolites, combine the independent metabolite (Pm) and gene (Pg) p-values using runCombinedFisherTest with the statistic Tcomb = -2 x [ln(Pm)+ln(Pg)] compared to a chi-square distribution with 2 degrees of freedom; fall back to the single-type p-value when one type is missing (Pm missing → Pcomb = Pg, Pg missing → Pcomb = Pm). Adjust the raw p-values with both Benjamini-Hochberg FDR and Holm step-down corrections and filter the results table with FilterFishersResults at user-specified p_holmadj_cutoff and p_fdradj_cutoff values. Finally write the enriched table to disk as fishers_results.csv using write_FishersResults. All database access is backed by a local RaMP MySQL database connected via connectToRaPM using RMySQL/DBI.

## Related tools

- **RaMP** (provides the R implementation of Fisher's exact test, Fisher's combined probability test, BH and Holm adjustment, and pathway enrichment queries) — https://github.com/ncats/RaMP-DB/
- **MySQL** (hosts the RaMP database that runFisherTest and runCombinedFisherTest query for pathway counts and background analyte totals)
- **RMySQL** (R package used by connectToRaPM to open the RaMP MySQL connection for pathway and analyte queries)

## Examples

```
fishers_df <- runCombinedFisherTest(pathwaydf, total_metabolites, total_genes, min_analyte, conpass, dbname, username, host); FilterFishersResults(fishers_df, p_holmadj_cutoff, p_fdradj_cutoff)
```

## Evaluation signals

- Each pathway's overrepresentation p-value equals a two-sided Fisher's exact test on the pathway's 2x2 contingency table built from the analyte list and background totals.
- For dual-annotated pathways, Tcomb = -2 x [ln(Pm)+ln(Pg)] must follow a chi-square distribution with 2 degrees of freedom under the null.
- When Pm is missing the combined p-value must equal Pg, and when Pg is missing it must equal Pm (no combined value is emitted without both types present unless the fallback is applied).
- The output must include both Benjamini-Hochberg FDR-adjusted and Holm step-down adjusted p-values columns, and rows with adjusted p-values above the specified cutoffs must be removed.
- The final written file must be named fishers_results.csv and contain the filtered enrichment table.

## Limitations

- Pathways are filtered by a min_analyte size threshold before testing, which can remove small pathways from the enrichment results.
- Fisher's combined probability test assumes the metabolite and gene p-values are independent; correlated inputs may distort Pcomb.
- The legacy Mathelab/RaMP-DB repo is no longer maintained — users should use https://github.com/ncats/RaMP-DB/ instead.

## Evidence

- [methods] runFisherTest computes pathway overrepresentation p-values using Fisher's exact test on each pathway's 2x2 contingency table: "runFisherTest(pathwaydf, total_metabolites, total_genes, analyte_type, conpass, dbname, username, host)"
- [methods] Fisher's method combines metabolite and gene p-values with the chi-square statistic and fallbacks: "runCombinedFisherTest(pathwaydf, total_metabolites, total_genes, min_analyte, conpass, dbname, username, host)"
- [methods] Combined p-values are adjusted with Benjamini-Hochberg and Holm and filtered at user cutoffs: "FilterFishersResults(fishers_df, p_holmadj_cutoff, p_fdradj_cutoff)"
- [readme] README confirms the Fisher-based enrichment workflow is part of the RaMP-DB package: [evidence span withheld — anchor status: external_doc]
