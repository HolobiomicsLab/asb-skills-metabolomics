---
name: pathway-over-representation-analysis
description: Use when a standalone metabolomics dataset (e.g., a table of 56 named
  urine metabolites) needs pathway-level interpretation via over-representation testing
  before downstream metabolite-gene set comparison.
when_to_use_negative:
- Input is paired metabolomics-GWAS data — metGWAS 1.0 is designed for standalone,
  unpaired datasets and is limited to genes with known metabolite interactions.
- The metabolite list has not been mapped to HMDB identifiers yet — run the name-mapping
  step first, since over-representation testing operates on mapped identifiers.
- You need user-adjustable normalization or scaling — the MetaboAnalyst-based preprocessing
  in this workflow is fixed/hard-coded in the Section 2.1 script.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: metGWAS 1.0
  role: R pipeline whose fixed first step performs MetaboAnalyst-based preprocessing,
    HMDB mapping, and hypergeometric over-representation testing against KEGG pathways
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Runtime environment executing metGWAS 1.0_19 April.R and its environment-setup
    scripts (qvalue and bioconductor install.R, RSelenium Driver SetUp.R)
  repo: https://github.com/saifurbd28/metGWAS-1.0
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:4916c42ca21060bac359758df09052386615adbe6019de7a4def38a86cb56b7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - metGWAS 1.0
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# pathway-over-representation-analysis

## Summary

Hypergeometric over-representation testing of mapped metabolites against KEGG pathway gene sets, as implemented in the MetaboAnalyst-based first step of the metGWAS 1.0 R workflow, to identify pathways enriched in a standalone metabolomics dataset.

## When to use

Use when a standalone metabolomics dataset (e.g., a table of 56 named urine metabolites) needs pathway-level interpretation via over-representation testing before downstream metabolite-gene set comparison.

## When NOT to use

- Input is paired metabolomics-GWAS data — metGWAS 1.0 is designed for standalone, unpaired datasets and is limited to genes with known metabolite interactions.
- The metabolite list has not been mapped to HMDB identifiers yet — run the name-mapping step first, since over-representation testing operates on mapped identifiers.
- You need user-adjustable normalization or scaling — the MetaboAnalyst-based preprocessing in this workflow is fixed/hard-coded in the Section 2.1 script.

## Inputs

- Standalone metabolomics dataset of named metabolites (e.g., 56 urine metabolites from Supplementary Material S2)
- HMDB reference for metabolite-name-to-HMDB-identifier mapping
- KEGG pathway gene sets used as the over-representation background

## Outputs

- name_map.csv (metabolite name to HMDB identifier crosswalk table)
- pathway_results.csv (hypergeometric over-representation statistics and betweenness-centrality-based pathway impact per KEGG pathway)
- List of over-represented KEGG pathways at FDR <= 0.05 (e.g., reported in Table 2)

## How to apply

Load the metabolite table (e.g., the case study 2 urine metabolomics dataset from Supplementary Material S2) into an R session running the metGWAS 1.0 pipeline, then apply the tool's fixed MetaboAnalyst-based preprocessing module with normalization/scaling hard-coded in the metGWAS 1.0_19 April.R script. Map the processed metabolite names to HMDB identifiers, writing the crosswalk as name_map.csv. Run hypergeometric over-representation testing of the mapped metabolites against pathway gene sets and compute betweenness-centrality-based pathway impact, writing the statistics table as pathway_results.csv. Call significant pathways at FDR <= 0.05; treat marginal (P ~ 0.10-0.13) hits as corroborating evidence when matching an original paired metabolomics-GWAS study.

## Related tools

- **metGWAS 1.0** (R pipeline whose fixed first step performs MetaboAnalyst-based preprocessing, HMDB mapping, and hypergeometric over-representation testing against KEGG pathways) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Runtime environment executing metGWAS 1.0_19 April.R and its environment-setup scripts (qvalue and bioconductor install.R, RSelenium Driver SetUp.R)) — https://github.com/saifurbd28/metGWAS-1.0

## Examples

```
Rscript 'metGWAS 1.0_19 April.R'
```

## Evaluation signals

- pathway_results.csv contains hypergeometric P-values and betweenness-centrality-based pathway impact for all mapped KEGG pathways.
- name_map.csv successfully crosswalks the processed metabolite names to HMDB identifiers with no unmapped entries for the input list.
- Over-represented pathways are called at FDR <= 0.05 (four pathways for the 56-metabolite case study 2 urine dataset, reported in Table 2).
- Glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism appear with marginal significance (P = 0.10 and P = 0.13), matching the kidney-related GWAS comparison in case study 2.

## Limitations

- Preprocessing (normalization/scaling) is fixed and hard-coded in the script, so users cannot alter it within this workflow.
- The required GWASnetwork_discoverableGenesOnly_2021_10_14.RData file (64,516KB) is not included in the supplementary material and must be downloaded from the Harvard Dataverse link.
- Associations are limited to genes with known metabolite interactions due to the unpaired nature of the datasets.
- Marginal results (P = 0.10 and P = 0.13) are only nominally significant and should not be treated as FDR-significant enrichment.

## Evidence

- [abstract] We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test.: "We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test."
- [results] Case study 2 marginal pathway significance: "we found marginal significance (P = 0.10 and P = 0.13) for glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism pathways"
- [methods] Metabolic gene set construction: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [readme] Workflow entry script: "metGWAS 1.0_19 April.R contains R-codes for running the workflow. The objects folder should contain the following files:"
