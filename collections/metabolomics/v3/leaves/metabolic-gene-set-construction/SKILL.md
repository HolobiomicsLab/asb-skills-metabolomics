---
name: metabolic-gene-set-construction
description: Use when you have a disease-associated or otherwise annotated metabolite
  list already carrying HMDB IDs and need to derive a metabolite-interacting gene
  set as input to over-representation analysis.
when_to_use_negative:
- You are building the genomic (GWAS-side) gene set from the GWAS Catalog network
  — that is a separate step using GWASnetwork_discoverableGenesOnly_2021_10_14.RData,
  not metabolite-to-protein mapping
- Your metabolite list lacks HMDB identifiers (e.g., only KEGG or LIPID MAPS IDs),
  so HMDB pages cannot be resolved
- You have paired metabolomics-GWAS data where direct variant-metabolite associations
  can be computed instead of limiting the analysis to genes with known metabolite
  interactions
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0203
tools:
- name: metGWAS 1.0
  role: R workflow whose Module 2 maps metabolites to interacting proteins and builds
    the metabolic gene set
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Implementation language of the pipeline that performs the HMDB scraping, UniProt-to-gene-symbol
    mapping, and CSV output
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: RSelenium
  role: Browser automation used to retrieve interacting-protein listings from online
    HMDB metabolite pages
  repo: https://github.com/saifurbd28/metGWAS-1.0
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:e2b57476ef0c1793d54f85826ddb3d2862c0aa123ee58ab08856292d5cbf06ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metabolic-gene-set-construction

## Summary

Constructs a metabolic gene set by mapping annotated metabolites (via their HMDB IDs) to interacting proteins in the online Human Metabolome Database, then converting UniProt protein IDs to deduplicated current human gene symbols within the metGWAS 1.0 R pipeline.

## When to use

Use when you have a disease-associated or otherwise annotated metabolite list already carrying HMDB IDs and need to derive a metabolite-interacting gene set as input to over-representation analysis.

## When NOT to use

- You are building the genomic (GWAS-side) gene set from the GWAS Catalog network — that is a separate step using GWASnetwork_discoverableGenesOnly_2021_10_14.RData, not metabolite-to-protein mapping
- Your metabolite list lacks HMDB identifiers (e.g., only KEGG or LIPID MAPS IDs), so HMDB pages cannot be resolved
- You have paired metabolomics-GWAS data where direct variant-metabolite associations can be computed instead of limiting the analysis to genes with known metabolite interactions

## Inputs

- annotated metabolite list with HMDB IDs (output of the upstream metabolite annotation step)
- online Human Metabolome Database (HMDB) metabolite pages queried for interacting proteins
- UniProt ID-to-HGNC gene symbol mapping reference

## Outputs

- metabolite_interacting_gene_set.csv: deduplicated list of unique current human gene symbols interacting with the input metabolites

## How to apply

Load the annotated metabolite list with HMDB IDs produced by the upstream annotation step of the metGWAS 1.0 workflow. Using the RSelenium-driven browser automation set up by 'RSelenium Driver SetUp.R', visit each metabolite's HMDB page and extract the listed interacting proteins, which HMDB reports as UniProt IDs. Map those UniProt protein IDs to current human gene symbols using standard UniProt-to-HGNC identifier mapping in the R pipeline (run via 'metGWAS 1.0_19 April.R', with the qvalue/Bioconductor install script used to prepare the environment). Deduplicate the gene symbols and write them to metabolite_interacting_gene_set.csv; this set is what the subsequent hypergeometric over-representation test compares against GWAS Catalog-derived genomic gene sets.

## Related tools

- **metGWAS 1.0** (R workflow whose Module 2 maps metabolites to interacting proteins and builds the metabolic gene set) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Implementation language of the pipeline that performs the HMDB scraping, UniProt-to-gene-symbol mapping, and CSV output) — https://github.com/saifurbd28/metGWAS-1.0
- **RSelenium** (Browser automation used to retrieve interacting-protein listings from online HMDB metabolite pages) — https://github.com/saifurbd28/metGWAS-1.0

## Examples

```
Rscript "RSelenium Driver SetUp.R"; Rscript "metGWAS 1.0_19 April.R"
```

## Evaluation signals

- metabolite_interacting_gene_set.csv exists and parses as a plain table of unique gene symbols (no duplicate symbols, no unresolved UniProt IDs)
- All returned identifiers are current human gene symbols (UniProt-to-HGNC mapping completed, no deprecated symbols)
- The gene set is consistent with the HMDB reference gene list discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB) used by metGWAS 1.0 for over-representation analysis
- Every input metabolite with a valid HMDB ID yields at least one retrieved interacting protein or an explicit, expected empty result — silent dropouts indicate the RSelenium scraping failed
- Downstream hypergeometric test on the constructed metabolic gene set reproduces known associations (e.g., APOA5 recovered in the cardiovascular disease case study)

## Limitations

- Analysis is limited to genes with known metabolite interactions because the metabolomics and GWAS datasets are unpaired
- The step depends on live HMDB web pages accessed via RSelenium browser automation, so it is fragile to HMDB layout changes and requires the RSelenium Driver SetUp.R environment script to run first
- The GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB) needed alongside this step is too large for the repository and must be downloaded externally from the Harvard Dataverse link, and no changelog exists for the repository

## Evidence

- [methods] Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases.: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [readme] RSelenium setup script required for HMDB web retrieval: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [readme] Main workflow script to run: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
- [intro] Scope limited to genes with known metabolite interactions: "Although such an analysis is limited to genes with known metabolite interactions due to the unpaired nature of the data sets, any discovered associations may represent biomarkers and druggable"
