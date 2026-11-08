---
name: discoverable-gene-population-background-selection
description: Use when running a hypergeometric over-representation test that compares
  a metabolite-interacting gene set against GWAS-network genes and you must choose
  a statistically defensible population size N (the discoverable genes).
when_to_use_negative:
- When performing standard GO/KEGG enrichment where the article-recommended HMDB/GWAS-network
  discoverable background is not the relevant universe.
- When the metabolite and GWAS gene sets are paired from a single metabolomics-GWAS
  study — metGWAS 1.0 is explicitly designed for independent, unpaired data sets limited
  to genes with known metabolite interactions.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: metGWAS 1.0
  role: R workflow whose Module 4 hypergeometric over-representation test consumes
    the discoverable gene population background
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Implementation environment for loading the RData background objects and running
    the hypergeometric test
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: qvalue
  role: Provided install script (qvalue and bioconductor install.R) for setting up
    the R environment that hosts the background objects
  repo: https://github.com/saifurbd28/metGWAS-1.0
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:b728a2ece22e4629d5d1fd842a83ddcb2453a1d8d479aa348efbb9485ce9624e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# discoverable-gene-population-background-selection

## Summary

Defines the population universe for metGWAS 1.0's hypergeometric over-representation test by restricting the background to the 'discoverable gene set' — the genes that could in principle appear in both the metabolite-interaction gene set and the GWAS-network gene set — rather than using all known genes.

## When to use

Use when running a hypergeometric over-representation test that compares a metabolite-interacting gene set against GWAS-network genes and you must choose a statistically defensible population size N (the discoverable genes).

## When NOT to use

- When performing standard GO/KEGG enrichment where the article-recommended HMDB/GWAS-network discoverable background is not the relevant universe.
- When the metabolite and GWAS gene sets are paired from a single metabolomics-GWAS study — metGWAS 1.0 is explicitly designed for independent, unpaired data sets limited to genes with known metabolite interactions.

## Inputs

- discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB, all genes in HMDB)
- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB, GWAS-network discoverable genes, downloaded from Harvard Dataverse doi:10.7910/DVN/B32FRP)
- Module 2 metabolite-interacting gene set (R object)
- Module 3 GWAS trait/disease subnetwork gene set (R object)

## Outputs

- Population size N (discoverable gene count) used as the hypergeometric universe
- Overlap size k, metabolic gene-set size n, and GWAS subnetwork size m recorded with the test
- hypergeometric_pvalue.csv containing the P-value together with k, n, m, and N

## How to apply

Load the pre-built RData background objects supplied with metGWAS 1.0: 'discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData' (all genes in HMDB) and 'GWASnetwork_discoverableGenesOnly_2021_10_14.RData' (genes discoverable in the GWAS Catalog network). Use the discoverable gene set as the population N, the GWAS trait/disease subnetwork genes as the number of 'disease' successes m, and the metabolite-interacting gene set size n as the draw size. The rationale is that the hypergeometric P-value P(X >= k) is only valid when the random-draw space matches genes that could actually be discovered by both sides of the comparison; using the full genome would inflate or deflate the upper-tail probability. Record k (intersection size), n, m, and N alongside the P-value so the test is reproducible and auditable.

## Related tools

- **metGWAS 1.0** (R workflow whose Module 4 hypergeometric over-representation test consumes the discoverable gene population background) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Implementation environment for loading the RData background objects and running the hypergeometric test) — https://github.com/saifurbd28/metGWAS-1.0
- **qvalue** (Provided install script (qvalue and bioconductor install.R) for setting up the R environment that hosts the background objects) — https://github.com/saifurbd28/metGWAS-1.0

## Examples

```
Rscript -e 'load("objects/GWASnetwork_discoverableGenesOnly_2021_10_14.RData"); N <- length(discoverableGenes); p <- phyper(k-1, m, N-m, n, lower.tail=FALSE)'
```

## Evaluation signals

- The RData background files load without error into the 'objects' folder of the metGWAS 1.0 workspace.
- The recorded population size N equals the number of genes in GWASnetwork_discoverableGenesOnly_2021_10_14.RData, not the full genome or full HMDB gene list.
- The invariants k <= min(n, m) and n, m <= N hold for every reported test.
- The upper-tail hypergeometric probability P(X >= k) is finite and reproducible given the same k, n, m, N quadruple.
- Case-study-1 results reproduce the paper's significance pattern (P < 0.005 for glycerophospholipid pathway genes including APOA5).

## Limitations

- The required 64,516KB RData background file is not bundled in the repository; it must be downloaded externally from Harvard Dataverse (doi:10.7910/DVN/B32FRP).
- The analysis is limited to genes with known metabolite interactions because the metabolomics and GWAS data sets are unpaired, so the background cannot represent undiscovered metabolite-gene links.
- No changelog exists for the repository, so background-file version drift cannot be tracked from the repo alone.

## Evidence

- [abstract] hypergeometric population background: "We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test."
- [readme] discoverable gene set files: "1. discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB) 2. GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB)"
- [discussion] external background download: "Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded."
- [intro] unpaired-data scope limit: "Although such an analysis is limited to genes with known metabolite interactions due to the unpaired nature of the data sets"
