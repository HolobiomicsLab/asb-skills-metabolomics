---
name: top-percentile-prioritization-thresholding
description: Use when evaluating a gene prioritization method against known causative
  genes on ranked candidate lists of varying length (e.g., 281-609 genes/patient),
  where a percentile-based success cutoff is fairer than absolute rank thresholds.
when_to_use_negative:
- When candidate lists have uniform, fixed length and an absolute rank cutoff (e.g.,
  top 50 genes) is meaningful and simpler
- When no known causative gene/diagnosis is available for the patient, since the percentile
  criterion cannot be scored
- When the goal is ranked-list evaluation without a reference gene (e.g., purely exploratory
  prioritization) - percentile-of-causative-gene scoring does not apply
edam_operation: http://edamontology.org/operation_1811
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: metPropagate
  role: Produces the propagated per-gene scores ('Score.ID') used to rank candidate
    genes whose top-percentile position is evaluated
  repo: github.com/emmagraham/metPropagate
- name: Exomiser
  role: Supplies the variant-filtered candidate gene lists and the comparator hiPHIVE
    phenotype-driven ranking (Exomiser-Phenotype score)
- name: DeepVariant
  role: Generates the singleton VCFs that Exomiser filters to produce the candidate
    gene lists being ranked
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:48f99ff52afa6145afa0a35f03a5615b4b7a9f89cbab35458c03ccae923c5e24
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepVariant
  - Exomiser
  - metPropagate
  edam_operation: http://edamontology.org/operation_1811
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# top-percentile-prioritization-thresholding

## Summary

Judges gene prioritization success by whether at least one causative gene falls in the top 20th percentile of a patient's ranked candidate list, rather than requiring an absolute rank. Applied to metPropagate and Exomiser rankings over Exomiser-filtered candidate gene lists (281-609 genes per TIDEX patient).

## When to use

Use when evaluating a gene prioritization method against known causative genes on ranked candidate lists of varying length (e.g., 281-609 genes/patient), where a percentile-based success cutoff is fairer than absolute rank thresholds.

## When NOT to use

- When candidate lists have uniform, fixed length and an absolute rank cutoff (e.g., top 50 genes) is meaningful and simpler
- When no known causative gene/diagnosis is available for the patient, since the percentile criterion cannot be scored
- When the goal is ranked-list evaluation without a reference gene (e.g., purely exploratory prioritization) - percentile-of-causative-gene scoring does not apply

## Inputs

- Per-patient ranked candidate gene list with propagated scores (metPropagate LPA_output 'Score.ID' and 'Original.Score.ID')
- Exomiser candidate gene list with hi-PHIVE phenotype scores (e.g., sampleA_candidate_genes.csv), highest-ranked variant per gene retained
- Per-patient known causative gene assignments (e.g., Supplementary Table 1 for the 11 TIDEX patients)

## Outputs

- Per-patient success flags indicating whether the causative gene is in the top 20th percentile
- Per-patient rank comparison table (e.g., metpropagate_vs_exomiser_rank_comparison.csv, 11 rows)
- Aggregate prioritization rate (e.g., 92% of 107 Miller patients; 9/11 TIDEX patients)

## How to apply

For each patient, take the ranked candidate gene list produced by the prioritization method (e.g., metPropagate's post-propagation 'Score.ID' from LPA_output, or Exomiser hiPHIVE phenotype scores) and compute the rank of the known causative gene (identified from Supplementary Table 1 or equivalent diagnoses). Declare success if at least one causative gene falls within the top 20th percentile of the list. Because candidate lists from Exomiser's variant effect and population frequency filters ranged from 281 to 609 genes, the top 20th percentile corresponds to roughly 55-115 genes; use list-specific percentile math rather than a fixed rank cutoff. For method comparison, additionally record per-patient ranks under each scheme (metPropagate vs Exomiser-Phenotype), count patients where one method ranks the causative gene more highly, and optionally test a weighted combination of both scores, which improved prioritization. Validate by checking the aggregate fraction against reported benchmarks (92% of Miller IEM patients; 9/11 TIDEX patients; 6/11 metPropagate-beats-Exomiser comparisons).

## Related tools

- **metPropagate** (Produces the propagated per-gene scores ('Score.ID') used to rank candidate genes whose top-percentile position is evaluated) — github.com/emmagraham/metPropagate
- **Exomiser** (Supplies the variant-filtered candidate gene lists and the comparator hiPHIVE phenotype-driven ranking (Exomiser-Phenotype score))
- **DeepVariant** (Generates the singleton VCFs that Exomiser filters to produce the candidate gene lists being ranked)

## Examples

```
Rscript rank_compare.R --lpa_output LPA_output/sampleA --candidates wes_files/sampleA_candidate_genes.csv --causative_gene GENE --top_pct 0.20 --out metpropagate_vs_exomiser_rank_comparison.csv
```

## Evaluation signals

- Per patient, verify rank(causative gene) <= ceil(0.2 * list length); for 281-609 gene TIDEX lists this means rank within roughly 55-115 genes
- Aggregate prioritization rate should reproduce reported values: at least one causative gene in top 20th percentile for 92% of Miller patients and 9/11 TIDEX patients
- Head-to-head comparison table should show metPropagate ranking the causative gene more highly than Exomiser-Phenotype in 6/11 patients (Exomiser higher in 5/11, with 7/13 gene-level comparisons)
- Check the comparison output has one row per patient (11 rows for TIDEX) with both ranking schemes recorded
- Confirm candidate list sizes fall in the 281-609 range before applying the 20th-percentile cutoff

## Limitations

- TIDEX candidate lists used singleton exomes; trio WES would shorten candidate lists and change the percentile window, so results may not reflect standard trio-based pipelines
- Success is 'at least one causative gene' in the top 20th percentile, which tolerates multiple false positives and does not require the top rank
- For the TIDEX comparison, the metabolomic system could not measure large glycosaminoglycans and feature-to-metabolite mapping was unavailable, which can depress metPropagate ranks for affected patients (e.g., the IDS patient)
- Metabolites commonly used as IEM biomarkers (acyl glycines, acyl carnitines) are not annotated to IEM-associated genes in HMDB, limiting score quality for some genes

## Evidence

- [abstract] top 20th percentile thresholding on metPropagate output: "metPropagate was able to prioritize at least one causative gene in the top 20th percentile of candidate genes for 92% of patients with known IEMs."
- [discussion] TIDEX candidate list size and percentile window: [evidence span withheld — anchor status: not_found]
- [abstract] head-to-head rank comparison benchmark: [evidence span withheld — anchor status: not_found]
- [readme] score columns used for ranking: "Results of label propagation will populate in *integration/results/*. File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as"
- [results] origin of candidate gene lists: "Singleton VCFs for all patient were created by DeepVariant, which were then filtered by Exomiser to generate a list of candidate genes"
