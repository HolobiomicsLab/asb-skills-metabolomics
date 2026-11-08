---
name: pathway-fdr-significance-filtering
description: Use when metabolite pathway over-representation output (e.g. MetaboAnalyst
  'pathway_results.csv') must be reduced to significant pathways and their constituent
  metabolites converted from KEGG IDs to HMDB IDs.
when_to_use_negative:
- Input is not MetaboAnalyst over-representation output (e.g. raw metabolite abundance
  tables or GWAS summary statistics) — this skill expects pathway_results.csv and
  name_map.csv.
- The analysis does not require KEGG-to-HMDB identifier mapping, e.g. downstream gene
  sets are keyed directly by KEGG metabolite IDs.
- No browser/Selenium environment is available and pathways must be filtered without
  annotating their metabolites — the KEGG retrieval step via RSelenium would fail.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3611
tools:
- name: metGWAS 1.0
  role: R workflow environment in which the pathway FDR filter and KEGG-to-HMDB annotation
    are executed; consumes the filtered HMDB metabolite sets for hypergeometric over-representation
    testing
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: qvalue
  role: Bioconductor package used to compute the FDR q-values that define the ≤ 0.05
    significance threshold
- name: RSelenium
  role: Drives the browser call-out to the KEGG database to retrieve metabolite entries
    for each significant pathway
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:1a28fcd1efbfe0c0c4c9b84962d85bf550834584fd6a7a72387a9471855977f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3611
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pathway-fdr-significance-filtering

## Summary

Filters MetaboAnalyst over-representation pathway results to statistically significant pathways (FDR q-value ≤ 0.05, computed via the qvalue package) and annotates each retained pathway's metabolites by mapping KEGG entries to HMDB IDs for downstream metGWAS 1.0 gene-set construction.

## When to use

Use when metabolite pathway over-representation output (e.g. MetaboAnalyst 'pathway_results.csv') must be reduced to significant pathways and their constituent metabolites converted from KEGG IDs to HMDB IDs.

## When NOT to use

- Input is not MetaboAnalyst over-representation output (e.g. raw metabolite abundance tables or GWAS summary statistics) — this skill expects pathway_results.csv and name_map.csv.
- The analysis does not require KEGG-to-HMDB identifier mapping, e.g. downstream gene sets are keyed directly by KEGG metabolite IDs.
- No browser/Selenium environment is available and pathways must be filtered without annotating their metabolites — the KEGG retrieval step via RSelenium would fail.

## Inputs

- MetaboAnalyst 'pathway_results.csv' (per-pathway over-representation statistics including FDR q-values)
- MetaboAnalyst 'name_map.csv' (KEGG-to-HMDB metabolite identifier mapping)

## Outputs

- filtered_pathway_metabolites_hmdb.csv — one annotated record per significant (FDR ≤ 0.05) pathway, listing its HMDB metabolite identifiers

## How to apply

Load MetaboAnalyst's 'name_map.csv' and 'pathway_results.csv' into the metGWAS 1.0 R environment. Filter pathway rows to those with FDR (q-value, computed with the qvalue package) ≤ 0.05. For each significant pathway, use RSelenium to drive a browser call-out to the KEGG database and retrieve the metabolite entries belonging to that pathway, then map the retrieved KEGG metabolite entries to HMDB IDs via the name_map table. Write the annotated per-pathway sets of HMDB metabolites to filtered_pathway_metabolites_hmdb.csv; these HMDB sets are what metGWAS 1.0 maps to interacting proteins (genes) for the hypergeometric over-representation comparison against GWAS Catalog gene sets.

## Related tools

- **metGWAS 1.0** (R workflow environment in which the pathway FDR filter and KEGG-to-HMDB annotation are executed; consumes the filtered HMDB metabolite sets for hypergeometric over-representation testing) — https://github.com/saifurbd28/metGWAS-1.0
- **qvalue** (Bioconductor package used to compute the FDR q-values that define the ≤ 0.05 significance threshold)
- **RSelenium** (Drives the browser call-out to the KEGG database to retrieve metabolite entries for each significant pathway)

## Examples

```
Rscript "metGWAS 1.0_19 April.R"
```

## Evaluation signals

- filtered_pathway_metabolites_hmdb.csv contains only pathways whose FDR q-value is ≤ 0.05.
- Each retained pathway row carries a non-empty annotation of HMDB metabolite identifiers.
- Every HMDB ID in the output traces back to a KEGG metabolite entry retrieved from KEGG for the corresponding pathway and is present in the name_map table.
- For the KEGG glycerophospholipid metabolism pathway, the annotated metabolites should support derivation of the nine reported genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7).
- RSelenium browser session completes the KEGG call-outs without unannotated (empty) pathway entries in the output file.

## Limitations

- The KEGG retrieval step depends on a live RSelenium browser session and on the online KEGG database; it is a screen-scraping call-out rather than a stable API, so KEGG or browser changes can break it.
- Annotations are limited to genes/metabolites with known interactions in the online databases (HMDB), due to the unpaired nature of the metabolomics and GWAS data sets.
- Environment setup requires the provided scripts (qvalue and bioconductor install.R, RSelenium Driver SetUp.R) and the large RData object GWASnetwork_discoverableGenesOnly_2021_10_14.RData, which is only available via an external Harvard Dataverse download link.

## Evidence

- [abstract] Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases.: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [results] we identified nine genes that interact with metabolites in the KEGG glycerophospholipid metabolism pathway: "we identified nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7) that interact with metabolites in the KEGG glycerophospholipid metabolism pathway and contain"
- [readme] The provided two necessary R-scripts have been given here for setting up the environment.: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [readme] metGWAS 1.0_19 April.R contains R-codes for running the workflow.: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
- [discussion] Since file-2 is too large to upload here, we have provided a downloading link: "Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded."
