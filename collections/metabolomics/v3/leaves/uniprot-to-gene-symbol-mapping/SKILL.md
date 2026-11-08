---
name: uniprot-to-gene-symbol-mapping
description: Use when a metabolite-to-protein step yields UniProt IDs (e.g., scraped
  from HMDB metabolite pages) and downstream over-representation analysis requires
  current human gene symbols instead.
when_to_use_negative:
- Identifiers are already gene symbols or HGNC IDs — re-mapping adds no information
  and risks symbol drift.
- The downstream analysis uses a background gene universe other than the HMDB-derived
  discoverableGenes list, so consistency checks against discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData
  would be invalid.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: metGWAS 1.0
  role: R workflow containing Module 2, which performs the HMDB retrieval, UniProt-to-gene-symbol
    mapping, deduplication, and CSV export
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: RSelenium
  role: Browser automation driver used by the workflow to fetch each HMDB metabolite
    page and extract interacting proteins listed as UniProt IDs
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Implementation language of the pipeline in which the UniProt-to-HGNC identifier
    mapping and deduplication are executed
  repo: https://github.com/saifurbd28/metGWAS-1.0
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:d773185b128118e53463c3a82269d57b3852682a51d53b31e3c816de577816dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# UniProt-to-gene-symbol mapping

## Summary

Converts UniProt protein identifiers retrieved from metabolite interaction records (e.g., HMDB metabolite pages) into a deduplicated set of current human gene symbols, producing the metabolite-interacting gene set that metGWAS 1.0 uses for over-representation analysis.

## When to use

Use when a metabolite-to-protein step yields UniProt IDs (e.g., scraped from HMDB metabolite pages) and downstream over-representation analysis requires current human gene symbols instead.

## When NOT to use

- Identifiers are already gene symbols or HGNC IDs — re-mapping adds no information and risks symbol drift.
- The downstream analysis uses a background gene universe other than the HMDB-derived discoverableGenes list, so consistency checks against discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData would be invalid.

## Inputs

- List of annotated metabolites with HMDB IDs
- Interacting-protein lists as UniProt IDs extracted from HMDB metabolite pages (via RSelenium)
- discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (HMDB gene universe used by metGWAS 1.0)

## Outputs

- metabolite_interacting_gene_set.csv: a table of unique current human gene symbols

## How to apply

In Module 2 of the metGWAS 1.0 R workflow, start from the annotated metabolite list carrying HMDB IDs; use the RSelenium-driven browser automation to open each metabolite's HMDB page and collect its interacting proteins, which are listed as UniProt IDs. Then map the collected UniProt IDs to current human gene symbols using standard UniProt-to-HGNC identifier mapping inside the R pipeline. Deduplicate the resulting symbols and write them to metabolite_interacting_gene_set.csv. The mapping must land on the same gene universe metGWAS 1.0 uses as its discoverable background (discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData), since the hypergeometric over-representation test compares the metabolic gene set against genes discoverable in the HMDB-derived network.

## Related tools

- **metGWAS 1.0** (R workflow containing Module 2, which performs the HMDB retrieval, UniProt-to-gene-symbol mapping, deduplication, and CSV export) — https://github.com/saifurbd28/metGWAS-1.0
- **RSelenium** (Browser automation driver used by the workflow to fetch each HMDB metabolite page and extract interacting proteins listed as UniProt IDs) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Implementation language of the pipeline in which the UniProt-to-HGNC identifier mapping and deduplication are executed) — https://github.com/saifurbd28/metGWAS-1.0

## Examples

```
Rscript "metGWAS 1.0_19 April.R"
```

## Evaluation signals

- Output file metabolite_interacting_gene_set.csv exists and parses as a table.
- Gene symbols in the output are unique (deduplicated) and correspond to current human gene symbols.
- The gene set is consistent with the HMDB reference gene list discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData used by metGWAS 1.0 for over-representation analysis.
- Sanity check against case-study 1: genes such as APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7 should be representable as mapped symbols when the corresponding metabolites are queried.
- Environment prerequisites are installed: qvalue/Bioconductor setup and RSelenium Driver SetUp scripts ran successfully before mapping.

## Limitations

- Retrieval of interacting proteins depends on live HMDB web pages via RSelenium, so the step is fragile to website changes or driver setup failures.
- The large required object GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB) is not bundled in the repository and must be downloaded from the Harvard Dataverse link, which can block reproducibility.
- Associations are limited to genes with known metabolite interactions because of the unpaired nature of the metabolomics and GWAS datasets.

## Evidence

- [methods] Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases.: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [readme] metGWAS 1.0_19 April.R contains R-codes for running the workflow.: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
- [discussion] The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment.: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [readme] discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB): "1. discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB)"
