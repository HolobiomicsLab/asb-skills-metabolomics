---
name: hypergeometric-overrepresentation-testing
description: Use when a metabolite-interacting gene set (e.g., from KEGG-pathway metabolites
  mapped via HMDB) must be statistically tested against a disease/trait-specific GWAS
  subnetwork gene set extracted from the GWAS Catalog.
when_to_use_negative:
- The metabolomics and GWAS data are paired from the same cohort — a direct metabolomics-GWAS
  (mGWAS) association analysis is more appropriate than the unpaired over-representation
  approach.
- The gene set of interest has no known metabolite interactions (associations are
  limited to genes with known metabolite interactions due to the unpaired nature of
  the datasets).
- A pathway-level enrichment against a general gene ontology universe is needed rather
  than a GWAS-Catalog-derived disease subnetwork comparison.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0605
tools:
- name: metGWAS 1.0
  role: R workflow implementing the hypergeometric over-representation test between
    metabolic and genomic gene sets ('metGWAS 1.0_19 April.R')
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: execution environment for the metGWAS pipeline, environment setup scripts,
    and the test computation
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: qvalue
  role: multiple-testing correction of hypergeometric P-values across KEGG-pathway
    tests (FDR <= 0.05 retention)
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: RSelenium
  role: browser driver enabling metGWAS 1.0's online HMDB queries that build the metabolic
    gene sets
  repo: https://github.com/saifurbd28/metGWAS-1.0
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:e62f9d7d6a3c38d53063f31575585b7f89df5f85b8f67d5b4840d7c5fa469342
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hypergeometric-overrepresentation-testing

## Summary

Runs the metGWAS 1.0 hypergeometric over-representation test (upper-tail P(X >= k)) to assess overlap between a metabolite-interacting gene set built from HMDB mappings and a GWAS Catalog disease subnetwork gene set, using the HMDB discoverable gene universe as the population.

## When to use

Use when a metabolite-interacting gene set (e.g., from KEGG-pathway metabolites mapped via HMDB) must be statistically tested against a disease/trait-specific GWAS subnetwork gene set extracted from the GWAS Catalog.

## When NOT to use

- The metabolomics and GWAS data are paired from the same cohort — a direct metabolomics-GWAS (mGWAS) association analysis is more appropriate than the unpaired over-representation approach.
- The gene set of interest has no known metabolite interactions (associations are limited to genes with known metabolite interactions due to the unpaired nature of the datasets).
- A pathway-level enrichment against a general gene ontology universe is needed rather than a GWAS-Catalog-derived disease subnetwork comparison.

## Inputs

- metabolite-interacting gene set from metGWAS 1.0 Module 2 (HMDB-mapped disease-associated metabolites)
- GWAS trait/disease subnetwork gene set from metGWAS 1.0 Module 3 (GWAS Catalog network representation)
- discoverable gene set universe: discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB)
- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB, from Harvard Dataverse doi:10.7910/DVN/B32FRP)

## Outputs

- hypergeometric P-value with overlap size k, metabolic gene-set size n, GWAS subnetwork size m, and discoverable population size N (hypergeometric_pvalue.csv)
- overlapping gene identities (e.g., metGWAS_casestudy1_results.csv, metgwas_case2_enrichment_results.csv)
- qvalue-adjusted FDR values for multi-pathway enrichment tables

## How to apply

Load the Module 2 metabolite-interacting gene set and the Module 3 GWAS trait/disease subnetwork gene set produced by metGWAS 1.0, plus the discoverable gene set (all genes in HMDB / GWAS-network discoverable genes) as the population universe N. Compute the intersection and record overlap size k, metabolic gene-set size n (draw size), GWAS subnetwork size m (number of successes), and population size N. In metGWAS 1.0's R implementation ('metGWAS 1.0_19 April.R'), evaluate the upper-tail hypergeometric probability P(X >= k) of observing at least k overlapping genes when the metabolic gene set is drawn at random from the discoverable population. Write the P-value together with k, n, m, and N, and apply qvalue multiple-testing correction when testing multiple KEGG pathways (retain FDR <= 0.05).

## Related tools

- **metGWAS 1.0** (R workflow implementing the hypergeometric over-representation test between metabolic and genomic gene sets ('metGWAS 1.0_19 April.R')) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (execution environment for the metGWAS pipeline, environment setup scripts, and the test computation) — https://github.com/saifurbd28/metGWAS-1.0
- **qvalue** (multiple-testing correction of hypergeometric P-values across KEGG-pathway tests (FDR <= 0.05 retention)) — https://github.com/saifurbd28/metGWAS-1.0
- **RSelenium** (browser driver enabling metGWAS 1.0's online HMDB queries that build the metabolic gene sets) — https://github.com/saifurbd28/metGWAS-1.0

## Examples

```
Rscript 'metGWAS 1.0_19 April.R'  # after loading discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData and GWASnetwork_discoverableGenesOnly_2021_10_14.RData into the objects folder
```

## Evaluation signals

- Case study 1 must reproduce the cardiovascular-disease overlap at P = 0.0028 with the nine genes APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7 in metGWAS_casestudy1_results.csv.
- Case study 2 must give P = 0.103 for 8 genes (glycine, serine, and threonine metabolism vs kidney network) and P = 0.13 for 5 genes including AGXT2 (alanine, aspartate, and glutamate metabolism vs urine network).
- The test must use the discoverable gene set (all genes in HMDB) as the population universe, not the whole genome.
- For each result, k (overlap), n (metabolic set), m (GWAS subnetwork), and N (discoverable population) must be reported alongside the P-value.
- Case study 2 enrichment tables should retain exactly the four KEGG pathways with qvalue FDR <= 0.05.

## Limitations

- Associations are limited to genes with known metabolite interactions because the datasets are unpaired/standalone.
- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB) is too large for the supplement and must be downloaded from the Harvard Dataverse (doi:10.7910/DVN/B32FRP).
- Online HMDB queries require a functioning RSelenium browser driver, adding environment fragility.
- Case study 2 results were only marginally significant (P = 0.10 and P = 0.13), so low-signal overlaps may not survive scrutiny.

## Evidence

- [methods] Module 4 performs the over-representation test with the hypergeometric test using all genes in the discoverable gene set as the population: [evidence span withheld — anchor status: not_found]
- [methods] upper-tail probability: [evidence span withheld — anchor status: not_found]
- [results] case study 1 overlap: [evidence span withheld — anchor status: not_found]
- [results] case study 2 marginal: [evidence span withheld — anchor status: not_found]
- [discussion] qvalue correction: [evidence span withheld — anchor status: not_found]
- [readme] README method: "We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test."
