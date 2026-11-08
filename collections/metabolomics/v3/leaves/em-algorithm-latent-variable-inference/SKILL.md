---
name: em-algorithm-latent-variable-inference
description: Use when merged GCxGC/TOF-MS peak lists must be assigned to reference
  library metabolites, and each observed spectrum may match multiple library entries
  so a latent-variable model with posterior confidence is needed.
when_to_use_negative:
- The input is already a feature/biomarker table — the EM identification step operates
  on raw-to-merged peak spectra, not downstream feature tables (use the fdr2d biomarker
  discovery procedure instead)
- The data are LC-MS or other single-dimension profiles where peak merging and two-dimensional
  cosine-spectrum matching against a GCxGC/TOF-MS reference library do not apply
tools:
- name: MetPC
  role: Bioinformatics tool implementing preprocessing, the hierarchical-model/EM
    metabolite identification, and fdr2d biomarker discovery for this workflow
  repo: jjs3098/CNU-Bioinformatics-Lab
- name: Expectation-Maximization (EM) algorithm
  role: Fits the hierarchical statistical model by alternating E-steps over latent
    peak-to-metabolite assignments and M-steps over model parameters, producing posterior
    confidence measures
  repo: jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:444375d4be5f8347e289a308c1fcabfb549a792bd4713a4180eed0eafdcb819b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - Expectation-Maximization (EM) algorithm
  - MetPC
  - jjs3098/CNU-Bioinformatics-Lab
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# EM-algorithm latent-variable inference

## Summary

Fits a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm to handle latent peak-to-metabolite assignments in GCxGC/TOF-MS metabolite identification, yielding posterior confidence measures per candidate metabolite.

## When to use

Use when merged GCxGC/TOF-MS peak lists must be assigned to reference library metabolites, and each observed spectrum may match multiple library entries so a latent-variable model with posterior confidence is needed.

## When NOT to use

- The input is already a feature/biomarker table — the EM identification step operates on raw-to-merged peak spectra, not downstream feature tables (use the fdr2d biomarker discovery procedure instead)
- The data are LC-MS or other single-dimension profiles where peak merging and two-dimensional cosine-spectrum matching against a GCxGC/TOF-MS reference library do not apply

## Inputs

- Two replicate peak lists of a GCxGC/TOF-MS run (e.g. Dataset1: 35-compound mixture, 76 merged peaks per run) from the deposited experiment files
- Reference metabolite library with replicate spectra for each library metabolite

## Outputs

- Per-run metabolite identification table (metabolite_identification_dataset1.csv) with columns for metabolite name and posterior confidence measure
- Ranked list of metabolites by posterior probability (top-20 reported as most likely to exist in the sample)

## How to apply

Start from merged peak lists (e.g. 76 merged peaks per replicate run of a 35-compound mixture in Dataset1 of Experiment I). Compute cosine similarity scores between each merged peak spectrum and the replicate spectra in the reference metabolite library to generate candidate matches. Then fit the hierarchical statistical model with the EM algorithm, which treats the true peak-to-metabolite assignment as a latent variable and iteratively estimates model parameters and assignment posteriors. From the fitted model, compute a posterior confidence measure for each candidate metabolite assignment in each replicate run. Rank metabolites by posterior probability (the top-20 are reported as most likely present) and assemble a per-run table with metabolite name and posterior confidence measure columns.

## Related tools

- **MetPC** (Bioinformatics tool implementing preprocessing, the hierarchical-model/EM metabolite identification, and fdr2d biomarker discovery for this workflow) — jjs3098/CNU-Bioinformatics-Lab
- **Expectation-Maximization (EM) algorithm** (Fits the hierarchical statistical model by alternating E-steps over latent peak-to-metabolite assignments and M-steps over model parameters, producing posterior confidence measures) — jjs3098/CNU-Bioinformatics-Lab

## Evaluation signals

- Output table contains exactly two kinds of columns per metabolite: name and posterior confidence measure
- The top-20 metabolites ranked by posterior probability reproduce the identification results reported for Dataset1 in Results 3.1.1–3.1.2
- The number of candidate matches per peak reflects cosine similarity scoring of merged-peak spectra against library replicate spectra (e.g. all 35 mixture compounds recoverable from 76 merged peaks)
- Posterior confidence measures are produced separately for each of the two replicate runs, and concordance across replicates is consistent

## Limitations

- Method was developed and demonstrated for GCxGC/TOF-MS metabolomics data of the deposited Experiment I mixtures; transfers to other instrument types or libraries are unvalidated in this context
- The provided article text gives only abstract-level detail (no discussion-section text available, no changelog), so exact parameter values and convergence criteria for the EM fit are not documented here
- Confidence is a posterior probability, not an identity proof — low-similarity or ambiguous library entries may still yield non-zero posteriors

## Evidence

- [abstract] Hierarchical model with EM over latent assignments: "we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm to take care of latent variables."
- [abstract] Identification step precedes biomarker discovery in MetPC: "we developed a new bioinformatics tool that includes a few of preprocessing steps and biomarker discovery procedure."
- [other] Repository hosting the pipeline: "jjs3098/CNU-Bioinformatics-Lab"
