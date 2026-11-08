---
name: pathway-overrepresentation-analysis
description: Use when you have a list of metabolites and/or genes of interest and
  need statistically adjusted pathway overrepresentation p-values from the RaMP relational
  database (KEGG, Reactome, WikiPathways, HMDB annotations).
when_to_use_negative:
- Your analyte list is already a scored/continuous feature table rather than a discrete
  list of identifiers — Fisher's exact test contingency tables require membership
  counts, not fold changes or intensities
- You need pathway enrichment across non-RaMP sources (e.g. GO-only or MSigDB-only
  gene sets) or species lacking KEGG/Reactome/WikiPathways/HMDB annotation coverage
- You only want raw pathway membership lookups without statistical overrepresentation
  testing — use RaMP's batch query functions (e.g. rampFastCata, getPathwayFromAnalyte
  alone) instead
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0605
tools:
- name: RaMP
  role: Core R package providing the relational database (KEGG, Reactome, WikiPathways,
    HMDB) and the Fisher-test/combined-test/adjustment workflow functions
  repo: https://github.com/ncats/RaMP-DB/
- name: RMySQL
  role: R/DBI driver used by connectToRaPM to query the local RaMP MySQL database
- name: MySQL
  role: Backend relational database hosting the pathway and analyte annotations; also
    distributable as a raw mysql dump
  repo: https://github.com/Mathelab/RaMP-DB/tree/master/inst/extdata
- name: R
  role: Runtime environment for the RaMP package functions used in this skill
  repo: https://github.com/Mathelab/RaMP-DB
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:1256e4518b71633d77a26770bd9584dca939dcd42d69479245f4b2686a676cbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncats/RaMP-DB
  tools:
  - MySQL
  - R
  - RMySQL
  - RaMP
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pathway-overrepresentation-analysis

## Summary

Runs RaMP's pathway overrepresentation analysis on user-supplied gene and/or metabolite lists, using Fisher's exact test per pathway and Fisher's combined-probability test to merge metabolite- and gene-side p-values, followed by Benjamini-Hochberg and Holm multiple-testing adjustment.

## When to use

Use when you have a list of metabolites and/or genes of interest and need statistically adjusted pathway overrepresentation p-values from the RaMP relational database (KEGG, Reactome, WikiPathways, HMDB annotations).

## When NOT to use

- Your analyte list is already a scored/continuous feature table rather than a discrete list of identifiers — Fisher's exact test contingency tables require membership counts, not fold changes or intensities
- You need pathway enrichment across non-RaMP sources (e.g. GO-only or MSigDB-only gene sets) or species lacking KEGG/Reactome/WikiPathways/HMDB annotation coverage
- You only want raw pathway membership lookups without statistical overrepresentation testing — use RaMP's batch query functions (e.g. rampFastCata, getPathwayFromAnalyte alone) instead

## Inputs

- List of metabolite and/or gene identifiers of interest (e.g. hmdb:HMDB04824, kegg:C02712, ensembl, entrez, gene_symbol, uniprot prefixes)
- RaMP MySQL database connection parameters (dbname, username, conpass, host)
- min_analyte pathway-size filter threshold
- p_holmadj_cutoff and p_fdradj_cutoff significance cutoffs
- analyte_type specification (metabolite and/or gene)

## Outputs

- fishers_results.csv containing pathway enrichment table with raw, combined, Holm-adjusted, and BH FDR-adjusted p-values
- Filtered fishers_df data frame of significant pathways at user-specified p_holmadj_cutoff and p_fdradj_cutoff

## How to apply

Connect to the local RaMP MySQL database with connectToRaPM(dbname, username, conpass, host), then map the analyte list to RaMP pathway identifiers via getPathwayFromAnalyte(analytes, find_synonym, conpass, host, dbname, username, NameOrIds), enabling synonym resolution. Compute pathway overrepresentation p-values separately for metabolites and genes with runFisherTest(pathwaydf, total_metabolites, total_genes, analyte_type, conpass, dbname, username, host), which applies Fisher's exact test to each pathway's 2x2 contingency table and enforces a min_analyte pathway-size filter. For pathways annotated with both genes and metabolites, combine the independent p-values using runCombinedFisherTest(pathwaydf, total_metabolites, total_genes, min_analyte, conpass, dbname, username, host): Tcomb = -2 x [ln(Pm)+ln(Pg)] compared to a chi-square distribution with 2 degrees of freedom, with fallbacks (Pm missing → Pcomb = Pg; Pg missing → Pcomb = Pm). Adjust raw p-values with both Benjamini-Hochberg FDR and Holm step-down corrections, then filter the enrichment table at user-specified cutoffs with FilterFishersResults(fishers_df, p_holmadj_cutoff, p_fdradj_cutoff) and write it to fishers_results.csv via write_FishersResults(fishResults, outputfile).

