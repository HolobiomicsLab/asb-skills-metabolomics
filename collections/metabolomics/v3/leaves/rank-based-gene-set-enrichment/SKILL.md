---
name: rank-based-gene-set-enrichment
description: Use when a paired spatial metabolomics–transcriptomics Seurat object
  has per-cluster differential metabolite (FindAllDEMs) and gene (DEG) results, and
  you need region-level pathway enrichment statistics.
when_to_use_negative:
- The m/z features have not yet been annotated to metabolite names — run SpaMTP's
  annotation pipeline (BuildMZAnnotationIndex/QueryMZAnnotationIndex) first, since
  enrichment operates on named analytes
- Only a single modality (metabolites or genes) is available and no ranked or binary
  differential analyte list exists to test against pathway feature sets
- The goal is pseudo-MS/MS structural annotation of full-scan data rather than set-
  or rank-based pathway testing (use ms1_id for the former)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SpaMTP
  role: Provides FishersPathwayAnalysis (ORA), FindRegionalPathways (rank-based GSEA-like
    enrichment) and PathwayNetworkPlots (Enrichment Map)
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: Seurat
  role: Object framework holding the paired SPM/SPT assays and per-cluster differential
    results used as enrichment input
  repo: https://satijalab.org/seurat/
- name: fgsea
  role: Bioconductor dependency underpinning rank-based (GSEA-like) enrichment in
    SpaMTP
- name: RaMP
  role: Source of pathway feature sets (KEGG, SMPdb, Reactome, WikiPathways) queried
    as the annotation database
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:c2ba6839d5f3873da447d2fabce4be0f163a60ea1a33307d0ac52c73dd9eee7e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - RaMP
  - Seurat
  - SpaMTP
  - fgsea
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# rank-based-gene-set-enrichment

## Summary

SpaMTP's pathway enrichment module for spatial metabolomics/transcriptomics: Fisher's exact over-representation analysis (FishersPathwayAnalysis) plus GSEA-like rank-based regional enrichment (FindRegionalPathways) applied to differentially expressed metabolites and genes, using RaMP-DB-derived pathway sets (KEGG, SMPdb, Reactome, WikiPathways).

## When to use

Use when a paired spatial metabolomics–transcriptomics Seurat object has per-cluster differential metabolite (FindAllDEMs) and gene (DEG) results, and you need region-level pathway enrichment statistics.

## When NOT to use

- The m/z features have not yet been annotated to metabolite names — run SpaMTP's annotation pipeline (BuildMZAnnotationIndex/QueryMZAnnotationIndex) first, since enrichment operates on named analytes
- Only a single modality (metabolites or genes) is available and no ranked or binary differential analyte list exists to test against pathway feature sets
- The goal is pseudo-MS/MS structural annotation of full-scan data rather than set- or rank-based pathway testing (use ms1_id for the former)

## Inputs

- Integrated SpaMTP/Seurat object with paired spatial metabolomics (SPM) and transcriptomics (SPT) assays
- Per-cluster differential metabolite results from FindAllDEMs
- Per-cluster differential gene expression results
- RaMP-DB-derived pathway feature sets (KEGG, SMPdb, Reactome, WikiPathways)

## Outputs

- pathway_enrichment_results.csv (pathway IDs, analyte types, region/cluster, enrichment statistics)
- PathwayNetworkPlots Enrichment Map network (feature sets as nodes, weighted Graphite topology edges)

## How to apply

Load the integrated SpaMTP Seurat object (e.g. the Parkinson's mouse brain paired MALDI FMP-10/DHB and 10X Visium assays 'SPM'/'SPT') together with the differential metabolite and gene results per integrated cluster. For set-based testing, run FishersPathwayAnalysis, which uses Fisher's exact test to ask whether differentially expressed metabolites and/or genes overlap RaMP-DB pathway feature sets more than random feature sets. For rank-based testing, run FindRegionalPathways, which accounts for the magnitude and direction of analyte expression changes (GSEA-like, backed by the fgsea dependency) to identify pathways differentially expressed between e.g. lesioned and intact striatum clusters. Build Enrichment Map-style networks with PathwayNetworkPlots, using Graphite-derived topology with feature sets as nodes and weighted edges reflecting biological relevance, integrating both metabolite and gene expression values. Write enrichment statistics (pathway IDs, analyte types, region/cluster, statistics) to pathway_enrichment_results.csv and render the network plot.

## Related tools

- **SpaMTP** (Provides FishersPathwayAnalysis (ORA), FindRegionalPathways (rank-based GSEA-like enrichment) and PathwayNetworkPlots (Enrichment Map)) — github.com/GenomicsMachineLearning/SpaMTP
- **Seurat** (Object framework holding the paired SPM/SPT assays and per-cluster differential results used as enrichment input) — https://satijalab.org/seurat/
- **fgsea** (Bioconductor dependency underpinning rank-based (GSEA-like) enrichment in SpaMTP)
- **RaMP** (Source of pathway feature sets (KEGG, SMPdb, Reactome, WikiPathways) queried as the annotation database)

## Examples

```
regional_pwys <- FindRegionalPathways(SPM)
```

## Evaluation signals

- The enrichment table must recover the dopamine beta-hydroxylase deficiency pathway as elevated in the damaged striatum, matching the article's Parkinson's brain case study
- GPCR alpha-associated signalling must be recovered in the urothelium-analog region clusters
- SAM-e and D-dopachrome must appear as key metabolites in the PathwayNetworkPlots network analysis
- pathway_enrichment_results.csv must contain pathway IDs, analyte types, region/cluster labels, and enrichment statistics for each tested pathway
- ORA results (Fisher's exact test) and rank-based results (magnitude/direction-aware) should be distinguishable per pathway in the output

## Limitations

- Paired targeted metabolomics-based refinement is a placeholder ('To come!') in the vignette, so enrichment currently relies on the standard RaMP-DB annotation path
- No SpaMTP version number or DOI/release identifier is stated in the article, so exact software provenance for the enrichment run must be recorded separately by the user
- Downstream pathway analysis requires upstream metabolite annotation; tools like METASPACE and pySM are optimised for high-resolution instruments and do not yet apply to most TOF data, which can limit which analytes enter enrichment

## Evidence

- [methods] FishersPathwayAnalysis ORA description: [evidence span withheld — anchor status: not_found]
- [methods] FindRegionalPathways rank-based description: [evidence span withheld — anchor status: not_found]
- [methods] RaMP-DB pathway source: [evidence span withheld — anchor status: not_found]
- [results] Case-study validation expectation: [evidence span withheld — anchor status: not_found]
- [readme] README: pathway analysis in SpaMTP's functionality: [evidence span withheld — anchor status: external_doc]
