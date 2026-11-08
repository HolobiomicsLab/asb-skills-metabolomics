---
name: intergenic-distance-clustering
description: Use when you have a gene prediction .tab file with coordinates and strands,
  and need to group adjacent same-strand genes into proximal clusters (proximons)
  as a precursor to operon definition.
when_to_use_negative:
- Input is raw sequencing reads without a gene prediction file — run the MetaRon 'ago'
  process (assembly via IDBA plus gene prediction via Prodigal) first
- Genes are on opposite strands or separated by intergenic distances >= 601 bp — they
  are excluded from proximon clusters by design
- You already have pre-defined operons and only need downstream secondary metabolite
  or pathway analysis (use antiSMASH/BOWTIE-based workflows instead)
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MetaRon
  role: Pipeline that computes intergenic distances between adjacent co-directional
    genes and clusters them into proximons under 'metaron --process op'
  repo: https://github.com/zaidissa/MetaRon
- name: prodigal
  role: Gene predictor whose .tab output (selected with --tool 2) supplies gene coordinates
    and strands used for IGD computation
- name: MetaGeneMark
  role: Alternative gene predictor whose .tab output (selected with --tool 1) supplies
    gene coordinates and strands used for IGD computation
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:2634f561106ac131bf2b3e02ef8a1d7648298a79b0a4df355efa282cd23b69b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - MetaGeneMark
  - MetaRon
  - NNPP
  - Prodigal
  - prodigal
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# intergenic-distance-clustering

## Summary

Clusters adjacent co-directional gene pairs into proximons using an intergenic distance (IGD) threshold of IGD < 601 bp (<= 600 bp), as the first clustering stage of MetaRon's operon prediction before promoter-based boundary refinement.

## When to use

Use when you have a gene prediction .tab file with coordinates and strands, and need to group adjacent same-strand genes into proximal clusters (proximons) as a precursor to operon definition.

## When NOT to use

- Input is raw sequencing reads without a gene prediction file — run the MetaRon 'ago' process (assembly via IDBA plus gene prediction via Prodigal) first
- Genes are on opposite strands or separated by intergenic distances >= 601 bp — they are excluded from proximon clusters by design
- You already have pre-defined operons and only need downstream secondary metabolite or pathway analysis (use antiSMASH/BOWTIE-based workflows instead)

## Inputs

- Gene prediction .tab file from MetaGeneMark or Prodigal (gene start, end, strand per scaffold)
- Scaftigs FASTA file (assembled scaffold sequences, e.g. from IDBA)

## Outputs

- Tab-delimited Operon_File.tab listing co-directional proximal gene clusters (proximons) satisfying IGD <= 600 bp

## How to apply

Load the gene prediction .tab file (generated via MetaGeneMark with --tool 1 or Prodigal with --tool 2) and extract each gene's start coordinate, end coordinate, and strand. Sort genes along each scaffold and, for every adjacent pair of genes on the same strand, compute the intergenic distance IGD(G1,G2) = (start(G2) - end(G1)) + 1. Merge adjacent co-directional gene pairs whose IGD is less than or equal to 600 bp into proximons, leaving pairs exceeding the threshold unjoined. In MetaRon this stage runs via 'metaron --process op', which consumes the --igp (gene prediction) and --isc (scaftigs FASTA) inputs and writes the gene-cluster listing to the tab-delimited Operon_File.tab. On the reference E. coli dataset this produced 822 co-directional proximal gene clusters containing 2955 genes, which downstream promoter prediction (NNPP + Prom_IGD_Clustering) further resolves into operons.

## Related tools

- **MetaRon** (Pipeline that computes intergenic distances between adjacent co-directional genes and clusters them into proximons under 'metaron --process op') — https://github.com/zaidissa/MetaRon
- **prodigal** (Gene predictor whose .tab output (selected with --tool 2) supplies gene coordinates and strands used for IGD computation)
- **MetaGeneMark** (Alternative gene predictor whose .tab output (selected with --tool 1) supplies gene coordinates and strands used for IGD computation)

## Examples

```
metaron --sample ERR022075 --process op --igp ERR022075 --isc ERR022075_scaf.fa --tool 2 --output ~/path/to/output/directory/
```

## Evaluation signals

- Every listed cluster in Operon_File.tab contains only adjacent same-strand gene pairs satisfying IGD <= 600 bp
- Operon_File.tab exists and is tab-delimited
- Cluster counts are consistent with the MetaRon benchmark: 822 co-directional proximal gene clusters (IGD < 601 bp) containing 2955 genes
- No pair of genes joined into a cluster has an intergenic distance >= 601 bp per the formula IGD(G1,G2) = (start(G2) - end(G1)) + 1

## Limitations

- IGD clustering alone does not define operon boundaries — promoter prediction (NNPP + Prom_IGD_Clustering) is required to delimit transcription units and remove non-operonic genes
- NNPP may fail to identify promoters, which limits the downstream boundary refinement step
- Threshold is fixed near 600 bp; gene pairs separated by larger intergenic distances are never merged into proximons

## Evidence

- [results] MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes.: "MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes."
- [results] Identify co-directional proximal gene clusters based on direction and intergenic distance: "The pipeline performs operon prediction with high sensitivity based on co-directionality, intergenic distance, and presence/absence of a promoter upstream and downstream of a gene"
- [readme] Metagenomic operon prediction redefines the operonic clusters by identifying promoters in co-directional genes with an intergenic distance threshold of <= 600 bp.: "Metagenomic operon prediction redefines the operonic clusters by identifying promoters in co-directional genes with an intergenic distance threshold of <= 600 bp."
- [readme] Select the gene prediction .tab file generated via MetageneMark or Prodigal: "Select the gene prediction .tab file generated via MetageneMark or Prodigal"
