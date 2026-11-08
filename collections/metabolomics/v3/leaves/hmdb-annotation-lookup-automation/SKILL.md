---
name: hmdb-annotation-lookup-automation
description: Use when you have a list of standalone metabolite identifiers (e.g.,
  plasma lipidome metabolites) and need their metabolite-interacting protein/gene
  annotations from HMDB to construct a metabolic gene set for over-representation
  analysis.
when_to_use_negative:
- You already have paired metabolomics-GWAS data (direct SNP-metabolite associations)
  — metGWAS 1.0 is designed for standalone, unpaired metabolomics and GWAS studies
  and is limited to genes with known metabolite interactions
- Your metabolites have no HMDB entries or known metabolite-interacting proteins,
  since associations are limited to genes with known metabolite interactions in HMDB
- You need offline/fully reproducible annotation lookup — the workflow depends on
  live HMDB web queries through RSelenium and cannot run without the browser driver
  and internet access
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: metGWAS 1.0
  role: Main R pipeline (metGWAS 1.0_19 April.R) that performs the HMDB-driven metabolite-to-gene
    mapping and the subsequent hypergeometric comparison against GWAS Catalog gene
    sets
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: RSelenium
  role: Browser automation driver (set up via RSelenium Driver SetUp.R) used to query
    HMDB online for metabolite-interacting protein/gene annotations
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Implementation language for the pipeline and the environment-setup scripts
    (qvalue and bioconductor install.R, RSelenium Driver SetUp.R)
  repo: https://github.com/saifurbd28/metGWAS-1.0
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:1642f3b0452982c29871b2f8c4800c752cc99cc316d153d1c2b03a9df7d2a0ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - RSelenium
  - metGWAS 1.0
  - qvalue
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# hmdb-annotation-lookup-automation

## Summary

Automates querying the Human Metabolome Database (HMDB) via RSelenium browser automation to map disease-associated metabolites (e.g., the 39 plasma lipidome metabolites of case study 1) to their interacting proteins/genes, building the metabolic gene set that metGWAS 1.0 compares against GWAS Catalog-derived genomic gene sets with a hypergeometric test.

## When to use

Use when you have a list of standalone metabolite identifiers (e.g., plasma lipidome metabolites) and need their metabolite-interacting protein/gene annotations from HMDB to construct a metabolic gene set for over-representation analysis.

## When NOT to use

- You already have paired metabolomics-GWAS data (direct SNP-metabolite associations) — metGWAS 1.0 is designed for standalone, unpaired metabolomics and GWAS studies and is limited to genes with known metabolite interactions
- Your metabolites have no HMDB entries or known metabolite-interacting proteins, since associations are limited to genes with known metabolite interactions in HMDB
- You need offline/fully reproducible annotation lookup — the workflow depends on live HMDB web queries through RSelenium and cannot run without the browser driver and internet access

## Inputs

- List of standalone metabolite identifiers (e.g., 39 case-study-1 plasma lipidome metabolites)
- discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (HMDB discoverable gene background, 64KB)
- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (GWAS Catalog network gene sets, 64,516KB, downloaded from Harvard Dataverse doi:10.7910/DVN/B32FRP)
- Running RSelenium browser driver session

## Outputs

- Metabolic gene set: metabolite-interacting proteins/genes per pathway (e.g., KEGG glycerophospholipid metabolism)
- Over-representation results table (e.g., metGWAS_casestudy1_results.csv) with overlap size, gene identities, and P values
- Nine-gene overlap list (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7) for case study 1 at P = 0.0028

## How to apply

First set up the R environment with the supplied scripts 'qvalue and bioconductor install.R' and 'RSelenium Driver SetUp.R', which installs and launches the RSelenium browser driver required because HMDB is queried online rather than from a local dump. Then, within metGWAS 1.0_19 April.R, feed the 39 case-study-1 plasma lipidome metabolites (or your own metabolite list) to the RSelenium-driven HMDB queries to extract interacting proteins/genes. Restrict the resulting gene list to 'discoverable genes' using the pre-built artifact discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (the all-genes-in-HMDB background), so the metabolic gene set is comparable to the GWAS Catalog network restricted to the same discoverable universe. The output metabolic gene set is then passed to the hypergeometric over-representation test; for case study 1, the glycerophospholipid-metabolism gene set overlapped the cardiovascular disease GWAS subnetwork at P = 0.0028 with nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7).

## Related tools

- **metGWAS 1.0** (Main R pipeline (metGWAS 1.0_19 April.R) that performs the HMDB-driven metabolite-to-gene mapping and the subsequent hypergeometric comparison against GWAS Catalog gene sets) — https://github.com/saifurbd28/metGWAS-1.0
- **RSelenium** (Browser automation driver (set up via RSelenium Driver SetUp.R) used to query HMDB online for metabolite-interacting protein/gene annotations) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Implementation language for the pipeline and the environment-setup scripts (qvalue and bioconductor install.R, RSelenium Driver SetUp.R)) — https://github.com/saifurbd28/metGWAS-1.0

## Examples

```
source('RSelenium Driver SetUp.R'); source('metGWAS 1.0_19 April.R')  # after loading discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData and GWASnetwork_discoverableGenesOnly_2021_10_14.RData
```

## Evaluation signals

- Case study 1 reproduction: the overlap with the cardiovascular disease GWAS subnetwork should appear in metGWAS_casestudy1_results.csv at P = 0.0028 (< 0.005) with exactly the nine genes APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7
- APOA5 must be present in the overlap, since it was matched from the original metabolomics-GWAS study
- The metabolic gene set must be restricted to discoverable genes defined by discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData so it is comparable to the GWAS Catalog network gene sets
- RSelenium driver must be running before the HMDB queries; pipeline failure to reach HMDB indicates the environment setup step was skipped or incomplete
- As a negative control on scale, case study 2 (urine metabolome, kidney) should show only marginal significance (P = 0.10 and P = 0.13), so a highly significant result there suggests a mapping or background error

## Limitations

- The required RData file GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB) is not included in the repository; it must be downloaded from Harvard Dataverse (doi:10.7910/DVN/B32FRP) via the link given in 'GWASnetwork_discoverableGenesOnly_2021_10_14.RData.docx'
- The analysis is limited to genes with known metabolite interactions due to the unpaired nature of the data sets
- HMDB is queried via online database access through RSelenium, so the workflow is fragile to website changes and network availability
- No changelog was found for the repository, so version drift in scripts (e.g., metGWAS 1.0_19 April.R) is not tracked

## Evidence

- [abstract] Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases.: "Metabolic gene sets are generated by mapping disease-associated metabolites to interacting proteins (genes) via online databases."
- [readme] RSelenium scripts provided for environment setup including browser driver for online database queries.: "The provided two necessary R-scripts have been given here (i.e., qvalue and bioconductor install.R, and RSelenium Driver SetUp.R) for setting up the environment."
- [readme] Large GWAS network RData file only available via external download link.: "Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded."
- [results] Case study 1 nine-gene overlap with KEGG glycerophospholipid metabolism pathway at P < 0.005.: "we identified nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7) that interact with metabolites in the KEGG glycerophospholipid metabolism pathway"
- [intro] Analysis limited to genes with known metabolite interactions due to unpaired data.: "Although such an analysis is limited to genes with known metabolite interactions due to the unpaired nature of the data sets"
