---
name: multi-omic-pathway-database-querying
description: Use when a paired spatial metabolomics + transcriptomics dataset (e.g.
  integrated Seurat object with MALDI and Visium assays) has per-cluster differential
  metabolite and gene results and you need regional pathway enrichment.
when_to_use_negative:
- The m/z values have not yet been annotated to metabolite and gene identifiers —
  pathway overlap tests require annotated feature sets, so run the SpaMTP annotation
  pipeline first
- The dataset is metabolomics-only with no transcriptomic modality, since the pipeline
  is built around metabolite-gene based downstream statistical analyses on integrated
  Seurat objects
- You only need single-m/z annotation statistics (use CalculateSingleAnnotationStatistics
  / CalculateAnnotationStatistics instead of pathway enrichment)
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3388
- http://edamontology.org/topic_3308
tools:
- name: SpaMTP
  role: R package supplying FishersPathwayAnalysis (Fisher's exact ORA), FindRegionalPathways
    (GSEA-like rank-based regional enrichment) and PathwayNetworkPlots (Enrichment
    Map networks)
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: Seurat
  role: Provides the Seurat-class object foundation holding the paired metabolite
    and gene assays used as enrichment input
- name: Cardinal
  role: Upstream processing of the MALDI spatial metabolomics data that feeds the
    integrated object
- name: RaMP
  role: Source of the pathway feature sets queried for enrichment (KEGG, SMPdb, Reactome,
    WikiPathways)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:346e2365042d27b173bcbcf1ddaefbb28f5eb2809a1450e254438e10a9b2a4d1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cardinal
  - R
  - RaMP
  - Seurat
  - SpaMTP
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3388
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# multi-omic-pathway-database-querying

## Summary

Joint metabolite- and gene-level pathway enrichment within SpaMTP's downstream statistical pipeline, using Fisher's exact over-representation and GSEA-like rank-based tests against RaMP-DB pathway sets, followed by Graphite-topology Enrichment Map network construction.

## When to use

Use when a paired spatial metabolomics + transcriptomics dataset (e.g. integrated Seurat object with MALDI and Visium assays) has per-cluster differential metabolite and gene results and you need regional pathway enrichment.

## When NOT to use

- The m/z values have not yet been annotated to metabolite and gene identifiers — pathway overlap tests require annotated feature sets, so run the SpaMTP annotation pipeline first
- The dataset is metabolomics-only with no transcriptomic modality, since the pipeline is built around metabolite-gene based downstream statistical analyses on integrated Seurat objects
- You only need single-m/z annotation statistics (use CalculateSingleAnnotationStatistics / CalculateAnnotationStatistics instead of pathway enrichment)

## Inputs

- Integrated SpaMTP Seurat object containing paired spatial metabolomics (MALDI) and spatial transcriptomics assays
- Per-cluster differential metabolite results from FindAllDEMs
- Per-cluster differential gene expression results
- RaMP-DB-derived pathway annotation sets (KEGG, SMPdb, Reactome, WikiPathways)

## Outputs

- pathway_enrichment_results.csv with pathway IDs, analyte types, region/cluster assignments and enrichment statistics
- Pathway network plot (Enrichment Map) with pathway feature sets as nodes and weighted edges reflecting Graphite-derived biological relevance

## How to apply

Load the integrated SpaMTP Seurat object (e.g. the Parkinson's mouse brain 'SPM'/'SPT' MALDI FMP-10/DHB + 10X Visium assays) together with per-cluster FindAllDEMs metabolite and differential gene results. First run over-representation analysis with FishersPathwayAnalysis, which uses Fisher's exact test to ask whether differentially expressed metabolites and/or genes overlap RaMP-DB-derived pathway feature sets (KEGG, SMPdb, Reactome, WikiPathways) more than random feature sets. Then run rank-based enrichment with FindRegionalPathways, which accounts for the magnitude and direction of analyte expression changes (GSEA-like) to identify pathways differentially expressed between e.g. lesioned and intact striatum clusters. Finally build pathway networks with PathwayNetworkPlots (Enrichment Map), where feature sets are nodes and weighted edges reflect biological relevance via Graphite-derived topology, integrating both metabolite and gene expression values. Write enrichment statistics (pathway IDs, analyte types, region/cluster, enrichment statistics) to pathway_enrichment_results.csv and render the network plot.

## Related tools

- **SpaMTP** (R package supplying FishersPathwayAnalysis (Fisher's exact ORA), FindRegionalPathways (GSEA-like rank-based regional enrichment) and PathwayNetworkPlots (Enrichment Map networks)) — github.com/GenomicsMachineLearning/SpaMTP
- **Seurat** (Provides the Seurat-class object foundation holding the paired metabolite and gene assays used as enrichment input)
- **Cardinal** (Upstream processing of the MALDI spatial metabolomics data that feeds the integrated object)
- **RaMP** (Source of the pathway feature sets queried for enrichment (KEGG, SMPdb, Reactome, WikiPathways))

## Examples

```
devtools::install_github("GenomicsMachineLearning/SpaMTP"); FishersPathwayAnalysis(SPM, DEMs, DEGs); FindRegionalPathways(SPM, DE_results)
```

## Evaluation signals

- pathway_enrichment_results.csv recovers the dopamine beta-hydroxylase deficiency pathway as elevated in the damaged striatum
- Enrichment table shows GPCR alpha-associated signalling in the urothelium-analog region clusters
- Network analysis identifies SAM-e and D-dopachrome as key metabolites
- Output schema contains pathway IDs, analyte types, region/cluster labels and enrichment statistics
- Both Fisher's exact ORA and rank-based FindRegionalPathways results are consistent with the article's reported findings

## Limitations

- Pathway feature sets are limited to those derived from RaMP-DB (KEGG, SMPdb, Reactome, WikiPathways); other sources are not queried
- Network construction depends on Graphite-derived pathway topology and the interaction-code correction documented in the Pathway Database Integration vignette
- No specific SpaMTP version is pinned in the article; reproduce with the GitHub install (devtools::install_github) and record the installed version

## Evidence

- [readme] README confirms pathway analysis as a major SpaMTP functionality.: [evidence span withheld — anchor status: external_doc]
