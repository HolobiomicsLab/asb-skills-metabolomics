---
name: quantile-rank-evaluation
description: Use when scoring how well a gene-ranking method (e.g., metPropagate,
  Exomiser, or a combined score) recovers a known causative gene, by testing whether
  it lands in the top 20th percentile of a patient's candidate gene list and comparing
  per-patient counts across methods.
when_to_use_negative:
- No known causative/diagnosed gene is available for the patient, so no ground-truth
  quantile hit can be assessed
- Candidate lists are not ranked gene sets from a prioritization pipeline (e.g., raw
  metabolite intensity matrices or unfiltered variant calls)
- You need absolute statistical significance rather than percentile-based per-patient
  hit counting; use the 1000-permutation random-list design instead
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: metPropagate
  role: Produces the propagated per-gene scores (LPA output) whose ranked candidate
    lists are evaluated by the top-20th-percentile criterion
  repo: github.com/emmagraham/metPropagate
- name: Exomiser
  role: Generates the candidate gene lists via variant effect and population frequency
    filtering and provides the phenotype-based ranking used as the comparison baseline
- name: STRING
  role: Protein functional linkage network whose 19,354 genes define the ranked population
    over which percentile bins are computed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:1a6080bb802aa806077a6edd02ea690fb522329935dcfd17f770f7ccf654a5d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Exomiser
  - STRING
  - metPropagate
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# quantile-rank-evaluation

## Summary

Evaluates gene-prioritization performance by checking whether a known causative gene falls within the top 20th percentile of each patient's ranked candidate gene list (e.g., rank <= 60 for 300-gene lists), and compares methods by counting patients hitting that quantile (e.g., 9/11 or 10/11 TIDEX patients).

## When to use

Use when scoring how well a gene-ranking method (e.g., metPropagate, Exomiser, or a combined score) recovers a known causative gene, by testing whether it lands in the top 20th percentile of a patient's candidate gene list and comparing per-patient counts across methods.

## When NOT to use

- No known causative/diagnosed gene is available for the patient, so no ground-truth quantile hit can be assessed
- Candidate lists are not ranked gene sets from a prioritization pipeline (e.g., raw metabolite intensity matrices or unfiltered variant calls)
- You need absolute statistical significance rather than percentile-based per-patient hit counting; use the 1000-permutation random-list design instead

## Inputs

- Per-patient ranked candidate gene lists with prioritization scores (metPropagate LPA propagated scores, e.g., LPA_output/sampleA; Exomiser-Phenotype scores; or combined scores from combined_rankings.csv)
- Known causative gene labels per patient (e.g., TIDEX genetically diagnosed genes, OMIM/IEMbase associations)
- Candidate gene lists from Exomiser's variant effect and population frequency filters (281-609 genes per TIDEX patient)

## Outputs

- Per-patient quantile evaluation: boolean top-20th-percentile hit and causative gene rank
- Per-patient summary counts of patients with a causative gene in the top 20th percentile (e.g., 9/11, 10/11) and patients whose rank improved versus another method
- Per-gene scores and ranks table, e.g., combined_rankings.csv

## How to apply

For each patient, rank the candidate genes (typically 281–609 genes after Exomiser variant filtering) by the prioritization score being evaluated, and compute the top-20th-percentile cutoff as 20% of the list length — 55–115 genes for TIDEX-sized lists, or rank <= 60 for a 300-gene list. Declare success for a patient if at least one known causative gene falls within that percentile. Aggregate over the cohort (e.g., 11 TIDEX patients) and report the fraction of patients with a causative gene in the top 20th percentile, then repeat identically for each competing method (metPropagate propagated/LPA scores, Exomiser-Phenotype scores, or the combined weighted score) to count patients whose causative gene rank improved. For null calibration, generate permutations — e.g., 1000 random 300-gene candidate lists that include the causative gene — to assess how often the quantile hit occurs by chance. Write per-gene ranks and per-patient summary counts (e.g., combined_rankings.csv) so the quantile outcomes are auditable.

## Related tools

- **metPropagate** (Produces the propagated per-gene scores (LPA output) whose ranked candidate lists are evaluated by the top-20th-percentile criterion) — github.com/emmagraham/metPropagate
- **Exomiser** (Generates the candidate gene lists via variant effect and population frequency filtering and provides the phenotype-based ranking used as the comparison baseline)
- **STRING** (Protein functional linkage network whose 19,354 genes define the ranked population over which percentile bins are computed)

## Evaluation signals

- Causative gene falls within rank <= 20% of the candidate list length (rank <= 60 for a 300-gene list; 55-115 genes for 281-609 gene TIDEX lists)
- Per-patient hit counts match published baselines: metPropagate 9/11 TIDEX patients; combined metPropagate + Exomiser score 10/11 patients with 8/11 improved ranks
- Exomiser-alone and metPropagate-alone baselines recomputed identically so per-patient rank comparisons are apples-to-apples
- Combined metPropagate + Exomiser ranking is higher than either algorithm alone for 8/11 patients
- Null expectation calibrated via 1000 permutations of random 300-gene candidate lists that include the causative gene

## Limitations

- Candidate gene lists vary from 281 to 609 genes per patient, so the absolute rank cutoff corresponding to the top 20th percentile differs across patients (55-115 genes)
- Analyses used singleton exomes rather than trio WES; trio parental data would shorten candidate lists and change percentile cutoffs but is not reflected in reported results
- Reported prioritization rate for known IEMs differs slightly between abstract variants (91% vs 92%), so exact reproduction of that aggregate figure is version-dependent

## Evidence

- [discussion] candidate gene lists for a single patient included between 281 and 609 genes, meaning that the top 20th percentile included 55–115 genes: "candidate gene lists for a single patient included between 281 and 609 genes, meaning that the top 20th percentile included 55–115 genes"
- [abstract] metPropagate placed at least one causative gene in the top 20th percentile in 9/11 patients: "metPropagate placed at least one causative gene in the top 20th percentile in 9/11 patients"
- [results] 1000 permutations of random candidate gene lists that included the causative gene were generated: "1000 permutations of random candidate gene lists that included the causative gene were generated"
- [other] evaluate whether at least one known causative gene falls in the top 20th percentile (rank <= 60 for 300-gene lists): [evidence span withheld — anchor status: not_found]
- [intro] Each patient’s set of candidate genes is ranked by the resulting propagated score, resulting in a prioritized list of candidate genes: "Each patient’s set of candidate genes is ranked by the resulting propagated score, resulting in a prioritized list of candidate genes"
