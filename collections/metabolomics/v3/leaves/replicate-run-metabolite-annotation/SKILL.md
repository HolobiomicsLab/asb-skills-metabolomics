---
name: replicate-run-metabolite-annotation
description: Use when you have merged peak lists from replicate GCxGC/TOF-MS runs
  (e.g. two runs of a 35-compound mixture) and need metabolite identification with
  a posterior confidence measure per metabolite, ranked by posterior probability.
when_to_use_negative:
- The goal is biomarker discovery rather than identification — MetPC's biomarker step
  controls the two-dimensional false discovery rate (fdr2d) in multiple hypothesis
  testing and is a separate stage of the pipeline
- Raw (unmerged) peak data is provided instead of merged peak lists; the model assigns
  peaks from merged peak lists, so preprocessing/peak merging must be done first
- No replicate spectra are available in the reference library to compute the cosine
  similarity candidate matches required by the model
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetPC
  role: Bioinformatics pipeline implementing preprocessing, metabolite identification
    via the hierarchical model with EM, and biomarker discovery under fdr2d control
  repo: jjs3098/CNU-Bioinformatics-Lab
- name: Expectation-Maximization (EM) algorithm
  role: Fits the hierarchical statistical model by handling the latent peak-to-metabolite
    assignment variables during parameter estimation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:3584783c7fe1851fca0e4b19aaadd32a3d2d68c90dc4c3e216060d9d906f9476
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Replicate-run metabolite annotation

## Summary

Annotates metabolites in GCxGC/TOF-MS replicate runs by scoring merged-peak spectra against a reference metabolite library (cosine similarity) and assigning peaks to metabolites with a hierarchical statistical model fitted by an Expectation-Maximization (EM) algorithm, yielding per-metabolite posterior confidence measures.

## When to use

Use when you have merged peak lists from replicate GCxGC/TOF-MS runs (e.g. two runs of a 35-compound mixture) and need metabolite identification with a posterior confidence measure per metabolite, ranked by posterior probability.

## When NOT to use

- The goal is biomarker discovery rather than identification — MetPC's biomarker step controls the two-dimensional false discovery rate (fdr2d) in multiple hypothesis testing and is a separate stage of the pipeline
- Raw (unmerged) peak data is provided instead of merged peak lists; the model assigns peaks from merged peak lists, so preprocessing/peak merging must be done first
- No replicate spectra are available in the reference library to compute the cosine similarity candidate matches required by the model

## Inputs

- Replicate GCxGC/TOF-MS merged peak lists with peak spectra (e.g. 76 merged peaks per run for the 35-compound mixture, Dataset1 of Experiment I)
- Reference metabolite library containing replicate peak spectra for candidate metabolites

## Outputs

- Per-run metabolite identification table with columns for metabolite name and posterior confidence measure (e.g. metabolite_identification_dataset1.csv)
- Ranked list of metabolites ordered by posterior probability (e.g. top-20 most likely metabolites in the sample)

## How to apply

Load the replicate peak lists (e.g. two runs of the 35-compound mixture with 76 merged peaks each) from the deposited experiment files. For each merged peak spectrum, compute cosine similarity scores against replicate spectra in the reference metabolite library to generate candidate matches. Fit the hierarchical statistical model with the Expectation-Maximization (EM) algorithm, which handles the latent assignment of peaks to metabolites, and compute the posterior confidence measure for each candidate metabolite assignment in each replicate run. Rank metabolites by posterior probability and assemble the per-run identification table with metabolite name and posterior confidence measure columns (e.g. metabolite_identification_dataset1.csv). The EM formulation is used precisely because peak-to-metabolite assignments are latent variables that cannot be observed directly.

## Related tools

- **MetPC** (Bioinformatics pipeline implementing preprocessing, metabolite identification via the hierarchical model with EM, and biomarker discovery under fdr2d control) — jjs3098/CNU-Bioinformatics-Lab
- **Expectation-Maximization (EM) algorithm** (Fits the hierarchical statistical model by handling the latent peak-to-metabolite assignment variables during parameter estimation)

## Evaluation signals

- The output table contains exactly two columns: metabolite name and posterior confidence measure, one row per candidate metabolite per replicate run
- The top-20 metabolites ranked by posterior probability reproduce the identification results reported for Dataset1 in Results sections 3.1.1–3.1.2
- Each merged peak (76 per run for Dataset1) has candidate matches generated via cosine similarity against the reference library spectra
- The EM algorithm converges and yields well-defined posterior probabilities for every metabolite in both replicate runs, allowing consistent ranking across runs

## Limitations

- Identification confidence depends on the quality and coverage of the reference metabolite library replicate spectra used for cosine similarity scoring
- The method was demonstrated on a controlled 35-compound mixture (Experiment I, Dataset1); behavior on complex biological mixtures with many more latent assignments is not established by this reproduction task
- No discussion section text is available for this article, so additional caveats (e.g. misassignment rates, EM initialization sensitivity) cannot be verified from the provided context

## Evidence

- [abstract] Hierarchical model with EM: "For metabolite identification, we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm"
- [abstract] Latent variable handling: "we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm to take care of latent variables."
- [abstract] Pipeline tooling: "we developed a new bioinformatics tool that includes a few of preprocessing steps and biomarker discovery procedure."
- [other] Repository: "jjs3098/CNU-Bioinformatics-Lab"
