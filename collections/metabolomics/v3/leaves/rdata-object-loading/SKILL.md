---
name: rdata-object-loading
description: 'Use when running the metGWAS 1.0 R workflow: the workflow scripts require
  the two .RData objects (GWAS network and HMDB gene list) to be present in the R
  session before any hypergeometric comparison can run.'
when_to_use_negative:
- Your GWAS and metabolomics data are paired (from the same metabolomics-GWAS study)
  — metGWAS 1.0 is explicitly designed for unpaired, standalone datasets.
- The data are not in R's serialized .RData format or cannot be materialized as R
  objects (e.g. you are working in a non-R pipeline); this skill only covers loading
  into an R session.
- You intend to build the GWAS Catalog network from scratch rather than reuse the
  pre-built discoverable-genes-only network object.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Runtime environment into which the .RData objects are loaded; metGWAS 1.0
    is an R pipeline whose scripts expect these objects in the session.
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: metGWAS 1.0
  role: 'Consumer of the loaded objects: its workflow script (metGWAS 1.0_19 April.R)
    reads the GWAS network and HMDB gene-list objects from the objects folder to perform
    the hypergeometric over-representation analysis.'
  repo: https://github.com/saifurbd28/metGWAS-1.0
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:28bb0e24f80dbab01b9f8748e1bba7100aeb00da991024f286f7761c38544f18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - metGWAS 1.0
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# rdata-object-loading

## Summary

Loads the pre-built .RData objects shipped with metGWAS 1.0 — the GWAS Catalog network (GWASnetwork_discoverableGenesOnly_2021_10_14.RData) and the HMDB discoverable-gene list — into the R environment so the metGWAS 1.0 workflow can access the network representation of the GWAS Catalog and the gene universe.

## When to use

Use when running the metGWAS 1.0 R workflow: the workflow scripts require the two .RData objects (GWAS network and HMDB gene list) to be present in the R session before any hypergeometric comparison can run.

## When NOT to use

- Your GWAS and metabolomics data are paired (from the same metabolomics-GWAS study) — metGWAS 1.0 is explicitly designed for unpaired, standalone datasets.
- The data are not in R's serialized .RData format or cannot be materialized as R objects (e.g. you are working in a non-R pipeline); this skill only covers loading into an R session.
- You intend to build the GWAS Catalog network from scratch rather than reuse the pre-built discoverable-genes-only network object.

## Inputs

- discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB) — HMDB discoverable gene list
- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB) — network representation of the GWAS Catalog restricted to discoverable genes
- Harvard Dataverse deposit doi:10.7910/DVN/B32FRP (source of the large network .RData file)

## Outputs

- R global-environment objects: the GWAS Catalog network object and the HMDB discoverable-gene list, named as expected by metGWAS 1.0_19 April.R

## How to apply

Place the two required .RData files into the 'objects' folder expected by metGWAS 1.0_19 April.R: discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB) and GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB). Because the GWAS network file is too large for the supplementary package, first fetch it from the Harvard Dataverse deposit (doi:10.7910/DVN/B32FRP) linked in 'GWASnetwork_discoverableGenesOnly_2021_10_14.RData.docx'. In R, first run the environment-setup scripts (qvalue and bioconductor install.R, RSelenium Driver SetUp.R), then load each .RData object (e.g. via load()) so the network object and gene list are attached to the global environment with the object names the workflow scripts expect. Subsequent metGWAS 1.0 steps — building metabolic gene sets from HMDB-mapped metabolites, extracting genomic gene sets from the GWAS Catalog network, and running the hypergeometric test — depend on these loaded objects, so loading must precede workflow execution.

## Related tools

- **R** (Runtime environment into which the .RData objects are loaded; metGWAS 1.0 is an R pipeline whose scripts expect these objects in the session.) — https://github.com/saifurbd28/metGWAS-1.0
- **metGWAS 1.0** (Consumer of the loaded objects: its workflow script (metGWAS 1.0_19 April.R) reads the GWAS network and HMDB gene-list objects from the objects folder to perform the hypergeometric over-representation analysis.) — https://github.com/saifurbd28/metGWAS-1.0

## Examples

```
load("objects/GWASnetwork_discoverableGenesOnly_2021_10_14.RData"); load("objects/discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData")
```

## Evaluation signals

- Both .RData objects load without error and the expected object names appear in the R global environment after load().
- File sizes match the documented values: 64KB for discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData and 64,516KB for GWASnetwork_discoverableGenesOnly_2021_10_14.RData.
- The loaded network object represents the GWAS Catalog restricted to discoverable genes only (no missing/discoverable-gene mismatches when gene sets are queried).
- metGWAS 1.0_19 April.R proceeds past object access without 'object not found' errors, indicating the load ordering (setup scripts, then .RData loads, then workflow) was correct.

## Limitations

- The GWASnetwork_discoverableGenesOnly_2021_10_14.RData file (64,516KB) is not included in the supplementary material; it must be downloaded from the Harvard Dataverse link, so the skill fails if the deposit is unavailable.
- Analysis is limited to genes with known metabolite interactions in HMDB due to the unpaired nature of the datasets.
- No changelog is available for the saifurbd28/metGWAS-1.0 repository, so object/format drift cannot be tracked from the repo.

## Evidence

- [readme] The objects folder should contain the following files: 1. discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB) 2. GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB): "The objects folder should contain the following files: 1. discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData (64KB)"
- [discussion] Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded.: "Since file-2 is too large to upload here, we have provided a downloading link from where this RData file can be downloaded."
- [methods] Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies.: "Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies."
