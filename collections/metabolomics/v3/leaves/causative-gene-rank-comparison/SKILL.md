---
name: causative-gene-rank-comparison
description: Use when patients have both a metPropagate post-propagation score file
  and a competing prioritization tool's score list (e.g. Exomiser hiPHIVE), plus a
  known causative gene, and you need a head-to-head ranking comparison.
when_to_use_negative:
- When the causative gene is unknown or unconfirmed — there is no ground-truth rank
  to compare against.
- When only one prioritization score is available per patient — a rank comparison
  needs at least two scoring schemes.
- When candidate gene lists use incompatible gene identifiers without a mapping, so
  a gene's rank cannot be traced across schemes.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0091
tools:
- name: metPropagate
  role: Produces the per-patient post-propagation candidate gene scores (Score.ID)
    that are ranked to obtain the causative gene's rank under the network-propagation
    scheme.
  repo: https://github.com/emmagraham/metPropagate
- name: Exomiser (hiPHIVE phenotype algorithm)
  role: Provides the competing phenotype-driven score used to rank each patient's
    candidate genes for the head-to-head comparison.
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:e7fd1bf810f419b4fe01563a28afc5098e19a20ead7b8fb436e22ffbbe60c551
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmagraham/metPropagate
  tools:
  - Exomiser (hiPHIVE phenotype algorithm)
  - metPropagate
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# causative-gene-rank-comparison

## Summary

Compare per-patient candidate gene rankings of the causative gene produced by two or more prioritization scores (e.g. metPropagate propagated scores vs Exomiser hiPHIVE phenotype scores) by sorting each candidate list descending by each score and recording the causative gene's rank under each scheme.

## When to use

Use when patients have both a metPropagate post-propagation score file and a competing prioritization tool's score list (e.g. Exomiser hiPHIVE), plus a known causative gene, and you need a head-to-head ranking comparison.

## When NOT to use

- When the causative gene is unknown or unconfirmed — there is no ground-truth rank to compare against.
- When only one prioritization score is available per patient — a rank comparison needs at least two scoring schemes.
- When candidate gene lists use incompatible gene identifiers without a mapping, so a gene's rank cannot be traced across schemes.

## Inputs

- metPropagate LPA_output per-patient post-propagation node score file (Score.ID, Original.Score.ID per gene)
- Competing prioritization tool's per-patient candidate gene list with scores (e.g. sampleA_candidate_genes.csv with Exomiser hiPHIVE phenotype scores; gene scores from all inheritance patterns combined, highest-ranked variant per gene retained)
- Per-patient known causative gene assignment (e.g. Supplementary Table 1)

## Outputs

- Per-patient rank comparison table with causative gene's rank under each scoring scheme (e.g. metpropagate_vs_exomiser_rank_comparison.csv)
- Summary count of patients where each scheme ranked the causative gene more highly (e.g. 6/11 vs 5/11 patients and 7/13 genes)

## How to apply

For each patient, load the metPropagate LPA_output file to get propagated node scores ('Score.ID') and original enrichment scores ('Original.Score.ID'), and load the competing tool's candidate gene list (e.g. sampleA_candidate_genes.csv with hiPHIVE phenotype scores). Combine scores across all inheritance patterns for the competing tool, keeping only the highest-ranked variant per gene, then sort the candidate gene list in descending order by each score separately. Identify the patient's known causative gene (e.g. from Supplementary Table 1) and record its 1-based rank under each scoring scheme. Count patients where the metPropagate rank is better (numerically lower) than the competing rank, and write the per-patient comparison table (e.g. metpropagate_vs_exomiser_rank_comparison.csv). In the TIDEX cohort this procedure yielded metPropagate ranked the causative gene more highly than Exomiser's phenotype-based ranking in 6/11 patients.

## Related tools

- **metPropagate** (Produces the per-patient post-propagation candidate gene scores (Score.ID) that are ranked to obtain the causative gene's rank under the network-propagation scheme.) — https://github.com/emmagraham/metPropagate
- **Exomiser (hiPHIVE phenotype algorithm)** (Provides the competing phenotype-driven score used to rank each patient's candidate genes for the head-to-head comparison.)

## Examples

```
Rscript compare_ranks.R --lpa_output integration/LPA_output/sampleA --exomiser_genes wes_files/sampleA_candidate_genes.csv --causative_gene IDS
```

## Evaluation signals

- For each patient, the causative gene appears in both candidate gene lists and its rank is a positive integer within the list length (281–609 genes per TIDEX patient).
- Per-patient ranks under both schemes are recorded in the output table; missing causative genes are flagged rather than silently dropped.
- Summary comparison is reported both per-patient and per-gene (e.g. 6/11 patients, 7/13 genes), matching the TIDEX reported split of 6/11 vs 5/11 patients.
- Exomiser gene scores are combined across all inheritance patterns and only the highest-ranked variant per gene is retained before ranking.

## Limitations

- Exomiser candidate lists came from singleton exome analysis (281–609 genes per patient); in practice trio WES shortens the candidate list, so reported rankings may not reflect real-world trio workflows.
- Gene scores from all inheritance patterns were combined for Exomiser, which may merge heterogeneous signals.
- The comparison depends on feature-to-metabolite mapping availability — for the TIDEX metabolomic system this mapping was unavailable, limiting metPropagate's ME score derivation for some patients.

## Evidence

- [abstract] ranked the causative gene more highly than Exomiser's phenotype-based ranking in 6/11 patients: "ranked the causative gene more highly than Exomiser’s phenotype-based ranking in 6/11 patients"
- [readme] LPA_output contains the post-propagation node scores (automatically generated): "LPA_output contains the post-propagation node scores (automatically generated)"
- [results] Singleton VCFs for all patient were created by DeepVariant, which were then filtered by Exomiser to generate a list of candidate genes: "Singleton VCFs for all patient were created by DeepVariant, which were then filtered by Exomiser to generate a list of candidate genes"
- [discussion] exhibits comparable performance to existing phenotype-based prioritization tools such as Exomiser’s hiPHIVE phenotype algorithm: "exhibits comparable performance to existing phenotype-based prioritization tools such as Exomiser’s hiPHIVE phenotype algorithm"
- [discussion] candidate gene lists for a single patient included between 281 and 609 genes, meaning that the top 20th percentile included 55–115 genes: "candidate gene lists for a single patient included between 281 and 609 genes, meaning that the top 20th percentile included 55–115 genes"
