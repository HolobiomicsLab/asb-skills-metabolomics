---
name: spatial-region-differential-analyte-interpretation
description: Use when an integrated SpaMTP Seurat object (paired SM/ST assays) already
  has per-cluster differential metabolites and genes, and the goal is region-level
  pathway enrichment or pathway-network interpretation across spatial regions.
when_to_use_negative:
- The input is raw m/z data without metabolite annotations or differential metabolite/gene
  results per cluster — the annotation and FindAllDEMs steps must precede pathway
  interpretation
- The goal is metabolite annotation refinement (e.g. CalculateAnnotationStatistics,
  RefineLipids) rather than downstream regional pathway-level interpretation
- The dataset has no spatial region/cluster structure to contrast (e.g. a single bulk
  ROI), so regional enrichment between lesioned and intact areas cannot be defined
edam_operation: http://edamontology.org/operation_3933
tools:
- name: SpaMTP
  role: Provides FishersPathwayAnalysis, FindRegionalPathways and PathwayNetworkPlots
    used to perform the over-representation, rank-based enrichment and Enrichment
    Map network analysis
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: Seurat
  role: 'Object foundation: the integrated SM-ST data is held in a Seurat Class Object
    on which the differential results and enrichment functions operate'
  repo: https://satijalab.org/seurat/
- name: R
  role: Implementation language for running the SpaMTP pathway analysis pipeline end
    to end
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:ca483f497abde8aeed4bd520d790ab6c5d0d31de7a0945c1139a36094fa23899
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - Seurat
  - SpaMTP
  edam_operation: http://edamontology.org/operation_3933
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spatial-region-differential-analyte-interpretation

## Summary

Regional interpretation of spatial metabolomics/transcriptomics differences in SpaMTP: over-representation (Fisher's exact) and rank-based (GSEA-like) pathway enrichment plus Enrichment Map network construction on integrated SM-ST data, as demonstrated on the Parkinson's mouse brain case study.

## When to use

Use when an integrated SpaMTP Seurat object (paired SM/ST assays) already has per-cluster differential metabolites and genes, and the goal is region-level pathway enrichment or pathway-network interpretation across spatial regions.

## When NOT to use

- The input is raw m/z data without metabolite annotations or differential metabolite/gene results per cluster — the annotation and FindAllDEMs steps must precede pathway interpretation
- The goal is metabolite annotation refinement (e.g. CalculateAnnotationStatistics, RefineLipids) rather than downstream regional pathway-level interpretation
- The dataset has no spatial region/cluster structure to contrast (e.g. a single bulk ROI), so regional enrichment between lesioned and intact areas cannot be defined

## Inputs

- Integrated SpaMTP Seurat object with paired spatial metabolomics (MALDI, FMP-10/DHB matrix) and spatial transcriptomics (10X Visium) assays ('SPM'/'SPT')
- Per-cluster differential metabolite results from FindAllDEMs
- Per-cluster differential gene expression results
- RaMP-DB-derived pathway feature sets (KEGG, SMPdb, Reactome, WikiPathways)

## Outputs

- pathway_enrichment_results.csv containing pathway IDs, analyte types, region/cluster labels and enrichment statistics
- Enrichment Map pathway network plot (Graphite-derived topology, feature sets as nodes with weighted edges)
- Ranked lists of regionally enriched pathways per cluster/region

## How to apply

Load the integrated SpaMTP Seurat object holding paired spatial metabolomics and transcriptomics assays (e.g. MALDI FMP-10/DHB and 10X Visium 'SPM'/'SPT' in the Parkinson's mouse brain study) together with FindAllDEMs and differential gene results per integrated cluster. Run over-representation analysis with FishersPathwayAnalysis, which uses Fisher's exact test to ask whether differentially expressed metabolites and/or genes overlap pathway feature sets more than random sets. Complement this with FindRegionalPathways, a GSEA-like rank-based enrichment that accounts for the magnitude and direction of analyte expression changes to identify pathways differentially expressed between regions (e.g. lesioned vs intact striatum clusters). Build pathway networks with PathwayNetworkPlots (Enrichment Map), using Graphite-derived topology where feature sets are nodes and weighted edges reflect biological relevance while integrating metabolite and gene expression values. Query RaMP-DB-derived pathway sets (KEGG, SMPdb, Reactome, WikiPathways) as the annotation source, then write pathway IDs, analyte types, region/cluster labels and enrichment statistics to pathway_enrichment_results.csv and render the network plot.

## Related tools

- **SpaMTP** (Provides FishersPathwayAnalysis, FindRegionalPathways and PathwayNetworkPlots used to perform the over-representation, rank-based enrichment and Enrichment Map network analysis) — github.com/GenomicsMachineLearning/SpaMTP
- **Seurat** (Object foundation: the integrated SM-ST data is held in a Seurat Class Object on which the differential results and enrichment functions operate) — https://satijalab.org/seurat/
- **R** (Implementation language for running the SpaMTP pathway analysis pipeline end to end)

## Examples

```
devtools::install_github("GenomicsMachineLearning/SpaMTP")
```

## Evaluation signals

- The enrichment table must recover the dopamine beta-hydroxylase deficiency pathway as elevated in the damaged striatum
- GPCR alpha-associated signalling should appear in the urothelium-analog region clusters
- SAM-e and D-dopachrome must surface as key metabolites in the Enrichment Map network analysis
- pathway_enrichment_results.csv must contain the expected schema: pathway IDs, analyte types, region/cluster, enrichment statistics
- Fisher's exact ORA and rank-based FindRegionalPathways results should be directionally consistent with the FindAllDEMs differential metabolite/gene calls per cluster

## Limitations

- Over-representation analysis (Fisher's exact test) ignores the magnitude and direction of analyte changes; rank-based FindRegionalPathways is needed when effect direction matters
- Pathway coverage depends on RaMP-DB-derived sets (KEGG, SMPdb, Reactome, WikiPathways); metabolites absent from these sets cannot be mapped
- No SpaMTP version number, benchmark metrics, or quantified case-study outcomes are stated in the article sections provided

## Evidence

- [readme] various downstream statistical analysis including differential metabolite expression and pathway analysis, and (3) integrative spatial-omics analysis: [evidence span withheld — anchor status: external_doc]
