---
name: proximon-operon-prediction
description: Use when you have scaffold/contig-level gene predictions (from Prodigal
  or MetaGeneMark) and need to cluster adjacent same-strand genes into proximons based
  on intergenic distance before operon prediction.
when_to_use_negative:
- Input genes come from different strands or are not adjacent along a scaffold — co-directionality
  and adjacency are prerequisites for proximon clustering
- A pre-assembled metagenome or gene predictions are unavailable and raw reads must
  be processed end-to-end — use the 'ago' process (assembly via IDBA, gene prediction
  via prodigal) instead of proximon prediction alone
- Operons must be predicted using functional or experimental annotations — MetaRon's
  proximon/operon prediction is designed to work without any functional information
  and cannot exploit such inputs
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MetaRon
  role: Pipeline that performs proximon prediction by clustering adjacent co-directional
    gene pairs with IGD < 601 bp and emits Operon_File.tab via 'metaron --process
    op'
  repo: https://github.com/zaidissa/MetaRon
- name: Prodigal
  role: Gene prediction tool whose .tab output (--tool 2) supplies gene coordinates
    and strands for proximon clustering
- name: MetaGeneMark
  role: Alternative gene prediction tool whose .tab output (--tool 1) supplies gene
    coordinates and strands for proximon clustering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:07fc40cdb591dc8ebcd3bcca42b538aaca585f0148068581c5ed8f8b0d2b7f0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - MetaGeneMark
  - MetaRon
  - Prodigal
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# proximon-operon-prediction

## Summary

Identifies proximons — clusters of adjacent co-directional genes — from a gene prediction (.tab) file using an intergenic distance (IGD) threshold of < 601 bp, as implemented in the MetaRon operon prediction pipeline.

## When to use

Use when you have scaffold/contig-level gene predictions (from Prodigal or MetaGeneMark) and need to cluster adjacent same-strand genes into proximons based on intergenic distance before operon prediction.

## When NOT to use

- Input genes come from different strands or are not adjacent along a scaffold — co-directionality and adjacency are prerequisites for proximon clustering
- A pre-assembled metagenome or gene predictions are unavailable and raw reads must be processed end-to-end — use the 'ago' process (assembly via IDBA, gene prediction via prodigal) instead of proximon prediction alone
- Operons must be predicted using functional or experimental annotations — MetaRon's proximon/operon prediction is designed to work without any functional information and cannot exploit such inputs

## Inputs

- Gene prediction .tab file generated via MetaGeneMark or Prodigal (start, end, strand per gene)
- Scaftigs file containing all assembled scaffolds (e.g., ERR022075_scaf.fa)

## Outputs

- Operon_File.tab — tab-delimited proximon/operon cluster listing in the specified output destination folder

## How to apply

Load the gene prediction .tab file (produced by MetaGeneMark or Prodigal, selected via the --tool flag) together with the scaftigs file, and extract each gene's start coordinate, end coordinate, and strand. Sort genes along each scaffold and compute the intergenic distance for every adjacent co-directional gene pair as IGD(G1,G2) = (start(G2) - end(G1)) + 1. Merge adjacent same-strand gene pairs whose IGD is less than 601 bp (i.e., IGD <= 600 bp, the README's stated threshold) into proximon clusters; pairs exceeding this distance are not joined. On the reference E. coli K-12 MG1655 run, this criterion yielded 822 co-directional proximal gene clusters containing 2955 genes. The result is written in operon-prediction format to Operon_File.tab in the output folder via 'metaron --process op'.

## Related tools

- **MetaRon** (Pipeline that performs proximon prediction by clustering adjacent co-directional gene pairs with IGD < 601 bp and emits Operon_File.tab via 'metaron --process op') — https://github.com/zaidissa/MetaRon
- **Prodigal** (Gene prediction tool whose .tab output (--tool 2) supplies gene coordinates and strands for proximon clustering)
- **MetaGeneMark** (Alternative gene prediction tool whose .tab output (--tool 1) supplies gene coordinates and strands for proximon clustering)

## Examples

```
metaron --sample ERR022075 --process op --igp ERR022075 --isc ERR022075_scaf.fa --tool 2 --output ~/path/to/output/directory/
```

## Evaluation signals

- Operon_File.tab exists in the specified output folder and is tab-delimited
- Every listed cluster contains only adjacent same-strand gene pairs satisfying IGD <= 600 bp
- Cluster counts are consistent with the MetaRon benchmark: 822 co-directional proximal gene clusters (IGD < 601 bp) containing 2955 genes on the E. coli K-12 MG1655 reference dataset
- Each gene appears in the output with the same start/end/strand as in the input gene prediction .tab file (no genes dropped or duplicated across clusters)
- Run reports no errors and exits cleanly when the correct 'op' process parameters (--sample, --process, --igp, --isc, --tool, --output) are supplied

## Limitations

- The IGD threshold is fixed at <= 600 bp for proximon definition in metagenomic mode; genes separated by more than 600 bp are not joined even if functionally related
- Pipeline restricted to Linux and academic use only (README requires Python 2.7 per prerequisites)
- Promoter prediction (NNPP 2.2) is needed for full operon prediction downstream; NNPP's inability to identify some promoters limits sensitivity, and the NNPP 2.2 path must be added to config.txt
- Secondary metabolite prediction, pathway annotation and graphical representation are not yet integrated into the pipeline (planned future work)

## Evidence

- [results] Co-directional proximal gene clustering threshold: "MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes."
- [readme] README threshold statement: "Metagenomic operon prediction redefines the operonic clusters by identifying promoters in co-directional genes with an intergenic distance threshold of <= 600 bp."
- [intro] Operon prediction basis: "The pipeline performs operon prediction with high sensitivity based on co-directionality, intergenic distance, and presence/absence of a promoter upstream and downstream of a gene"
- [readme] README output description: "This will save metagenomic operon predictions ``Operon_File.tab``.  The prediction file will report the operonic information based on the above mentioned parameters."
