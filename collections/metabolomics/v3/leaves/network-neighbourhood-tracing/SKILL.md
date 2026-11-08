---
name: network-neighbourhood-tracing
description: Use when the intersecting genes from a hypergeometric metabolic/GWAS
  comparison need to be attributed to primary keyword nodes versus first-neighbour
  nodes of a GWAS Catalog network to interpret where the signal arises.
when_to_use_negative:
- When running the over-representation analysis itself — this skill is post-hoc node
  attribution, not the hypergeometric comparison step.
- When paired metabolomics-GWAS data are available, since direct SNP-metabolite associations
  bypass the network neighbourhood of the GWAS Catalog.
- When the GWAS Catalog network object GWASnetwork_discoverableGenesOnly_2021_10_14.RData
  has not been downloaded, as it is not bundled in the supplementary material.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: metGWAS 1.0
  role: R workflow used to trace intersection genes in the GWAS subnetwork and label
    them as primary keyword nodes or first-neighbour nodes
  repo: https://github.com/saifurbd28/metGWAS-1.0
- name: R
  role: Execution environment in which the GWAS Catalog network object is loaded and
    the node-tracing tabulation is run
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad523
    title: 'metGWAS 1.0: an R workflow for network-driven over-representation analysis
      between independent metabolomic and meta-genome-wide association studies'
  source_content_hash: sha256:d3b95f17f58368ac8f7ee0a67e1773d266caf325c99660f445d5d692c2044e65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saifurbd28/metGWAS-1.0
  tools:
  - R
  - metGWAS 1.0
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# network-neighbourhood-tracing

## Summary

Traces each gene in the metGWAS 1.0 metabolic/GWAS intersection back to its position in the GWAS Catalog subnetwork, labelling it as a primary keyword node or a first-neighbour node. This reveals whether metabolite-gene associations originate from keyword-matched GWAS studies or from their immediate network neighbours.

## When to use

Use when the intersecting genes from a hypergeometric metabolic/GWAS comparison need to be attributed to primary keyword nodes versus first-neighbour nodes of a GWAS Catalog network to interpret where the signal arises.

## When NOT to use

- When running the over-representation analysis itself — this skill is post-hoc node attribution, not the hypergeometric comparison step.
- When paired metabolomics-GWAS data are available, since direct SNP-metabolite associations bypass the network neighbourhood of the GWAS Catalog.
- When the GWAS Catalog network object GWASnetwork_discoverableGenesOnly_2021_10_14.RData has not been downloaded, as it is not bundled in the supplementary material.

## Inputs

- GWASnetwork_discoverableGenesOnly_2021_10_14.RData (GWAS Catalog network object, 64,516KB, downloaded from Harvard Dataverse doi:10.7910/DVN/B32FRP)
- Intersection gene sets from metGWAS 1.0 hypergeometric comparison (case study 1 cardiovascular disease; case study 2 urine/kidney)
- metGWAS 1.0 R-scripts and objects folder (discoverableGenes_akaAllGenesInHMDB_2021_10_14.RData)

## Outputs

- gene_node_classification.csv (per-gene, per-case-study table of node-type labels: primary keyword node vs first-neighbour node)

## How to apply

Load the GWAS Catalog network object GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB) in R to access the network representation of the GWAS Catalog comprising 100s of studies. Load the case-study intersection gene sets produced upstream by metGWAS 1.0's hypergeometric comparison of metabolic and genomic gene sets (e.g., the nine glycerophospholipid-pathway genes from case study 1). For each intersection gene, use metGWAS 1.0 in R to trace its position in the GWAS subnetwork and label it as a primary keyword node or a first-neighbour node, repeating this per case study since a study's intersection may arise entirely from one node class. Tabulate the node-type label per gene per case study and write the result to gene_node_classification.csv. Judge correctness by confirming every intersection gene resolves to exactly one labelled position in the loaded network object.

## Related tools

- **metGWAS 1.0** (R workflow used to trace intersection genes in the GWAS subnetwork and label them as primary keyword nodes or first-neighbour nodes) — https://github.com/saifurbd28/metGWAS-1.0
- **R** (Execution environment in which the GWAS Catalog network object is loaded and the node-tracing tabulation is run)

## Examples

```
Rscript "metGWAS 1.0_19 April.R"
```

## Evaluation signals

- Every intersection gene from each case study resolves to exactly one node-type label in the loaded GWASnetwork_discoverableGenesOnly_2021_10_14.RData object.
- gene_node_classification.csv contains one row per intersection gene per case study with a binary label (primary keyword node vs first-neighbour node).
- Label distributions are internally consistent per case study — in the source data, case study 1 intersections were exclusively first-neighbour genes while case study 2 intersections were exclusively primary genes.
- Gene labels match the intersection gene sets reported by metGWAS 1.0 (e.g., case study 1 must cover all nine genes APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, PLA2G7).

## Limitations

- The required network object GWASnetwork_discoverableGenesOnly_2021_10_14.RData is too large for the supplementary material and must be downloaded externally from Harvard Dataverse (doi:10.7910/DVN/B32FRP).
- Analysis is limited to genes with known metabolite interactions due to the unpaired nature of the metabolomics and GWAS data sets.
- Case study 2 intersections rest on only marginal pathway significance (P = 0.10 and P = 0.13), so node-type attribution there is supported by weak evidence.

## Evidence

- [methods] We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test.: "We developed a bioinformatics tool, metGWAS 1.0, that generates and statistically compares metabolic and genomic gene sets using a hypergeometric test."
- [abstract] Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies.: "Genomic gene sets are identified from a network representation of the GWAS Catalog comprising 100s of studies."
- [readme] 2. GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB): "2. GWASnetwork_discoverableGenesOnly_2021_10_14.RData (64,516KB)"
- [readme] metGWAS 1.0_19 April.R contains R-codes for running the workflow.: "metGWAS 1.0_19 April.R contains R-codes for running the workflow."
- [results] we identified nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7): "we identified nine genes (APOA5, PLA2G5, PLA2G2D, PLA2G2E, PLA2G2F, LRAT, PLA2G2A, PLB1, and PLA2G7)"
