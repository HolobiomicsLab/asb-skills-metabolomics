---
name: gwas-network-construction
description: Use when constructing genomic gene-set networks from GWAS Catalog studies,
  or when extracting disease/trait-specific GWAS subnetworks to serve as genomic gene
  sets for hypergeometric over-representation testing against metabolic gene sets.
when_to_use_negative:
- The analysis requires paired metabolomics-GWAS data from the same subjects — metGWAS
  1.0 is designed for independent, unpaired standalone studies and limits associations
  to genes with known metabolite interactions.
- The goal is de novo GWAS association testing on raw genotypes rather than over-representation
  analysis against a prebuilt GWAS Catalog network representation.
- The disease/trait of interest cannot be matched by keyword to nodes in the GWAS
  Catalog network, so no primary nodes can be identified for subnetwork extraction.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: metGWAS 1.0
  role: R workflow that loads the GWAS Catalog network object, computes Jaccard-based
    significant edges, filters nodes by trait keywords, and extracts subnetworks
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Execution environment for the metGWAS 1.0 pipeline scripts (metGWAS 1.0_19
    April.R, qvalue and bioconductor install.R, RSelenium Driver SetUp.R)
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: qvalue
  role: Bioconductor package used to compute FDR-corrected P-values for the edge-significance
    test (threshold <= 0.05)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:9937eeee827dcd960b6bad0f8e3db128fa9975796ba3b55eae2756f3c6da88bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gwas-network-construction

## Summary

Builds a trait/disease-related GWAS network from a network representation of the GWAS Catalog, retaining edges between GWAS study gene sets only when their Jaccard coefficient is >= 0.5 with FDR-corrected P <= 0.05, then extracts keyword-filtered subnetworks (e.g. 'Kidney', 'Urine') for downstream over-representation analysis in metGWAS 1.0.

## When to use

Use when constructing genomic gene-set networks from GWAS Catalog studies, or when extracting disease/trait-specific GWAS subnetworks to serve as genomic gene sets for hypergeometric over-representation testing against metabolic gene sets.

## When NOT to use

- The analysis requires paired metabolomics-GWAS data from the same subjects — metGWAS 1.0 is designed for independent, unpaired standalone studies and limits associations to genes with known metabolite interactions.
- The goal is de novo GWAS association testing on raw genotypes rather than over-representation analysis against a prebuilt GWAS Catalog network representation.
- The disease/trait of interest cannot be matched by keyword to nodes in the GWAS Catalog network, so no primary nodes can be identified for subnetwork extraction.

## Inputs

- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (network representation of the GWAS Catalog, from Harvard Dataverse doi:10.7910/DVN/B32FRP)
- GWAS Catalog study gene sets (100s of studies)
- Disease/trait keyword list (e.g. 'Kidney', 'Urine')

## Outputs

- GWAS network whose edges all satisfy Jaccard >= 0.5 and FDR-corrected P <= 0.05
- gwas_keyword_subnetworks.RData containing per-keyword subnetworks (primary node + first neighbours)

## How to apply

Load the prebuilt network object GWASnetwork_discoverableGenesOnly_2021_10_14.RData (downloaded from the Harvard Dataverse deposit doi:10.7910/DVN/B32FRP, ~64,516KB) into R via metGWAS 1.0. Compute pairwise Jaccard coefficients between GWAS study gene sets in the network representation of the GWAS Catalog and retain edges only where the Jaccard coefficient is >= 0.5 and the FDR-corrected P-value is <= 0.05, performing FDR correction with the qvalue package. Filter nodes by disease/trait keywords (e.g. 'Kidney', 'Urine') to identify primary nodes, then extract subnetworks comprising each primary node plus its first-neighbour nodes. Save the keyword-filtered subnetworks as gwas_keyword_subnetworks.RData for use in the hypergeometric comparison against metabolic gene sets mapped from HMDB.

## Related tools

- **metGWAS 1.0** (R workflow that loads the GWAS Catalog network object, computes Jaccard-based significant edges, filters nodes by trait keywords, and extracts subnetworks) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Execution environment for the metGWAS 1.0 pipeline scripts (metGWAS 1.0_19 April.R, qvalue and bioconductor install.R, RSelenium Driver SetUp.R)) — https://github.com/saifurbd28/metGWAS-1.0
- **qvalue** (Bioconductor package used to compute FDR-corrected P-values for the edge-significance test (threshold <= 0.05))

## Examples

```
Rscript "metGWAS 1.0_19 April.R"  # after loading GWASnetwork_discoverableGenesOnly_2021_10_14.RData into the objects folder
```

## Evaluation signals

- Every edge in the extracted subnetworks has Jaccard coefficient >= 0.5 and FDR-corrected P <= 0.05
- The saved gwas_keyword_subnetworks.RData file exists and contains subnetwork objects for each keyword-filtered primary node
- Each extracted subnetwork contains its keyword-matched primary node plus its first-neighbour nodes only
- The GWASnetwork_discoverableGenesOnly_2021_10_14.RData input (~64,516KB) loads successfully and matches the Harvard Dataverse deposit doi:10.7910/DVN/B32FRP

## Limitations

- The GWASnetwork_discoverableGenesOnly_2021_10_14.RData file is too large for the GitHub repository and must be downloaded separately from the Harvard Dataverse link.
- Analysis is limited to genes with known metabolite interactions due to the unpaired nature of the datasets; discovered associations may represent biomarkers and druggable targets rather than causal links.
- No changelog is available for the repository, so script version provenance is difficult to verify.

## Evidence

- [abstract] Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies.: "Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies."
- [other] Jaccard/FDR edge significance thresholds: [evidence span withheld — anchor status: not_found]
- [readme] RData download from Dataverse: "GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB)"
- [discussion] FDR correction tooling in supplementary scripts: "The supplementary material-1 consists of several R-scripts (i.e., metGWAS 1.0_19 April, qvalue and bioconductor install, and RSelenium Driver SetUp)"
