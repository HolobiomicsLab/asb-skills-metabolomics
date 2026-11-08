---
name: metabolite-gene-network-enrichment-visualisation
description: Use when you have cluster-level differentially expressed metabolites
  and/or genes from an integrated spatial multi-omics object and need regional pathway
  enrichment plus an Enrichment Map network linking pathway feature sets.
when_to_use_negative:
- Input is raw m/z imaging data that has not yet been metabolite-annotated — run the
  SpaMTP annotation pipeline first, since enrichment operates on named metabolites
  and genes
- Differential expression has not been performed per cluster — Fisher's exact test
  and rank-based enrichment both require FindAllDEMs-style differential metabolite/gene
  results as input
- The target pathways lie outside the RaMP-DB-derived annotation sources (KEGG, SMPdb,
  Reactome, WikiPathways) used by this implementation
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: SpaMTP
  role: Provides FishersPathwayAnalysis (Fisher exact over-representation), FindRegionalPathways
    (rank-based, direction-aware enrichment), and PathwayNetworkPlots (Enrichment
    Map) over integrated SM/ST Seurat objects
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: Seurat
  role: Object model carrying the paired MALDI and Visium assays and per-cluster differential
    results that enrichment functions consume
  repo: https://satijalab.org/seurat/
- name: R
  role: Runtime environment; SpaMTP is an R-based package bridging Cardinal metabolomics
    processing and Seurat cell-centric analyses
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:40a6791fcc3c802442355c22d8532a091d34c50ece64cd30be358540932bfe8a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - Seurat
  - SpaMTP
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Metabolite-gene network enrichment visualisation

## Summary

Regional pathway enrichment and Enrichment-Map-style network construction over metabolite and gene feature sets, as implemented in SpaMTP for paired spatial metabolomics (MALDI) and transcriptomics (Visium) data. It converts cluster-level differential analytes into Fisher over-representation and GSEA-like rank-based pathway results, then renders topology-weighted pathway networks.

## When to use

Use when you have cluster-level differentially expressed metabolites and/or genes from an integrated spatial multi-omics object and need regional pathway enrichment plus an Enrichment Map network linking pathway feature sets.

## When NOT to use

- Input is raw m/z imaging data that has not yet been metabolite-annotated — run the SpaMTP annotation pipeline first, since enrichment operates on named metabolites and genes
- Differential expression has not been performed per cluster — Fisher's exact test and rank-based enrichment both require FindAllDEMs-style differential metabolite/gene results as input
- The target pathways lie outside the RaMP-DB-derived annotation sources (KEGG, SMPdb, Reactome, WikiPathways) used by this implementation

## Inputs

- Integrated SpaMTP Seurat object with paired spatial metabolomics (MALDI FMP-10/DHB) and spatial transcriptomics (10X Visium) assays
- Per-cluster differential metabolite results from FindAllDEMs
- Per-cluster differential gene results (differential expression analysis)
- RaMP-DB-derived pathway feature sets (KEGG, SMPdb, Reactome, WikiPathways)
- Graphite-derived pathway topology

## Outputs

- pathway_enrichment_results.csv (pathway IDs, analyte types, region/cluster, enrichment statistics)
- Fisher over-representation results per metabolite/gene analyte type and cluster
- Rank-based (GSEA-like) regional pathway enrichment calls between contrasted clusters
- Enrichment Map pathway network plot with feature sets as nodes and weighted biological-relevance edges

## How to apply

Load the integrated SpaMTP Seurat object holding paired MALDI (e.g. FMP-10/DHB) and Visium assays together with per-cluster differential metabolite (FindAllDEMs) and differential gene results. Run over-representation analysis with FishersPathwayAnalysis, which applies Fisher's exact test to ask whether differentially expressed metabolites and/or genes overlap pathway feature sets more than random feature sets. Complement this with FindRegionalPathways, a GSEA-like rank-based enrichment that accounts for the magnitude and direction of analyte expression changes, to identify pathways differentially expressed between regions such as lesioned versus intact striatum clusters. Build networks with PathwayNetworkPlots (Enrichment Map), using Graphite-derived topology where feature sets are nodes and weighted edges reflect biological relevance while integrating metabolite and gene expression values. Draw pathway sets from RaMP-DB-derived annotations spanning KEGG, SMPdb, Reactome and WikiPathways, then write enrichment statistics (pathway IDs, analyte types, region/cluster, enrichment statistics) to pathway_enrichment_results.csv and render the network plot.

## Related tools

- **SpaMTP** (Provides FishersPathwayAnalysis (Fisher exact over-representation), FindRegionalPathways (rank-based, direction-aware enrichment), and PathwayNetworkPlots (Enrichment Map) over integrated SM/ST Seurat objects) — github.com/GenomicsMachineLearning/SpaMTP
- **Seurat** (Object model carrying the paired MALDI and Visium assays and per-cluster differential results that enrichment functions consume) — https://satijalab.org/seurat/
- **R** (Runtime environment; SpaMTP is an R-based package bridging Cardinal metabolomics processing and Seurat cell-centric analyses)

## Evaluation signals

- Enrichment table recovers the dopamine beta-hydroxylase deficiency pathway as elevated in the damaged striatum
- GPCR alpha-associated signalling is recovered in the urothelium-analog region clusters
- SAM-e and D-dopachrome appear as key metabolites in the network analysis, matching the article's reported findings
- pathway_enrichment_results.csv contains pathway IDs, analyte types, region/cluster labels and enrichment statistics for every call
- Rendered network uses Graphite-derived topology with feature sets as nodes and weighted edges reflecting biological relevance across both analyte types

## Limitations

- Validation depends on the brain case-study expectations; other tissues lack equivalent reported ground-truth pathways
- Pathway coverage is bounded by the RaMP-DB-derived sets (KEGG, SMPdb, Reactome, WikiPathways); no SpaMTP version number, DOI or release identifier is stated for this vignette section
- The article reports no comparison benchmarks or quantified evaluation metrics for the pathway-analysis component

## Evidence

- [readme] (2) various downstream statistical analysis including differential metabolite expression and pathway analysis, and (3) integrative spatial-omics analysis: [evidence span withheld — anchor status: external_doc]
- [readme] SpaMTP is an R package designed for the integrative analysis of spatial metabolomics and spatial transcriptomics data: [evidence span withheld — anchor status: external_doc]
