---
name: rdata-object-manipulation
description: Use when an analysis pipeline distributes its inputs/outputs as R .RData
  objects (e.g., GWASnetwork_discoverableGenesOnly_2021_10_14.RData) that must be
  loaded into R, filtered, and re-saved before downstream statistical comparison.
when_to_use_negative:
- The working environment is not R and the data are provided in other formats (e.g.,
  CSV/GraphML) — convert first rather than forcing .RData handling.
- The large network file has not yet been retrieved from the Harvard Dataverse deposit;
  the objects folder will be incomplete and load() will fail.
tools:
- name: metGWAS 1.0
  role: R workflow whose scripts load the objects-folder .RData inputs and save the
    extracted subnetworks back to .RData
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Runtime used to load(), manipulate, and save() the .RData objects
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:ce44a2244296c6af5b9c27783f956cfd753ab60bc2c2950e05cda868a9c5959f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - metGWAS 1.0
  - qvalue
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5238095238095238
n_claims: 21
---

# RData object manipulation

## Summary

Load, inspect, filter, and save R objects held in .RData files (e.g., the 64,516KB GWASnetwork_discoverableGenesOnly_2021_10_14.RData network and the HMDB discoverable-genes object) as the data backbone of the metGWAS 1.0 R workflow.

## When to use

Use when an analysis pipeline distributes its inputs/outputs as R .RData objects (e.g., GWASnetwork_discoverableGenesOnly_2021_10_14.RData) that must be loaded into R, filtered, and re-saved before downstream statistical comparison.

## When NOT to use

- The working environment is not R and the data are provided in other formats (e.g., CSV/GraphML) — convert first rather than forcing .RData handling.
- The large network file has not yet been retrieved from the Harvard Dataverse deposit; the objects folder will be incomplete and load() will fail.

## Inputs

- discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB, HMDB discoverable genes list)
- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB, network representation of the GWAS Catalog)
- R scripts from supplementary material-1 (metGWAS 1.0_19 April.R, qvalue and bioconductor install.R, RSelenium Driver SetUp.R)

## Outputs

- In-memory R objects: GWAS Catalog network and HMDB gene-list objects loaded into the R session
- gwas_keyword_subnetworks.RData containing keyword-filtered GWAS subnetworks (primary node plus first-neighbour nodes)

## How to apply

Place the required .RData files in the metGWAS 1.0 'objects' folder: discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB) and GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB), the latter fetched from the Harvard Dataverse deposit (doi:10.7910/DVN/B32FRP) because it is too large for supplementary upload. Use R's load() to materialize the network representation of the GWAS Catalog and the HMDB gene list into the session, then run metGWAS 1.0_19 April.R, which contains the R-codes for the workflow (with qvalue installed via qvalue and bioconductor install.R and browser automation via RSelenium Driver SetUp.R). After deriving keyword-filtered trait/disease subnetworks (Jaccard >= 0.5, FDR-corrected P <= 0.05 edges), persist the results back to disk as gwas_keyword_subnetworks.RData so they can be reloaded and validated in later sessions.

## Related tools

- **metGWAS 1.0** (R workflow whose scripts load the objects-folder .RData inputs and save the extracted subnetworks back to .RData) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Runtime used to load(), manipulate, and save() the .RData objects)

## Examples

```
load("objects/GWASnetwork_discoverableGenesOnly_2021_10_14.RData"); save(gwas_keyword_subnetworks, file="gwas_keyword_subnetworks.RData")
```

## Evaluation signals

- The objects folder contains both required files with expected sizes (discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData at 64KB; GWASnetwork_discoverableGenesOnly_2021_10_14.RData at 64,516KB).
- load() succeeds for both .RData files without error and the expected R objects appear in the session.
- gwas_keyword_subnetworks.RData exists after the run and contains subnetwork objects when re-loaded.
- Every edge in the saved subnetworks satisfies Jaccard >= 0.5 and FDR-corrected P <= 0.05.
- Keyword-filtered subnetworks each comprise a primary disease/trait node (e.g., 'Kidney', 'Urine') plus its first-neighbour nodes.

## Limitations

- The 64,516KB GWASnetwork_discoverableGenesOnly_2021_10_14.RData file is not included in the supplementary material; only an external Harvard Dataverse download link is provided, wrapped in an Outlook safelinks URL.
- No changelog exists for the metGWAS-1.0 repository, so .RData object schema changes between versions are hard to track.

## Evidence

- [readme] The objects folder should contain the following files:: "The objects folder should contain the following files:"
- [readme] 2. GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB): "2. GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB)"
- [readme] Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded.: "Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded."
- [discussion] metGWAS 1.0_19 April.R contains R-codes for running the workflow.: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
- [discussion] The supplementary material-1 consists of several R-scripts (i.e., metGWAS 1.0_19 April, qvalue and bioconductor install, and RSelenium Driver SetUp): "The supplementary material-1 consists of several R-scripts (i.e., metGWAS 1.0_19 April, qvalue and bioconductor install, and RSelenium Driver SetUp)"
