---
name: score-distribution-summary-statistics
description: Use when a prediction tool outputs per-pair scores for a known-positive
  set and a random-negative set, and you must confirm the score separates the two
  distributions with medians/means and a U test.
when_to_use_negative:
- Input is a single unlabeled score list with no paired positive (Known) and negative
  (Random) sets to contrast — median/mean alone cannot support a separation claim
- The goal is de novo pathway discovery from transcriptomic-metabolomic data rather
  than benchmarking a scoring mechanism against labeled pair sets
- Scores were produced by a different predictor whose reported reference statistics
  (0.86/0.70 vs 0.29/0.39, U=2573.0) do not apply
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MEANtools
  role: Framework whose GNN-SOM + ReactionDecoder scoring produces the reaction-likelihood
    scores being summarized and tested
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: createGnnSom
  role: Instantiates the GNN-SOM scorer used to compute a score for every enzyme-substrate
    pair in both sets
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: loadGnnSomState
  role: Restores the pretrained model state before scoring
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: scipy.stats.mannwhitneyu
  role: Two-sided Mann-Whitney U test comparing Known vs Random score distributions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:101c6e15398f68614d951b38858822c539faeaba756300c4b1dfb999ff33d16d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - MEANtools
  - createGnnSom
  - loadGnnSomState
  - scipy.stats.mannwhitneyu
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# score-distribution-summary-statistics

## Summary

Summarize a predicted enzyme-substrate reaction-likelihood score distribution with median/mean statistics and validate separation between positive (Known) and negative (Random) pair sets using a two-sided Mann-Whitney U test, as done for MEANtools GNN-SOM + ReactionDecoder scores.

## When to use

Use when a prediction tool outputs per-pair scores for a known-positive set and a random-negative set, and you must confirm the score separates the two distributions with medians/means and a U test.

## When NOT to use

- Input is a single unlabeled score list with no paired positive (Known) and negative (Random) sets to contrast — median/mean alone cannot support a separation claim
- The goal is de novo pathway discovery from transcriptomic-metabolomic data rather than benchmarking a scoring mechanism against labeled pair sets
- Scores were produced by a different predictor whose reported reference statistics (0.86/0.70 vs 0.29/0.39, U=2573.0) do not apply

## Inputs

- Known experimentally characterized enzyme-substrate pair file from zenodo.org/records/15697913
- Random (randomly assembled) enzyme-substrate pair file from zenodo.org/records/15697913
- Pretrained GNN-SOM model state loaded via loadGnnSomState
- ReactionDecoder reaction-center predictions combined with GNN-SOM site-of-metabolism likelihoods

## Outputs

- gnnsom_scores.csv with per-pair rows (set label, enzyme, substrate, score)
- mannwhitney_result.json with U statistic (expected 2573.0), p-value (expected 4.4e-07), and per-set median/mean (Known 0.86/0.70; Random 0.29/0.39)

## How to apply

Load the deposited positive and negative pair files from the Zenodo deposit (zenodo.org/records/15697913) and compute one score per pair with the model scorer (restore pretrained state with loadGnnSomState, then instantiate via createGnnSom). Summarize each set's score distribution as median and mean — for the MEANtools case the expected values are Known: median 0.86 / mean 0.70 and Random: median 0.29 / mean 0.39, so a correct scoring run should reproduce these. Then run the two-sided Mann-Whitney U test (scipy.stats.mannwhitneyu) comparing Known vs Random scores and check against the reported U=2573.0, p=4.4e-07. Persist a per-pair table (set label, enzyme, substrate, score) as gnnsom_scores.csv and the U statistic, p-value, and median/mean summary as mannwhitney_result.json. Matching both the summary statistics and the U-test result constitutes replication of the separation claim.

## Related tools

- **MEANtools** (Framework whose GNN-SOM + ReactionDecoder scoring produces the reaction-likelihood scores being summarized and tested) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **createGnnSom** (Instantiates the GNN-SOM scorer used to compute a score for every enzyme-substrate pair in both sets) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **loadGnnSomState** (Restores the pretrained model state before scoring) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **scipy.stats.mannwhitneyu** (Two-sided Mann-Whitney U test comparing Known vs Random score distributions)

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(known_scores, random_scores, alternative='two-sided')
```

## Evaluation signals

- Known-set median/mean of scores reproduces 0.86/0.70
- Random-set median/mean of scores reproduces 0.29/0.39
- Two-sided Mann-Whitney U test yields U=2573.0 and p=4.4e-07
- gnnsom_scores.csv contains exactly one row per pair with the correct set label, enzyme, substrate, and score columns
- Known median (0.86) substantially exceeds Random median (0.29), confirming the score prioritizes true reactions

## Limitations

- Scores depend on the pretrained GNN-SOM state and the exact deposited Known/Random pair files; a different model state or pair assembly will not reproduce the reference statistics
- Median/mean summaries alone do not establish significance — the Mann-Whitney U test result must be reported alongside them
- The scoring mechanism was validated only on the reported Known vs Random construction; other negative-sampling schemes may shift the distributions

## Evidence

- [abstract] Here, we present MEANtools, a systematic and unsupervised computational integrative omics workflow: "Here, we present MEANtools, a systematic and unsupervised computational integrative omics workflow"
- [discussion] The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913.: "The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913."
- [discussion] MEANtools can generate testable hypotheses on metabolic pathways with little to no prior knowledge, by integrating metabolomics and transcriptomics data.: "MEANtools can generate testable hypotheses on metabolic pathways with little to no prior knowledge, by integrating metabolomics and transcriptomics data."
