---
name: metabolic-geneset-to-gwas-subnetwork-comparison
description: Use when you have a metabolic gene set (metabolite-interacting genes)
  and a GWAS trait/disease subnetwork gene set and need the over-representation P-value
  of their intersection.
when_to_use_negative:
- The metabolomics and GWAS data are paired (a direct metabolite-quantity GWAS/mGWAS
  is available), since metGWAS 1.0 targets standalone, independent studies
- The gene sets have not been restricted to the discoverable gene population (genes
  with known metabolite interactions), so the hypergeometric universe would be invalid
- You need SNP-level or metabolite-level association statistics rather than a gene-set
  overlap P-value
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: metGWAS 1.0
  role: R workflow whose Module 4 performs the hypergeometric over-representation
    test comparing metabolic and genomic gene sets
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Implementation language for the pipeline (metGWAS 1.0_19 April.R) and the
    phyper-based hypergeometric calculation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:6ce8f6a90fcfd206c2670554b3b55cf6ee6025b64ea52a010bc11b4be202e8fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - metGWAS 1.0
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolic-geneset-to-gwas-subnetwork-comparison

## Summary

Module 4 of metGWAS 1.0 quantifies whether the intersection between a metabolite-interacting gene set and a trait/disease-related GWAS subnetwork gene set is larger than expected by chance, using an upper-tail hypergeometric test over the discoverable gene population.

## When to use

Use when you have a metabolic gene set (metabolite-interacting genes) and a GWAS trait/disease subnetwork gene set and need the over-representation P-value of their intersection.

## When NOT to use

- The metabolomics and GWAS data are paired (a direct metabolite-quantity GWAS/mGWAS is available), since metGWAS 1.0 targets standalone, independent studies
- The gene sets have not been restricted to the discoverable gene population (genes with known metabolite interactions), so the hypergeometric universe would be invalid
- You need SNP-level or metabolite-level association statistics rather than a gene-set overlap P-value

## Inputs

- Metabolite-interacting gene set from Module 2 (mapping of disease-associated metabolites to interacting proteins/genes via HMDB and online databases)
- Trait/disease-related GWAS subnetwork gene set from Module 3 (derived from a network representation of the GWAS Catalog)
- Discoverable gene set universe (GWASnetwork_discoverableGenesOnly_2021_10_14.RData; HMDB all-genes universe in discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData)

## Outputs

- hypergeometric_pvalue.csv containing the over-representation P-value with the overlap size k, metabolic gene-set size n, GWAS subnetwork size m, and discoverable population size N

## How to apply

Load the Module 2 metabolite-interacting gene set and the Module 3 GWAS trait/disease subnetwork gene set into R alongside the discoverable gene universe (GWAS-network discoverable genes). Compute the intersection size k between the metabolic gene set (draw size n) and the GWAS subnetwork genes (number of successes m) within the discoverable population of size N. Evaluate the upper-tail hypergeometric probability P(X >= k), i.e. the chance of observing at least k overlapping genes when drawing the metabolic gene set at random from the discoverable population. Write the P-value together with k, n, m, and N to hypergeometric_pvalue.csv. The rationale is that associations from unpaired data are restricted to genes with known metabolite interactions, so enrichment of GWAS subnetwork genes among metabolite-interacting genes indicates a disease/trait-specific metabolite-gene link.

## Related tools

- **metGWAS 1.0** (R workflow whose Module 4 performs the hypergeometric over-representation test comparing metabolic and genomic gene sets) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Implementation language for the pipeline (metGWAS 1.0_19 April.R) and the phyper-based hypergeometric calculation)

## Examples

```
phyper(k-1, m, N-m, n, lower.tail=FALSE)  # upper-tail P(X >= k) for the metabolic vs GWAS-subnetwork overlap
```

## Evaluation signals

- Recorded P-value equals the upper-tail probability P(X >= k), not a two-sided or lower-tail test
- Overlap size k satisfies 0 <= k <= min(n, m) and the population size N is at least max(n, m)
- hypergeometric_pvalue.csv contains all four counts (k, n, m, N) alongside the P-value
- On the cardiovascular case study, the test recovers significance (P < 0.005) for the nine KEGG glycerophospholipid metabolism genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7)
- Hits reproduce known loci from the original paired metabolomics-GWAS study (e.g. APOA5) before accepting novel associations

## Limitations

- Analysis is limited to genes with known metabolite interactions due to the unpaired nature of the data sets, so associations are restricted to the discoverable gene universe
- The required universe file GWASnetwork_discoverableGenesOnly_2021_10_14.RData (~64,516KB) is not bundled in the repository and must be downloaded from Harvard Dataverse
- Weak pathway signals can remain non-significant (e.g. P = 0.10 and P = 0.13 in the kidney/urine metabolome case study), so marginal results should not be over-interpreted

## Evidence

- [methods] We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test.: "We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test."
- [abstract] Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases.: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [methods] Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies.: "Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies."
- [readme] metGWAS 1.0_19 April.R contains R-codes for running the workflow.: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
- [discussion] The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment.: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [abstract] The metGWAS 1.0 tool was evaluated using standalone metabolomics datasets extracted from two metabolomics-GWAS case studies.: "The metGWAS 1.0 tool was evaluated using standalone metabolomics datasets extracted from two metabolomics-GWAS case studies."
