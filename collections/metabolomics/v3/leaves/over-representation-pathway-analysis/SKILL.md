---
name: over-representation-pathway-analysis
description: Use when a SpaMTP-integrated Seurat object has per-cluster differentially
  expressed metabolites (FindAllDEMs) and genes, and you need region-level pathway
  enrichment (e.g. lesioned vs intact striatum).
when_to_use_negative:
- Input is raw m/z data not yet annotated to metabolites or mapped onto a Seurat object
  - run the SpaMTP annotation and integration pipeline first.
- You need annotation-level metabolite prioritisation rather than pathway-level enrichment
  - use CalculateAnnotationStatistics / CalculateSingleAnnotationStatistics instead.
- No differential metabolite or gene contrast exists (e.g. only a single spatial region
  without cluster labels) - over-representation and regional enrichment both require
  defined analyte sets or ranked lists.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3172
tools:
- name: SpaMTP
  role: Provides FishersPathwayAnalysis (over-representation), FindRegionalPathways
    (rank-based regional enrichment) and PathwayNetworkPlots (Enrichment Map network
    visualisation) on a Seurat-class spatial-omics object
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: Seurat
  role: Foundation object class holding the paired spatial metabolomics and transcriptomics
    assays, from which differential metabolite/gene lists are extracted
  repo: https://satijalab.org/seurat/
- name: RaMP
  role: Source of pathway feature sets (KEGG, SMPdb, Reactome, WikiPathways) queried
    for enrichment
- name: Graphite
  role: Source of pathway topology used to build the Enrichment Map network in PathwayNetworkPlots
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:33d4af002dc71677d4546c85fc9b570cf83c22727b67a4ba8bf64997ed00d740
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Graphite
  - R
  - RaMP
  - Seurat
  - SpaMTP
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# over-representation-pathway-analysis

## Summary

Regional pathway enrichment for spatial metabolomics/transcriptomics using Fisher's exact over-representation (FishersPathwayAnalysis) and GSEA-like ranked enrichment (FindRegionalPathways) on differentially expressed metabolites and genes, with Enrichment Map network visualisation via PathwayNetworkPlots.

## When to use

Use when a SpaMTP-integrated Seurat object has per-cluster differentially expressed metabolites (FindAllDEMs) and genes, and you need region-level pathway enrichment (e.g. lesioned vs intact striatum).

## When NOT to use

- Input is raw m/z data not yet annotated to metabolites or mapped onto a Seurat object - run the SpaMTP annotation and integration pipeline first.
- You need annotation-level metabolite prioritisation rather than pathway-level enrichment - use CalculateAnnotationStatistics / CalculateSingleAnnotationStatistics instead.
- No differential metabolite or gene contrast exists (e.g. only a single spatial region without cluster labels) - over-representation and regional enrichment both require defined analyte sets or ranked lists.

## Inputs

- Integrated SpaMTP Seurat object (assays 'SPM' for MALDI FMP-10/DHB, 'SPT' for 10X Visium)
- Per-cluster differential metabolite and gene lists from FindAllDEMs
- RaMP-DB-derived pathway feature sets (KEGG, SMPdb, Reactome, WikiPathways)
- Graphite-derived pathway topology

## Outputs

- Pathway enrichment statistics table (pathway IDs, analyte types, region/cluster, enrichment statistics)
- Pathway network plot (Enrichment Map) with feature sets as nodes and weighted edges

## How to apply

Start from the integrated SpaMTP Seurat object for paired MALDI (FMP-10/DHB, assay 'SPM') and Visium ('SPT') data and run FindAllDEMs to obtain differential metabolite and gene lists per integrated cluster. Apply FishersPathwayAnalysis to test whether differentially expressed metabolites and/or genes overlap pathway feature sets more than expected for random feature sets (over-representation), and FindRegionalPathways for rank-based, GSEA-like enrichment that accounts for the magnitude and direction of analyte expression changes between lesioned and intact striatum clusters. Pull pathway feature sets from RaMP-DB annotations spanning KEGG, SMPdb, Reactome and WikiPathways, with topology from Graphite, as documented in the Pathway Database Integration vignette. Visualise results with PathwayNetworkPlots (Enrichment Map), where feature sets are nodes and weighted edges reflect biological relevance and integrated metabolite/gene expression values. Write the enrichment table (pathway IDs, analyte types, region/cluster, enrichment statistics) for downstream validation against known biology such as dopamine beta-hydroxylase deficiency elevation in the damaged striatum.

## Related tools

- **SpaMTP** (Provides FishersPathwayAnalysis (over-representation), FindRegionalPathways (rank-based regional enrichment) and PathwayNetworkPlots (Enrichment Map network visualisation) on a Seurat-class spatial-omics object) — github.com/GenomicsMachineLearning/SpaMTP
- **Seurat** (Foundation object class holding the paired spatial metabolomics and transcriptomics assays, from which differential metabolite/gene lists are extracted) — https://satijalab.org/seurat/
- **RaMP** (Source of pathway feature sets (KEGG, SMPdb, Reactome, WikiPathways) queried for enrichment)
- **Graphite** (Source of pathway topology used to build the Enrichment Map network in PathwayNetworkPlots)

## Examples

```
R> FishersPathwayAnalysis(SPM, dems = FindAllDEMs(SPM)) ; FindRegionalPathways(SPM, ranks = DE_results) ; PathwayNetworkPlots(enrichment_results)
```

## Evaluation signals

- Enrichment table recovers the dopamine beta-hydroxylase deficiency pathway as elevated in the damaged striatum (lesioned) clusters.
- GPCR alpha-associated signalling appears enriched in the urothelium-analog region clusters.
- Network analysis identifies SAM-e and D-dopachrome as key metabolite nodes.
- Every enrichment row carries pathway ID, analyte type (metabolite/gene), region/cluster and enrichment statistics fields.
- Enrichment ranks respect both magnitude and direction of expression change (GSEA-like scoring), not just set membership.

## Limitations

- Pathway membership and topology depend on the RaMP-DB snapshot and Graphite conversion, which must be rebuilt/documented per the Pathway Database Integration vignette - results will change with database updates.
- Over-representation testing requires a pre-defined analyte contrast; it cannot discover pathways from un-clustered, unlabeled spatial data.
- No benchmark metrics or hardware/environment requirements are stated in the article for the enrichment module itself.

## Evidence

- [methods] FishersPathwayAnalysis uses Fisher's exact test: [evidence span withheld — anchor status: not_found]
- [methods] FindRegionalPathways accounts for magnitude and direction: [evidence span withheld — anchor status: not_found]
- [methods] PathwayNetworkPlots builds Enrichment Map: [evidence span withheld — anchor status: not_found]
- [results] Validation: dopamine beta-hydroxylase deficiency pathway: [evidence span withheld — anchor status: not_found]
- [readme] SpaMTP published citation: [evidence span withheld — anchor status: external_doc]
