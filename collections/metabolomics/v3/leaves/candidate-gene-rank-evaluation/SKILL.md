---
name: candidate-gene-rank-evaluation
description: Use when you have per-patient prioritized candidate gene lists (281-609
  genes from Exomiser filtering) plus each patient's previously diagnosed causative
  gene, and need head-to-head rank comparison of scoring schemes.
when_to_use_negative:
- Causative gene is unknown for the patient cohort — rank comparison requires a previously
  diagnosed causative gene per patient.
- Candidate gene lists were not produced through Exomiser's variant filtering pipeline
  — the comparison assumes a common candidate universe per patient.
- Your metabolomic system cannot measure the disease's diagnostic metabolites (e.g.,
  large glycosaminoglycans for the IDS patient), since metPropagate scores will be
  insensitive for those samples.
tools:
- name: metPropagate
  role: Produces the propagated candidate-gene scores (Score.ID) used for one ranking
    scheme; adapted from yamaguchiyuto/label_propagation
  repo: https://github.com/emmagraham/metPropagate
- name: Exomiser (hiPHIVE phenotype algorithm)
  role: Provides the comparator phenotype-driven candidate gene lists and phenotype
    scores ranked per patient
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:357b0ddc73d01d174dfba915e2e8772ba7d22df170899e5a292aecc39526efb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmagraham/metPropagate
  tools:
  - Exomiser (hiPHIVE phenotype algorithm)
  - metPropagate
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# candidate-gene-rank-evaluation

## Summary

Per-patient benchmarking of causative-gene ranks across prioritization schemes — specifically comparing the rank of each patient's known causative gene under metPropagate's post-propagation Score.ID ordering against Exomiser's hiPHIVE phenotype-driven ranking.

## When to use

Use when you have per-patient prioritized candidate gene lists (281-609 genes from Exomiser filtering) plus each patient's previously diagnosed causative gene, and need head-to-head rank comparison of scoring schemes.

## When NOT to use

- Causative gene is unknown for the patient cohort — rank comparison requires a previously diagnosed causative gene per patient.
- Candidate gene lists were not produced through Exomiser's variant filtering pipeline — the comparison assumes a common candidate universe per patient.
- Your metabolomic system cannot measure the disease's diagnostic metabolites (e.g., large glycosaminoglycans for the IDS patient), since metPropagate scores will be insensitive for those samples.

## Inputs

- metPropagate LPA_output post-propagation score files per sample (Score.ID, Original.Score.ID columns)
- Per-patient Exomiser candidate gene lists with hi-PHIVE phenotype scores (e.g., sampleA_candidate_genes.csv)
- Known causative gene per patient from Supplementary Table 1 (e.g., 11 TIDEX neurometabolic patients)

## Outputs

- Per-patient comparison table (11 rows) recording causative-gene rank under metPropagate and under Exomiser-Phenotype (metpropagate_vs_exomiser_rank_comparison.csv)
- Win-count summary (e.g., metPropagate higher in 6/11 patients, Exomiser-Phenotype higher in 5/11)

## How to apply

For each of the 11 patients, load the post-propagation node scores from the LPA_output files (columns 'Score.ID' and 'Original.Score.ID') and sort the candidate gene list in descending order by the metPropagate propagated score; record the rank of the causative gene identified from Supplementary Table 1. Independently rank the same patient's candidate genes by Exomiser's hiPHIVE phenotype score, noting that Exomiser gene scores from all inheritance patterns were combined and only the highest-ranked variant per gene was retained, and record that rank. Compare the two ranks per patient, flagging patients where metPropagate ranked the causative gene more highly, and write the comparison to an 11-row table such as metpropagate_vs_exomiser_rank_comparison.csv. Tally the win counts per scheme and, optionally, evaluate the weighted combination of metPropagate and Exomiser scores, which the article reports improved prioritization. Candidate lists come from Exomiser's variant effect and population frequency filters applied to DeepVariant singleton VCFs, yielding 281-609 genes per patient (top 20th percentile = 55-115 genes).

## Related tools

- **metPropagate** (Produces the propagated candidate-gene scores (Score.ID) used for one ranking scheme; adapted from yamaguchiyuto/label_propagation) — https://github.com/emmagraham/metPropagate
- **Exomiser (hiPHIVE phenotype algorithm)** (Provides the comparator phenotype-driven candidate gene lists and phenotype scores ranked per patient)

## Examples

```
import pandas as pd; df = pd.read_csv('integration/LPA_output/sampleA'); rank = df.sort_values('Score.ID', ascending=False)['ID'].tolist().index(causative_gene) + 1
```

## Evaluation signals

- The written comparison table has exactly one row per patient (11 for TIDEX) with both ranks populated from the correct candidate gene universe.
- Win-count tally matches the published finding: metPropagate ranked the causative gene higher in 6/11 patients; Exomiser-Phenotype in 5/11 patients (and in 7/13 genes).
- Each patient's candidate list size falls in the 281-609 range reported for Exomiser's variant effect and population frequency filters; top 20th percentile = 55-115 genes.
- Ranks computed on Score.ID ordering are internally consistent — re-sorting the LPA_output file by Score.ID descending reproduces the recorded causative-gene rank.
- Exomiser ranks reflect the preprocessing convention: scores from all inheritance modes combined and one (highest-ranked) variant per gene retained.

## Limitations

- Singleton exomes were analyzed rather than trio WES; trio parental data would shorten candidate lists and is not reflected in these results.
- No Exomiser/hiPHIVE or STRING version is reported, limiting exact reproducibility of the comparator ranking.
- Feature-to-metabolite mapping was unavailable for the TIDEX metabolomic system, and biomarker metabolites such as acyl glycines or acyl carnitines are not annotated to IEM-associated genes in HMDB, weakening metPropagate scores for some patients.

## Evidence

- [abstract] ranked the causative gene more highly than Exomiser’s phenotype-based ranking in 6/11 patients: "ranked the causative gene more highly than Exomiser’s phenotype-based ranking in 6/11 patients"
- [readme] File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as "Original.Score.ID".: "File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as "Original.Score.ID"."
- [results] between 281 and 609 total genes emerged from Exomiser’s variant effect and population frequency filters: "between 281 and 609 total genes emerged from Exomiser’s variant effect and population frequency filters"
- [readme] each patient's Exomiser candidate gene list with hi-PHIVE phenotype scores (e.g., sampleA_candidate_genes.csv): "sampleA_candidate_genes.csv is an example input file from Exomiser"
- [abstract] ranking by a weighted combination of metPropagate and Exomiser scores resulted in improved prioritization: "ranking by a weighted combination of metPropagate and Exomiser scores resulted in improved prioritization"
