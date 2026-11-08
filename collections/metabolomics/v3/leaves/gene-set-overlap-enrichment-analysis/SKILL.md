---
name: gene-set-overlap-enrichment-analysis
description: Use when two gene sets from independent studies (e.g. metabolite-interacting
  genes and GWAS trait genes) must be statistically compared for over-represented
  overlap via a hypergeometric upper-tail test.
when_to_use_negative:
- Input is paired metabolomics-GWAS data (metabolite-GWAS studies) — metGWAS 1.0 is
  designed for independent, unpaired metabolomics and GWAS studies, and analysis is
  limited to genes with known metabolite interactions
- A full rank-based or permutation gene-set enrichment (GSEA-style) test is needed
  — this skill computes only a hypergeometric over-representation P-value on set overlap
  sizes
- Gene sets are defined on different identifier universes that cannot be mapped to
  the discoverable gene population (GWAS-network discoverable genes), since k, n,
  m, N must be counts in a common universe
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0625
tools:
- name: metGWAS 1.0
  role: Provides the R workflow and hypergeometric over-representation test comparing
    metabolic and genomic gene sets
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Implementation language for the pipeline (metGWAS 1.0_19 April.R) that runs
    the hypergeometric test and writes hypergeometric_pvalue.csv
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: qvalue
  role: Bioconductor package installed via the qvalue and bioconductor install.R setup
    script required before running the workflow
- name: RSelenium
  role: Browser driver set up by RSelenium Driver SetUp.R to support the metGWAS 1.0
    workflow environment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:598e3ed546e8ff4afe74ebfd254c725bdb2a1ceb86a3021819726c2b08302edc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gene-set-overlap-enrichment-analysis

## Summary

MetGWAS 1.0 Module 4 computes an upper-tail hypergeometric over-representation P-value for the overlap between a metabolite-interacting gene set (Module 2) and a trait/disease-related GWAS subnetwork gene set (Module 3), against a discoverable-gene population.

## When to use

Use when two gene sets from independent studies (e.g. metabolite-interacting genes and GWAS trait genes) must be statistically compared for over-represented overlap via a hypergeometric upper-tail test.

## When NOT to use

- Input is paired metabolomics-GWAS data (metabolite-GWAS studies) — metGWAS 1.0 is designed for independent, unpaired metabolomics and GWAS studies, and analysis is limited to genes with known metabolite interactions
- A full rank-based or permutation gene-set enrichment (GSEA-style) test is needed — this skill computes only a hypergeometric over-representation P-value on set overlap sizes
- Gene sets are defined on different identifier universes that cannot be mapped to the discoverable gene population (GWAS-network discoverable genes), since k, n, m, N must be counts in a common universe

## Inputs

- Module 2 metabolite-interacting gene set (R output from mapping disease-associated metabolites to interacting proteins via HMDB)
- Module 3 GWAS trait/disease subnetwork gene set (R output from network representation of the GWAS Catalog)
- Discoverable gene set population universe (GWASnetwork_discoverableGenesOnly_2021_10_14.RData, 64,516KB)

## Outputs

- hypergeometric_pvalue.csv containing the over-representation P-value with the overlap size k, metabolic gene-set size n, GWAS subnetwork size m, and discoverable population size N

## How to apply

Load into R the Module 2 metabolite-interacting gene set and the Module 3 GWAS trait/disease subnetwork gene set produced by metGWAS 1.0, plus the discoverable gene set (GWASnetwork_discoverableGenesOnly_2021_10_14.RData) as the population universe. Compute the intersection and record the overlap size k, the metabolic gene-set size n (draw size), the GWAS subnetwork size m (number of successes), and the discoverable population size N. Run the hypergeometric test as the upper-tail probability P(X >= k) of observing at least k overlapping genes when drawing the metabolic gene set at random from the discoverable population. Write the P-value together with k, n, m, and N to hypergeometric_pvalue.csv. The test is implemented in the metGWAS 1.0 R pipeline (metGWAS 1.0_19 April.R); environment setup requires the qvalue and bioconductor install.R and RSelenium Driver SetUp.R scripts.

## Related tools

- **metGWAS 1.0** (Provides the R workflow and hypergeometric over-representation test comparing metabolic and genomic gene sets) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Implementation language for the pipeline (metGWAS 1.0_19 April.R) that runs the hypergeometric test and writes hypergeometric_pvalue.csv) — https://github.com/saifurbd28/metGWAS-1.0
- **qvalue** (Bioconductor package installed via the qvalue and bioconductor install.R setup script required before running the workflow)
- **RSelenium** (Browser driver set up by RSelenium Driver SetUp.R to support the metGWAS 1.0 workflow environment)

## Examples

```
Rscript 'metGWAS 1.0_19 April.R'  # runs Modules 2-4, writing hypergeometric_pvalue.csv with k, n, m, N
```

## Evaluation signals

- hypergeometric_pvalue.csv exists and contains the P-value along with the four test parameters k, n, m, and N
- The reported P-value equals the upper-tail probability P(X >= k) of a hypergeometric draw with population N, successes m, and draw size n — recompute independently in R (e.g. phyper(k-1, m, N-m, n, lower.tail=FALSE)) to confirm
- Overlap size k equals the size of the intersection of the Module 2 metabolite-interacting gene set and the Module 3 GWAS subnetwork gene set, all counted within the discoverable gene population N
- Case study 1 sanity check: the KEGG glycerophospholipid metabolism pathway analysis yielded nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7) with cardiovascular disease association at P < 0.005
- Case study 2 sanity check: glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism pathways showed marginal significance at P = 0.10 and P = 0.13 against kidney-disease GWAS data

## Limitations

- Associations are limited to genes with known metabolite interactions due to the unpaired nature of the data sets
- The required GWASnetwork_discoverableGenesOnly_2021_10_14.RData file (64,516KB) is too large to be included in the repository and must be downloaded from the Harvard Dataverse link
- Case study 2 results were only marginally significant (P = 0.10 and P = 0.13), so the test may lack power for some pathway/trait combinations

## Evidence

- [methods] Hypergeometric test parameterization and rationale: "We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test."
- [readme] Discoverable gene universe input file: "1. discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB) 2. GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB)"
- [readme] Workflow script naming: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
- [results] Case study 1 result validating the test: "we identified nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7) that interact with metabolites in the KEGG glycerophospholipid metabolism pathway"
- [results] Case study 2 marginal significance levels: "we found marginal significance (P = 0.10 and P = 0.13) for glycine, serine, and threonine metabolism and alanine, aspartate, and glutamate metabolism pathways"
