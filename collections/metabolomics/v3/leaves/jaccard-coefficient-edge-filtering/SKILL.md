---
name: jaccard-coefficient-edge-filtering
description: Use when you have a network representation of the GWAS Catalog gene sets
  and need to prune non-significant study-study edges and extract disease/trait-specific
  subnetworks for downstream over-representation analysis.
when_to_use_negative:
- When data are paired metabolomics-GWAS measurements rather than standalone/independent
  studies — metGWAS 1.0 is explicitly designed for the unpaired setting, limiting
  associations to genes with known metabolite interactions
- When building the metabolic gene-set side of the analysis — this skill only prunes
  the GWAS Catalog genomic network, not the metabolite-to-protein mapping step
- When you need all GWAS Catalog study overlaps regardless of significance — discarding
  Jaccard < 0.5 edges removes the majority of weakly overlapping study pairs by design
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: metGWAS 1.0
  role: R workflow whose genomic gene-set generation step performs the Jaccard/FDR
    edge filtering on the GWAS Catalog network representation
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: qvalue
  role: R/Bioconductor package used to FDR-correct the edge overlap P-values before
    applying the 0.05 cutoff
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Environment in which the GWAS network RData object is loaded, filtered, and
    subnetworks extracted and saved
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:470ed549c427b4781c62ef9f28c9f61194d9992688107939e04831c1a75859eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Jaccard coefficient edge filtering

## Summary

Builds trait/disease-related GWAS subnetworks from the GWAS Catalog by keeping only network edges whose overlap of gene sets is statistically significant — Jaccard coefficient >= 0.5 with FDR-corrected P-value <= 0.05 — then extracting keyword-filtered first-neighbour subnetworks.

## When to use

Use when you have a network representation of the GWAS Catalog gene sets and need to prune non-significant study-study edges and extract disease/trait-specific subnetworks for downstream over-representation analysis.

## When NOT to use

- When data are paired metabolomics-GWAS measurements rather than standalone/independent studies — metGWAS 1.0 is explicitly designed for the unpaired setting, limiting associations to genes with known metabolite interactions
- When building the metabolic gene-set side of the analysis — this skill only prunes the GWAS Catalog genomic network, not the metabolite-to-protein mapping step
- When you need all GWAS Catalog study overlaps regardless of significance — discarding Jaccard < 0.5 edges removes the majority of weakly overlapping study pairs by design

## Inputs

- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (GWAS Catalog network restricted to discoverable genes, downloaded from Harvard Dataverse doi:10.7910/DVN/B32FRP)
- Disease/trait keyword list for node filtering (e.g. 'Kidney', 'Urine')

## Outputs

- gwas_keyword_subnetworks.RData containing keyword-filtered subnetworks (each primary node plus its first-neighbour nodes)
- Filtered GWAS network whose edges all satisfy Jaccard >= 0.5 and FDR-corrected P <= 0.05

## How to apply

Load the GWASnetwork_discoverableGenesOnly RData object (GWASnetwork_discoverableGenesOnly_2021_10_14.RData from the Harvard Dataverse deposit, doi:10.7910/DVN/B32FRP) into R within the metGWAS 1.0 workflow. Compute pairwise Jaccard coefficients between GWAS study gene sets and their overlap P-values, then FDR-correct with the qvalue package (installed via the provided 'qvalue and bioconductor install.R' script). Retain only edges where the Jaccard coefficient is >= 0.5 AND the FDR-corrected P-value is <= 0.05, discarding all other edges so that genomic gene sets derive from a statistically defensible network representation of the GWAS Catalog. Filter remaining nodes by disease/trait keywords (e.g. 'Kidney', 'Urine') to identify primary nodes, then extract subnetworks comprising each primary node plus its first-neighbour nodes. Save the extracted keyword-filtered subnetworks as gwas_keyword_subnetworks.RData for the subsequent hypergeometric comparison against metabolic gene sets.

## Related tools

- **metGWAS 1.0** (R workflow whose genomic gene-set generation step performs the Jaccard/FDR edge filtering on the GWAS Catalog network representation) — https://github.com/saifurbd28/metGWAS-1.0
- **qvalue** (R/Bioconductor package used to FDR-correct the edge overlap P-values before applying the 0.05 cutoff) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Environment in which the GWAS network RData object is loaded, filtered, and subnetworks extracted and saved)

## Examples

```
Rscript -e "load('GWASnetwork_discoverableGenesOnly_2021_10_14.RData')"
```

## Evaluation signals

- Every edge in the saved subnetworks satisfies the joint threshold: Jaccard coefficient >= 0.5 AND FDR-corrected P-value <= 0.05
- gwas_keyword_subnetworks.RData exists after the run and loads in R with subnetwork objects present
- Each extracted subnetwork contains a keyword-matched primary node plus its first-neighbour nodes only (no second-degree nodes)
- FDR correction was applied with the qvalue package before thresholding, not raw P-values
- The loaded network matches GWASnetwork_discoverableGenesOnly_2021_10_14.RData from the Harvard Dataverse deposit (doi:10.7910/DVN/B32FRP)

## Limitations

- The required GWASnetwork_discoverableGenesOnly_2021_10_14.RData file (64,516KB) is too large for the repository and must be downloaded separately from the Harvard Dataverse link given in GWASnetwork_discoverableGenesOnly_2021_10_14.RData.docx
- Associations are limited to genes with known metabolite interactions due to the unpaired nature of the data sets
- Environment setup depends on two prerequisite R-scripts (qvalue and bioconductor install.R, RSelenium Driver SetUp.R) and the main metGWAS 1.0_19 April.R script; no repository changelog is available to track network-version changes
- In case study 2, only marginal significance was reached (P = 0.10 and P = 0.13), so keyword subnetworks do not guarantee strong downstream hypergeometric results

## Evidence

- [methods] Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies.: "Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies."
- [discussion] 2. GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB): "2. GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB)"
- [readme] The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment.: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [results] generates and statistically compares metabolic and genomic gene sets using a hypergeometric test: "generates and statistically compares metabolic and genomic gene sets using a hypergeometric test"
