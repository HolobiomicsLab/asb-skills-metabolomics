---
name: rank-aggregation-across-prioritization-tools
description: Use when candidate gene lists from Exomiser variant filtering and metPropagate
  LPA propagation are available and you suspect either single-tool ranking is biased
  — e.g. metPropagate over-ranking genes with many metabolic first-degree STRING neighbors.
when_to_use_negative:
- No STRING network and/or HMDB gene-metabolite annotations are available, since p
  cannot be computed without first-degree neighbor metabolic annotation percentages
- Only variant/exome data exist with no patient untargeted metabolomics, so there
  is no metPropagate score to combine — Exomiser alone is the appropriate ranking
- Candidate gene lists come from trio WES rather than singleton exomes, which the
  authors note shortens lists (55–115 vs 281–609 genes), changing the top-20th-percentile
  rank threshold calibration
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3366
tools:
- name: metPropagate
  role: Provides the propagated per-gene metabolomic scores (LPA output) that form
    one additive term of the combined ranking
  repo: https://github.com/emmagraham/metPropagate
- name: Exomiser
  role: Provides the phenotype-based (hiPHIVE-driven) candidate gene scores and variant-filtered
    gene lists that form the other additive term of the combined ranking
- name: STRING
  role: Source of the v11 functional linkage network (19,354 genes) over which first-degree
    neighbor metabolic percentages are computed to derive the weight p
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:97770fe3a429d42dccfec1d15fabf3acd3d015d359b579c6993a1d67d536019f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmagraham/metPropagate
  tools:
  - Exomiser
  - STRING
  - metPropagate
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# rank-aggregation-across-prioritization-tools

## Summary

A weighted additive rank-aggregation scheme that combines metPropagate's propagated metabolomic scores with Exomiser's phenotype-based scores, using the gene's HMDB metabolic-neighbor fraction (p) as the weight to dampen metPropagate's bias toward metabolite-rich genes.

## When to use

Use when candidate gene lists from Exomiser variant filtering and metPropagate LPA propagation are available and you suspect either single-tool ranking is biased — e.g. metPropagate over-ranking genes with many metabolic first-degree STRING neighbors.

## When NOT to use

- No STRING network and/or HMDB gene-metabolite annotations are available, since p cannot be computed without first-degree neighbor metabolic annotation percentages
- Only variant/exome data exist with no patient untargeted metabolomics, so there is no metPropagate score to combine — Exomiser alone is the appropriate ranking
- Candidate gene lists come from trio WES rather than singleton exomes, which the authors note shortens lists (55–115 vs 281–609 genes), changing the top-20th-percentile rank threshold calibration

## Inputs

- Per-patient metPropagate propagated (LPA output) gene scores, e.g. integration/LPA_output sample files with 'Score.ID' and 'Original.Score.ID' columns
- Per-patient Exomiser-Phenotype candidate gene scores derived from DeepVariant singleton VCFs (281–609 candidate genes per TIDEX patient)
- STRING v11 functional linkage network covering all 19,354 genes (graph file plus gene-to-nodeid mapping)
- HMDB gene-metabolite annotations (HMDB_db_ver4.csv) used to identify genes' metabolic first-degree neighbors

## Outputs

- combined_rankings.csv containing per-gene combined scores C, combined ranks, Exomiser ranks, metPropagate ranks, and causative-gene status per patient
- Per-patient summary counts: number of patients with a top-20th-percentile causative gene and number with improved causative-gene rank

## How to apply

For each of the 11 TIDEX patients, load per-gene metPropagate LPA propagated scores and Exomiser-Phenotype candidate gene scores, scaling both vectors between 0 and 1. Using the STRING v11 functional network and HMDB gene-metabolite annotations, compute for every gene the percentage of first-degree neighbors annotated in HMDB, and bin all 19,354 STRING genes into ten bins of 10% intervals (0–10%, 10–20%, etc.). Within each bin compute p, the percentage of genes ranked in the top 20th percentile of all 19,354 genes, so that p captures how strongly that neighborhood region is metabolite-associated. Then compute the combined score C = p*Exomiser + (1-p)*metPropagate for each candidate gene (rescaling each term to 0–1 before summing) and rank each patient's candidates by C. The rationale is that p shifts weight toward Exomiser for genes in metabolite-dense neighborhoods, where metPropagate's label propagation is least informative, and toward metPropagate for genes whose metabolites are undetectable or poorly annotated.

## Related tools

- **metPropagate** (Provides the propagated per-gene metabolomic scores (LPA output) that form one additive term of the combined ranking) — https://github.com/emmagraham/metPropagate
- **Exomiser** (Provides the phenotype-based (hiPHIVE-driven) candidate gene scores and variant-filtered gene lists that form the other additive term of the combined ranking)
- **STRING** (Source of the v11 functional linkage network (19,354 genes) over which first-degree neighbor metabolic percentages are computed to derive the weight p)

## Examples

```
df['C'] = p*df['exomiser_scaled'] + (1-p)*df['metpropagate_scaled']; df.to_csv('combined_rankings.csv')
```

## Evaluation signals

- Validation target: at least one causative gene falls in the top 20th percentile (rank <= 60 for 300-gene lists) in 10/11 TIDEX patients
- For 8/11 patients, the causative gene's combined ranking is higher than under Exomiser alone or metPropagate alone
- All rescaled score terms and the combined score C fall within [0, 1] for every candidate gene
- All 19,354 STRING genes fall into exactly one of the ten 10%-interval bins, and p values are percentages in [0, 100] per bin
- Per-patient combined-score vectors cover the same candidate gene sets (281–609 genes) as the Exomiser-filtered inputs

## Limitations

- Feature-to-metabolite mapping was unavailable for the TIDEX metabolomic system, so exact metabolite identities could not be determined for the metabolomic term being aggregated
- The metabolic system could not measure large glycosaminoglycans, limiting ME analysis for the IDS patient
- Metabolites commonly used as IEM biomarkers (e.g. acyl glycines, acyl carnitines) are not annotated to IEM-associated genes in HMDB, weakening the HMDB-based neighbor weighting for those genes
- No software version reported for the STRING network or for Exomiser/hiPHIVE, complicating exact reproduction

## Evidence

- [abstract] Weighted combination formula and validation counts: "ranking by a weighted combination of metPropagate and Exomiser scores resulted in improved prioritization"
- [discussion] Top-20th-percentile threshold calibration for candidate gene lists: "candidate gene lists for a single patient included between 281 and 609 genes, meaning that the top 20th percentile included 55–115 genes"
- [results] Baseline single-tool comparison that the aggregation must beat: "ranked the causative gene more highly than Exomiser’s phenotype-based ranking in 6/11 patients"
- [readme] Origin of the metPropagate scores being aggregated: "LPA_output contains the post-propagation node scores (automatically generated)"
- [results] Origin of the Exomiser candidate gene lists being aggregated: "Singleton VCFs for all patient were created by DeepVariant, which were then filtered by Exomiser to generate a list of candidate genes"
- [results] Precedent for 0–1 scaling of scores in the pipeline: "DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network"
