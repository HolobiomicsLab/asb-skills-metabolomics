---
name: cosine-score-spectral-library-matching
description: Use when raw or merged GCxGC/TOF-MS peak lists (e.g., 76 merged peaks
  per replicate run of a 35-compound mixture) must be matched against a reference
  metabolite spectral library to produce candidate identifications with posterior
  confidence scores.
when_to_use_negative:
- When the goal is biomarker discovery rather than identification — that requires
  MetPC's separate two-dimensional FDR (fdr2d) procedure
- When input data are already identified feature/metabolite tables with confirmed
  annotations rather than raw or merged peak spectra needing library matching
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetPC
  role: Fits the hierarchical statistical model with EM algorithm over the cosine-similarity
    candidate scores to produce posterior confidence measures for metabolite identification
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:21924d483ba160f4882a83de58926dc8406fd6222e8389e2157ea0979d36862c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - MetPC
  - jjs3098/CNU-Bioinformatics-Lab
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# cosine-score-spectral-library-matching

## Summary

Generates candidate metabolite matches for GCxGC/TOF-MS peaks by computing cosine similarity between each observed merged-peak spectrum and replicate reference-library spectra, then feeding those scores into MetPC's hierarchical-model EM identification to rank metabolites by posterior confidence.

## When to use

Use when raw or merged GCxGC/TOF-MS peak lists (e.g., 76 merged peaks per replicate run of a 35-compound mixture) must be matched against a reference metabolite spectral library to produce candidate identifications with posterior confidence scores.

## When NOT to use

- When the goal is biomarker discovery rather than identification — that requires MetPC's separate two-dimensional FDR (fdr2d) procedure
- When input data are already identified feature/metabolite tables with confirmed annotations rather than raw or merged peak spectra needing library matching

## Inputs

- Two replicate GCxGC/TOF-MS peak lists of a compound mixture after peak merging (e.g., 76 merged peaks per run for the 35-compound Dataset1)
- Reference metabolite spectral library with replicate spectra per metabolite

## Outputs

- Per-metabolite identification table with metabolite name and posterior confidence measure per replicate run (e.g., metabolite_identification_dataset1.csv)
- Ranked list of metabolites by posterior probability (top 20 reported as most likely present in the sample)

## How to apply

Load the replicate peak lists (e.g., Dataset1 of Experiment I: two replicate runs of the 35-compound mixture after peak merging, 76 merged peaks per run) from the deposited experiment files. For each merged peak spectrum, compute cosine similarity against the replicate spectra in the reference metabolite library to generate candidate matches. Use MetPC to fit the hierarchical statistical model with the Expectation-Maximization (EM) algorithm, which handles the latent peak-to-metabolite assignments using these similarity scores. Compute the posterior confidence measure for each candidate metabolite assignment in each replicate run, then rank metabolites by posterior probability (e.g., top 20 most likely to exist in the sample) and export the per-run table with metabolite name and posterior confidence columns (e.g., metabolite_identification_dataset1.csv).

## Related tools

- **MetPC** (Fits the hierarchical statistical model with EM algorithm over the cosine-similarity candidate scores to produce posterior confidence measures for metabolite identification) — https://github.com/jjs3098/CNU-Bioinformatics-Lab

## Evaluation signals

- The output CSV contains exactly the columns 'metabolite name' and 'posterior confidence measure' for each replicate run
- Top-20 metabolites ranked by posterior probability reproduce the identification results reported for Dataset1 in Results 3.1.1-3.1.2
- Every merged peak (76 per run for Dataset1) has been scored against the library and assigned at least a candidate match before EM fitting
- Ranking is by posterior probability, and the top-ranked metabolites correspond to compounds known to be in the 35-compound mixture

## Limitations

- Identification confidence depends on the quality and coverage of the reference metabolite library and its replicate spectra
- Latent peak-to-metabolite assignments are only estimated via EM, so posterior confidences are model-based rather than definitive identifications
- The deposited document lacks discussion-section text, so tuning guidance for score thresholds is not extractable from this source

## Evidence

- [abstract] Hierarchical model EM identification on GCxGC/TOF-MS merged peak lists: "For metabolite identification, we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm"
- [abstract] MetPC preprocessing and identification pipeline context: "we developed a new bioinformatics tool that includes a few of preprocessing steps and biomarker discovery procedure."
- [other] Cosine-similarity candidate generation workflow: [evidence span withheld — anchor status: not_found]
