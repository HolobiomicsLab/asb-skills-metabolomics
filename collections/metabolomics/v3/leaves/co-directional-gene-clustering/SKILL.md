---
name: co-directional-gene-clustering
description: Use when a gene prediction .tab file (MetaGeneMark or Prodigal output)
  plus a scaftig file are available and you need to identify proximal co-directional
  gene clusters (proximons) as the basis for operon prediction in shotgun metagenomic
  data.
when_to_use_negative:
- Raw reads rather than scaftigs and gene predictions are the input — use 'metaron
  --process ago' (assembly, gene prediction and operon prediction) instead of 'op'
- Genes are on opposite strands — MetaRon's proximons are defined only for adjacent
  co-directional gene pairs
- Functional or experimental information is required for the clustering — MetaRon
  deliberately predicts operons without any functional or experimental data, so this
  skill is not appropriate when annotation-driven grouping is wanted
edam_operation: http://edamontology.org/operation_3903
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaRon
  role: 'Implements the proximons identification: merges adjacent co-directional gene
    pairs with IGD <= 600 bp and emits the cluster listing'
  repo: https://github.com/zaidissa/MetaRon
- name: Prodigal
  role: Produces the gene prediction .tab file (input option --tool 2) whose gene
    coordinates feed the co-directional clustering
- name: MetaGeneMark
  role: Alternative gene predictor producing the .tab input (input option --tool 1)
    for proximons identification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:25b2f796842c6c0c899fac035297a4a30c15cf87c9fc95ab40004e81624c4dab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - MetaGeneMark
  - MetaRon
  - Prodigal
  edam_operation: http://edamontology.org/operation_3903
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# co-directional-gene-clustering

## Summary

Clusters adjacent co-directional gene pairs into proximons using an intergenic distance (IGD) threshold of <= 600 bp (pipeline-level criterion IGD < 601 bp), as implemented in MetaRon for metagenomic and whole-genome operon prediction.

## When to use

Use when a gene prediction .tab file (MetaGeneMark or Prodigal output) plus a scaftig file are available and you need to identify proximal co-directional gene clusters (proximons) as the basis for operon prediction in shotgun metagenomic data.

## When NOT to use

- Raw reads rather than scaftigs and gene predictions are the input — use 'metaron --process ago' (assembly, gene prediction and operon prediction) instead of 'op'
- Genes are on opposite strands — MetaRon's proximons are defined only for adjacent co-directional gene pairs
- Functional or experimental information is required for the clustering — MetaRon deliberately predicts operons without any functional or experimental data, so this skill is not appropriate when annotation-driven grouping is wanted

## Inputs

- Gene prediction .tab file generated via MetaGeneMark or Prodigal (coordinates and strand per gene)
- Scaftig file containing all scaftigs (e.g. ERR022075_scaf.fa)
- Tool selector: 1 for MetaGeneMark, 2 for Prodigal

## Outputs

- Operon_File.tab — tab-delimited proximon/operonic cluster listing written to the specified output destination folder

## How to apply

Load the gene prediction .tab file via 'metaron --process op' (set --tool 1 for MetaGeneMark or 2 for Prodigal) and --isc with the scaftig file containing all scaftigs. For every adjacent pair of genes on the same strand, compute the intergenic distance IGD(G1,G2) = (start(G2) - end(G1)) + 1 and merge pairs whose IGD is <= 600 bp (criterion IGD < 601 bp); genes separated by more than 600 bp are not joined into a proximon. Write the resulting gene-cluster listing to Operon_File.tab in the output folder. On the E. coli K-12 MG1655 reference run, this criterion yielded 822 co-directional proximal gene clusters containing 2955 genes, which can serve as a sanity benchmark.

## Related tools

- **MetaRon** (Implements the proximons identification: merges adjacent co-directional gene pairs with IGD <= 600 bp and emits the cluster listing) — https://github.com/zaidissa/MetaRon
- **Prodigal** (Produces the gene prediction .tab file (input option --tool 2) whose gene coordinates feed the co-directional clustering)
- **MetaGeneMark** (Alternative gene predictor producing the .tab input (input option --tool 1) for proximons identification)

## Examples

```
metaron --sample ERR022075 --process op --igp ERR022075 --isc ERR022075_scaf.fa --tool 2 --output ~/path/to/output/directory/
```

## Evaluation signals

- Operon_File.tab exists and is tab-delimited
- Every listed cluster contains only adjacent same-strand gene pairs satisfying IGD <= 600 bp (i.e. IGD < 601 bp)
- Recomputation of IGD = (start(G2) - end(G1)) + 1 for each adjacent pair matches the threshold criterion
- Cluster/gene counts are consistent with the MetaRon benchmark: 822 co-directional proximal gene clusters containing 2955 genes on the reference dataset
- The selected --tool flag (1 = MetaGeneMark, 2 = Prodigal) matches the actual format of the provided .tab file

## Limitations

- The proximons criterion is a fixed maximum intergenic distance of 600 bp; genes separated by more than this threshold are never joined, which may split true operons with long spacer regions
- Cluster definition depends on the accuracy of upstream gene prediction via MetaGeneMark or Prodigal and on scaftig assembly quality (e.g. scaftigs <= 500 bp were removed)
- Promoter prediction via NNPP 2.2 is a required prerequisite (NNPP2.2 path must be added to config.txt), and NNPP was reported unable to identify some promoters

## Evidence

- [readme] Metagenomic operon prediction redefines the operonic clusters by identifying promoters in co-directional genes with an intergenic distance threshold of <= 600 bp.: "Metagenomic operon prediction redefines the operonic clusters by identifying promoters in co-directional genes with an intergenic distance threshold of <= 600 bp."
- [results] MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes.: "MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes."
- [intro] The pipeline performs operon prediction with high sensitivity based on co-directionality, intergenic distance, and presence/absence of a promoter upstream and downstream of a gene: "The pipeline performs operon prediction with high sensitivity based on co-directionality, intergenic distance, and presence/absence of a promoter upstream and downstream of a gene"
- [readme] Select the gene prediction .tab file generated via MetageneMark or Prodigal: "Select the gene prediction .tab file generated via MetageneMark or Prodigal"
