---
name: posterior-probability-confidence-ranking
description: Use when merged GCxGC/TOF-MS peak lists (e.g. 76 merged peaks per replicate
  run) must be assigned to reference-library metabolites and ranked by posterior confidence
  to produce an identification table.
when_to_use_negative:
- The goal is biomarker discovery with fdr2d control rather than metabolite identification
  — MetPC treats these as separate pipeline stages
- Input data has not yet been preprocessed and peaks have not been merged across the
  replicate runs
- No reference metabolite library is available to provide candidate spectra for matching
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetPC
  role: Pipeline tool implementing metabolite identification via the hierarchical
    statistical model with EM algorithm whose posterior probabilities are ranked
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
- name: Expectation-Maximization (EM) algorithm
  role: Fits the hierarchical statistical model, handling latent variables to estimate
    peak-to-metabolite assignment for posterior confidence computation
- name: jjs3098/CNU-Bioinformatics-Lab
  role: Deposited code repository used to run the identification workflow on Experiment
    I Dataset1
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:e8234fc456ad40736b70cf3fa62f8a03e9a735064371ee599749bf71b13a2fb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - Expectation-Maximization (EM) algorithm
  - MetPC
  - jjs3098/CNU-Bioinformatics-Lab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# posterior-probability-confidence-ranking

## Summary

Ranks candidate metabolite assignments by the posterior confidence measure computed from a hierarchical statistical model fitted with the Expectation-Maximization (EM) algorithm, so that the metabolites most likely to exist in a sample (e.g. the top-20 of a 35-compound GCxGC/TOF-MS mixture) can be reported with per-metabolite confidence values.

## When to use

Use when merged GCxGC/TOF-MS peak lists (e.g. 76 merged peaks per replicate run) must be assigned to reference-library metabolites and ranked by posterior confidence to produce an identification table.

## When NOT to use

- The goal is biomarker discovery with fdr2d control rather than metabolite identification — MetPC treats these as separate pipeline stages
- Input data has not yet been preprocessed and peaks have not been merged across the replicate runs
- No reference metabolite library is available to provide candidate spectra for matching

## Inputs

- Merged peak lists from replicate GCxGC/TOF-MS runs (e.g. two replicate runs with 76 merged peaks each)
- Reference metabolite library with replicate spectra per metabolite

## Outputs

- metabolite_identification_dataset1.csv: table of metabolite name and posterior confidence measure per replicate run
- Ranked list of metabolites (e.g. top 20) most likely to exist in the sample, ordered by posterior probability

## How to apply

Load the replicate peak lists (e.g. Dataset1, two replicate runs of a 35-compound mixture with 76 merged peaks per run) and compute cosine similarity scores between each merged peak spectrum and the replicate spectra in the reference metabolite library to generate candidate matches. Fit the hierarchical statistical model with the EM algorithm to handle the latent assignment of peaks to metabolites. Compute the posterior confidence measure for each candidate metabolite assignment in each replicate run. Rank metabolites by posterior probability and assemble a per-run results table with columns for metabolite name and posterior confidence measure, reporting the top-ranked metabolites (e.g. top 20) as those most likely to exist in the sample. The rationale is that the EM algorithm accounts for latent variables in peak-to-metabolite assignment rather than relying on cosine similarity alone.

## Related tools

- **MetPC** (Pipeline tool implementing metabolite identification via the hierarchical statistical model with EM algorithm whose posterior probabilities are ranked) — https://github.com/jjs3098/CNU-Bioinformatics-Lab
- **Expectation-Maximization (EM) algorithm** (Fits the hierarchical statistical model, handling latent variables to estimate peak-to-metabolite assignment for posterior confidence computation)
- **jjs3098/CNU-Bioinformatics-Lab** (Deposited code repository used to run the identification workflow on Experiment I Dataset1) — https://github.com/jjs3098/CNU-Bioinformatics-Lab

## Evaluation signals

- The output table contains exactly the columns metabolite name and posterior confidence measure for each replicate run
- Top-20 metabolites ranked by posterior probability reproduce the identification results reported for Dataset1 in Results 3.1.1-3.1.2
- Each merged peak (76 per replicate run) receives candidate matches via cosine similarity against library replicate spectra before EM fitting
- Posterior confidence measures are produced per candidate metabolite assignment in each of the two replicate runs

## Limitations

- Confidence ranking depends on prior peak merging; results were validated on a 35-compound mixture with 76 merged peaks per run, so behavior on much more complex samples is not demonstrated by this source
- The article's discussion section text was unavailable, so caveats on posterior calibration or library coverage could not be extracted

## Evidence

- [abstract] hierarchical model EM identification: "we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm to take care of latent variables."
- [other] EM identification workflow: [evidence span withheld — anchor status: not_found]
- [other] ranked output table: [evidence span withheld — anchor status: not_found]
