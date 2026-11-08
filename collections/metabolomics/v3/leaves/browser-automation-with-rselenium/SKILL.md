---
name: browser-automation-with-rselenium
description: Use when an R pipeline must query web pages that have no batch download
  or REST API — e.g., retrieving HMDB metabolite records by HMDB ID to harvest interacting
  UniProt proteins as in metGWAS 1.0 Module 2.
when_to_use_negative:
- The target database offers a bulk file download or REST API covering the needed
  fields — direct data access is more robust than browser scraping of HMDB pages.
- The metabolite list lacks HMDB IDs (e.g., only KEGG or PubChem identifiers) — the
  metGWAS 1.0 lookup is keyed on HMDB IDs.
- The goal is genomic gene sets from the GWAS Catalog — those are loaded from the
  prebuilt GWASnetwork_discoverableGenesOnly_2021_10_14.RData network file, not scraped.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3489
tools:
- name: RSelenium
  role: R binding that automates the browser session used to open HMDB metabolite
    pages by HMDB ID and extract interacting UniProt proteins
- name: metGWAS 1.0
  role: Workflow whose RSelenium Driver SetUp.R script provisions the Selenium driver
    and whose Module 2 consumes the scraped protein list for UniProt-to-gene-symbol
    annotation
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Pipeline language in which the scraping logic (metGWAS 1.0_19 April.R) and
    identifier mapping are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:cfdc3e2fbf6df35cdbd8076bdf0bf9c84b49309d3aec3a1eb2e5efc23c4b73a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# browser-automation-with-rselenium

## Summary

Drives a Selenium-controlled browser from R using RSelenium to scrape web resources lacking a programmatic API — in metGWAS 1.0, fetching HMDB metabolite pages by HMDB ID to collect each metabolite's interacting proteins (UniProt IDs) that form the metabolic gene set.

## When to use

Use when an R pipeline must query web pages that have no batch download or REST API — e.g., retrieving HMDB metabolite records by HMDB ID to harvest interacting UniProt proteins as in metGWAS 1.0 Module 2.

## When NOT to use

- The target database offers a bulk file download or REST API covering the needed fields — direct data access is more robust than browser scraping of HMDB pages.
- The metabolite list lacks HMDB IDs (e.g., only KEGG or PubChem identifiers) — the metGWAS 1.0 lookup is keyed on HMDB IDs.
- The goal is genomic gene sets from the GWAS Catalog — those are loaded from the prebuilt GWASnetwork_discoverableGenesOnly_2021_10_14.RData network file, not scraped.

## Inputs

- Annotated metabolite list with HMDB IDs (from the upstream metabolite annotation step)
- HMDB metabolite web pages, addressed one-per-metabolite by HMDB ID

## Outputs

- List of interacting proteins annotated as UniProt IDs
- Deduplicated metabolite-interacting gene set of current human gene symbols (metabolite_interacting_gene_set.csv)

## How to apply

First run the RSelenium Driver SetUp.R script shipped in metGWAS 1.0 supplementary material-1 (alongside qvalue and bioconductor install.R) to provision a working Selenium driver/browser session before executing the main workflow in metGWAS 1.0_19 April.R; keeping environment setup separate from workflow code is the README's explicit design. In Module 2, iterate over the annotated metabolite list's HMDB IDs, navigate the automated browser to each metabolite's HMDB page, and scrape the interacting proteins recorded there as UniProt IDs. Map those UniProt IDs to current human gene symbols, deduplicate, and write the metabolite-interacting gene set (e.g., metabolite_interacting_gene_set.csv) that feeds the downstream hypergeometric over-representation test against the GWAS Catalog network. Rationale: metGWAS 1.0 generates metabolic gene sets by mapping disease-associated metabolites to interacting proteins via online databases, and HMDB page retrieval is the step where browser automation replaces direct data access.

## Related tools

- **RSelenium** (R binding that automates the browser session used to open HMDB metabolite pages by HMDB ID and extract interacting UniProt proteins)
- **metGWAS 1.0** (Workflow whose RSelenium Driver SetUp.R script provisions the Selenium driver and whose Module 2 consumes the scraped protein list for UniProt-to-gene-symbol annotation) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Pipeline language in which the scraping logic (metGWAS 1.0_19 April.R) and identifier mapping are implemented)

## Examples

```
Rscript "RSelenium Driver SetUp.R" && Rscript "metGWAS 1.0_19 April.R"
```

## Evaluation signals

- RSelenium Driver SetUp.R completes without error and a live browser/driver session exists before metGWAS 1.0_19 April.R runs
- Every HMDB ID in the annotated metabolite list resolves to a fetched HMDB page with no unresolved entries
- Scraped protein identifiers are all valid UniProt IDs before gene-symbol mapping
- Output file parses as a table of unique current human gene symbols (no duplicates, no NA symbols)
- Resulting gene set is consistent with the HMDB reference gene list discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData used by metGWAS 1.0 for over-representation analysis

## Limitations

- Scraping depends on live HMDB page structure and an operational Selenium driver/browser; page-layout changes or driver failures break the metabolite-to-protein mapping step.
- The required GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB) is not bundled in the repository and must be downloaded from the Harvard Dataverse link.
- Coverage is bounded by genes with known metabolite interactions, since the unpaired nature of the metabolomics and GWAS datasets restricts analysis to such genes.

## Evidence

- [discussion] The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment.: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [abstract] Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases.: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [readme] metGWAS 1.0_19 April.R contains R-codes for running the workflow.: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
- [readme] Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded.: "Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded."
- [abstract] The developed metGWAS 1.0 framework is implemented in an R pipeline and available at: https://github.com/saifurbd28/metGWAS-1.0.: "The developed metGWAS 1.0 framework is implemented in an R pipeline and available at: https://github.com/saifurbd28/metGWAS-1.0."
