---
name: metabolite-name-to-hmdb-mapping
description: Use when a standalone metabolomics dataset contains free-text or non-standard
  compound names (e.g. urine metabolite lists like Supplementary Material S2) that
  must be converted to HMDB IDs before gene-set mapping and over-representation testing
  in metGWAS 1.0.
when_to_use_negative:
- The metabolite table already carries standardized HMDB identifiers, so re-running
  web-based ID conversion would only add risk
- The data are paired metabolomics-GWAS (variant-to-metabolite) results, where direct
  association testing rather than name-to-HMDB mapping via online databases applies
- No R environment with the RSelenium driver can be set up, so the MetaboAnalyst-driven
  conversion step cannot be executed
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: metGWAS 1.0
  role: R pipeline whose fixed MetaboAnalyst-based module performs the metabolite-name-to-HMDB
    mapping and writes name_map.csv
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: RSelenium
  role: Browser-automation driver set up by RSelenium Driver SetUp.R so the R workflow
    can drive the MetaboAnalyst web interface for identifier conversion
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: MetaboAnalyst
  role: Online platform whose preprocessing/identifier-conversion module the metGWAS
    1.0 script drives to map metabolite names to HMDB identifiers
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:46f718015bae1dabbe645b13e90018684f589d6bd294f66978557935277b0293
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - MetaboAnalyst
  - R
  - RSelenium
  - metGWAS 1.0
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# metabolite-name-to-hmdb-mapping

## Summary

Maps the metabolite names from a standalone metabolomics table (e.g. the 56 urine metabolites of case study 2 in metGWAS 1.0) to Human Metabolome Database (HMDB) identifiers, producing a crosswalk table that downstream hypergeometric over-representation analysis consumes. This identifier normalization is what allows disease-associated metabolites to be linked to interacting proteins and pathway gene sets.

## When to use

Use when a standalone metabolomics dataset contains free-text or non-standard compound names (e.g. urine metabolite lists like Supplementary Material S2) that must be converted to HMDB IDs before gene-set mapping and over-representation testing in metGWAS 1.0.

## When NOT to use

- The metabolite table already carries standardized HMDB identifiers, so re-running web-based ID conversion would only add risk
- The data are paired metabolomics-GWAS (variant-to-metabolite) results, where direct association testing rather than name-to-HMDB mapping via online databases applies
- No R environment with the RSelenium driver can be set up, so the MetaboAnalyst-driven conversion step cannot be executed

## Inputs

- Standalone metabolomics compound table with metabolite names (e.g. Supplementary Material S2 urine metabolite list, 56 compounds)
- Preprocessed metabolite table from the fixed MetaboAnalyst-based normalization/scaling module of metGWAS 1.0
- HMDB reference gene list: discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB)
- RSelenium browser driver configured via RSelenium Driver SetUp.R

## Outputs

- name_map.csv — metabolite-name-to-HMDB-identifier crosswalk table
- Mapped metabolite/HMDB list feeding the hypergeometric over-representation test and pathway_results.csv

## How to apply

Load the preprocessed metabolite table (after the fixed MetaboAnalyst-based normalization/scaling step of metGWAS 1.0, run via metGWAS 1.0_19 April.R in R) and pass the compound names through the MetaboAnalyst identifier-conversion module, which metGWAS 1.0 drives over the web using the RSelenium Driver SetUp.R script. Validate the returned HMDB identifiers against the local HMDB reference object discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB) shipped in the objects folder, since only metabolites with discoverable HMDB genes can contribute to metabolic gene sets. Write the compound-name-to-HMDB crosswalk as name_map.csv, flagging any unmapped compounds, and hand the mapped list to the hypergeometric over-representation step that emits pathway_results.csv. The rationale is that the unpaired design limits metGWAS 1.0 to genes with known metabolite interactions, so a complete, correctly normalized HMDB crosswalk directly determines which metabolic gene sets can be generated.

## Related tools

- **metGWAS 1.0** (R pipeline whose fixed MetaboAnalyst-based module performs the metabolite-name-to-HMDB mapping and writes name_map.csv) — https://github.com/saifurbd28/metGWAS-1.0
- **RSelenium** (Browser-automation driver set up by RSelenium Driver SetUp.R so the R workflow can drive the MetaboAnalyst web interface for identifier conversion) — https://github.com/saifurbd28/metGWAS-1.0
- **MetaboAnalyst** (Online platform whose preprocessing/identifier-conversion module the metGWAS 1.0 script drives to map metabolite names to HMDB identifiers)

## Examples

```
Rscript "metGWAS 1.0_19 April.R"
```

## Evaluation signals

- name_map.csv exists and every row contains a syntactically valid HMDB identifier cross-referenced against discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData
- Mapped-compound coverage is consistent with the source table (e.g. all 56 case study 2 urine metabolites either mapped or explicitly flagged as unmapped)
- Downstream pathway_results.csv reproduces the four over-represented KEGG pathways at FDR <= 0.05 reported in Table 2 (including glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism)
- Marginal-significance validation values appear: P = 0.10 and P = 0.13 for the two named pathways, matching the kidney-related GWAS comparison for case study 2
- RSelenium driver and objects folder (both required RData files, with the 64,516KB GWASnetwork file fetched from the Harvard Dataverse link) were present before the run

## Limitations

- Because the data sets are unpaired, associations are limited to genes with known metabolite interactions — compounds that fail HMDB mapping are silently excluded from metabolic gene sets
- The MetaboAnalyst conversion runs through a web browser via RSelenium, so the mapping is sensitive to web-service availability and driver setup
- The required 64,516KB GWASnetwork_discoverableGenesOnly_2021_10_14.RData is not in the supplementary material and must be downloaded externally from the Harvard Dataverse link before the pipeline can run

## Evidence

- [methods] Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases.: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [other] Map the processed metabolite names to HMDB identifiers against the HMDB reference and write the resulting crosswalk table as name_map.csv.: [evidence span withheld — anchor status: not_found]
- [readme] discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB): "1. discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB)"
- [discussion] and RSelenium Driver SetUp) with an object named folder and the user manual: "The supplementary material-1 consists of several R-scripts (i.e., metGWAS 1.0_19 April, qvalue and bioconductor install, and RSelenium Driver SetUp)"