## Related tools

- **RaMP** (Core R package providing the relational database (KEGG, Reactome, WikiPathways, HMDB) and the Fisher-test/combined-test/adjustment workflow functions) — https://github.com/ncats/RaMP-DB/
- **RMySQL** (R/DBI driver used by connectToRaPM to query the local RaMP MySQL database)
- **MySQL** (Backend relational database hosting the pathway and analyte annotations; also distributable as a raw mysql dump) — https://github.com/Mathelab/RaMP-DB/tree/master/inst/extdata
- **R** (Runtime environment for the RaMP package functions used in this skill) — https://github.com/Mathelab/RaMP-DB

## Examples

```
library(RaMP); con <- connectToRaPM(dbname="ramp", username="user", conpass="pw", host="localhost"); pw <- getPathwayFromAnalyte(analytes=c("hmdb:HMDB04824","ensembl:ENSG00000141510"), find_synonym=TRUE, conpass="pw", host="localhost", dbname="ramp", username="user", NameOrIds="ids"); res <- runCombinedFisherTest(pw, total_metabolites, total_genes, min_analyte, conpass="pw", dbname="ramp", username="user", host="localhost"); sig <- FilterFishersResults(res, p_holmadj_cutoff=0.05, p_fdradj_cutoff=0.05); write_FishersResults(sig, "fishers_results.csv")
```

## Evaluation signals

- fishers_results.csv is produced and contains columns for raw Pm, Pg, combined Pcomb, Holm-adjusted, and BH FDR-adjusted p-values
- Every reported pathway's combined p-value is consistent with Tcomb = -2 x [ln(Pm)+ln(Pg)] under a chi-square distribution with 2 df, or matches the documented fallback (Pcomb = Pg when Pm missing, Pcomb = Pm when Pg missing)
- No pathway with fewer than min_analyte analytes of the queried type appears in the output
- Adjusted p-values are monotonically non-decreasing relative to raw p-values for both BH and Holm corrections
- Filtered table contains only rows where the Holm-adjusted and FDR-adjusted p-values are below p_holmadj_cutoff and p_fdradj_cutoff respectively

## Limitations

- Fisher's combined-probability test assumes the metabolite-side and gene-side p-values are independent; correlated evidence across omics types is not modeled
- Single-type fallbacks (Pcomb = Pg or Pm) are applied when one analyte type is missing, so mixed-type pathway results may not reflect a true joint test
- Requires a local RaMP MySQL database connection (connectToRaPM with dbname, username, conpass, host); the Mathelab/RaMP-DB repo is no longer maintained and users are directed to https://github.com/ncats/RaMP-DB/
- Pathways below the min_analyte size threshold are excluded, which can bias results toward larger pathways
- Database updates are checked only multiple times a year, so annotation coverage lags source databases

## Evidence

- [methods] Fisher's-method pathway overrepresentation: [evidence span withheld — anchor status: not_found]
- [methods] combined probability test statistic: [evidence span withheld — anchor status: not_found]
- [methods] multiple-testing adjustment: [evidence span withheld — anchor status: not_found]
- [methods] filter and output functions: "FilterFishersResults(fishers_df, p_holmadj_cutoff, p_fdradj_cutoff)"
- [methods] pathway mapping entry point: "getPathwayFromAnalyte(analytes, find_synonym, conpass, host, dbname, username, NameOrIds)"
- [readme] database scope (readme): "biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the Human Metabolome DataBase (HMDB)"
- [readme] runtime dependency (readme): "The package also includes the raw database file (mysql dump), thereby providing a stand-alone downloadable framework for public use"
- [readme] repo deprecation notice (readme): "This repo is no longer maintained. Please go to https://github.com/ncats/RaMP-DB/ to use RaMP."
