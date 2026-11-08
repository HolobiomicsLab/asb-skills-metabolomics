---
name: graph-neural-network-inference
description: Use when you need to score candidate enzyme–substrate reactions with
  a trained GNN-SOM model and check whether scores differentiate true (Known) from
  random pairs via a two-sided Mann-Whitney U test.
when_to_use_negative:
- The input is unannotated mass features without predicted metabolite structures —
  the GNN-SOM scorer requires molecular structures of substrates, not raw m/z features.
- The goal is correlation-based pathway discovery from paired transcriptomics/metabolomics
  data (mutual-rank correlation, ClusterONE feature clusters) — use the MEANtools
  MR/ClusterONE pipeline instead of GNN scoring.
- The task requires multi-substrate reaction prediction — reaction-rule based prediction
  in this context is designed for single-substrate reactions only.
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: MEANtools
  role: Framework providing loadGnnSomState and createGnnSom for restoring the pretrained
    GNN-SOM state and instantiating the reaction-likelihood scorer
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: createGnnSom
  role: Instantiates the GNN-SOM scorer used to compute reaction-likelihood scores
    for each enzyme–substrate pair
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: loadGnnSomState
  role: Restores the pretrained GNN-SOM model state prior to scoring
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: ReactionDecoder
  role: Supplies reaction-center predictions that are combined with GNN-SOM site-of-metabolism
    likelihoods into the final pair score
- name: scipy.stats.mannwhitneyu
  role: Two-sided Mann-Whitney U test comparing Known vs Random score distributions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:754377964632e2ca319397fc8daa7d2dd7931f9b21f301896121630976e2e13b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MEANtools
  - ReactionDecoder
  - createGnnSom
  - loadGnnSomState
  - scipy.stats.mannwhitneyu
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# graph-neural-network-inference

## Summary

Applies a pretrained GNN-SOM graph neural network, combined with ReactionDecoder reaction-center prediction, to compute reaction-likelihood scores for enzyme–substrate pairs, and validates the scoring by statistically separating experimentally characterized (Known) pairs from randomly assembled (Random) pairs.

## When to use

Use when you need to score candidate enzyme–substrate reactions with a trained GNN-SOM model and check whether scores differentiate true (Known) from random pairs via a two-sided Mann-Whitney U test.

## When NOT to use

- The input is unannotated mass features without predicted metabolite structures — the GNN-SOM scorer requires molecular structures of substrates, not raw m/z features.
- The goal is correlation-based pathway discovery from paired transcriptomics/metabolomics data (mutual-rank correlation, ClusterONE feature clusters) — use the MEANtools MR/ClusterONE pipeline instead of GNN scoring.
- The task requires multi-substrate reaction prediction — reaction-rule based prediction in this context is designed for single-substrate reactions only.

## Inputs

- Known enzyme–substrate pair file (experimentally characterized pairs) from zenodo.org/records/15697913
- Random enzyme–substrate pair file (randomly assembled pairs) from zenodo.org/records/15697913
- Pretrained GNN-SOM model state (restored via loadGnnSomState)
- ReactionDecoder reaction-center predictions for each enzyme–substrate pair

## Outputs

- gnnsom_scores.csv — per-pair score table with columns: set label, enzyme, substrate, score
- mannwhitney_result.json — Mann-Whitney U statistic, p-value, and median/mean summary statistics for Known and Random score distributions

## How to apply

Load the deposited Known and Random enzyme–substrate pair files from the Zenodo deposit (zenodo.org/records/15697913), then restore the pretrained model state with loadGnnSomState and instantiate the scorer via createGnnSom. Compute a combined GNN-SOM + ReactionDecoder score for every pair in both sets, where GNN-SOM supplies site-of-metabolism likelihoods and ReactionDecoder supplies reaction-center predictions. Summarize each score distribution (median and mean) and confirm Known ≈ 0.86/0.70 and Random ≈ 0.29/0.39 before running the two-sided Mann-Whitney U test (scipy.stats.mannwhitneyu), expecting U=2573.0, p=4.4e-07. Write the per-pair table (set label, enzyme, substrate, score) to gnnsom_scores.csv and the test statistics plus summary statistics to mannwhitney_result.json. The rationale is that a scoring mechanism that prioritizes true reactions should place experimentally characterized pairs distinctly above random ones, making the separation itself the acceptance criterion.

## Related tools

- **MEANtools** (Framework providing loadGnnSomState and createGnnSom for restoring the pretrained GNN-SOM state and instantiating the reaction-likelihood scorer) — github.com/IntegrativeBioinformaticsLab/meantools
- **createGnnSom** (Instantiates the GNN-SOM scorer used to compute reaction-likelihood scores for each enzyme–substrate pair) — github.com/IntegrativeBioinformaticsLab/meantools
- **loadGnnSomState** (Restores the pretrained GNN-SOM model state prior to scoring) — github.com/IntegrativeBioinformaticsLab/meantools
- **ReactionDecoder** (Supplies reaction-center predictions that are combined with GNN-SOM site-of-metabolism likelihoods into the final pair score)
- **scipy.stats.mannwhitneyu** (Two-sided Mann-Whitney U test comparing Known vs Random score distributions)

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(known_scores, random_scores, alternative='two-sided')
```

## Evaluation signals

- Known set score summary matches median 0.86 / mean 0.70
- Random set score summary matches median 0.29 / mean 0.39
- Two-sided Mann-Whitney U test reproduces U=2573.0, p=4.4e-07 for Known vs Random
- gnnsom_scores.csv contains one row per pair with exactly the fields set label, enzyme, substrate, score
- All emitted scores fall within the valid likelihood range and Known scores are predominantly above Random scores

## Limitations

- The validation criterion is distributional separation between Known and Random pairs; a passing Mann-Whitney result does not rank individual candidate reactions within a set.
- Mass spectrometric data does not capture stereoisomers, which hindered pathway reconstruction in this context and can complicate interpretation of reaction predictions.
- The score combines a learned GNN-SOM likelihood with ReactionDecoder reaction-center prediction, so errors in either component propagate to the final score.

## Evidence

- [results] Reaction-likelihood scores for experimentally characterized enzyme-substrate (Known) pairs are significantly higher than for random pairs: [evidence span withheld — anchor status: not_found]
- [results] median/mean 0.86/0.70 versus 0.29/0.39, confirming the scoring mechanism prioritizes true reactions: [evidence span withheld — anchor status: not_found]
- [methods] Restore the pretrained model state with loadGnnSomState and instantiate the scorer via createGnnSom: [evidence span withheld — anchor status: not_found]
- [discussion] MEANtools can generate testable hypotheses on metabolic pathways with little to no prior knowledge, by integrating metabolomics and transcriptomics data.: "MEANtools can generate testable hypotheses on metabolic pathways with little to no prior knowledge, by integrating metabolomics and transcriptomics data."
- [readme] Read the documentation here: https://meantools.readthedocs.io/en/latest/: "Read the documentation here: https://meantools.readthedocs.io/en/latest/"
