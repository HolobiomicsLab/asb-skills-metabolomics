---
name: nonparametric-hypothesis-testing
description: Use when two independent score distributions (e.g., Known vs Random enzyme-substrate
  pairs scored via MEANtools createGnnSom) must be compared nonparametrically, with
  reported statistics available for validation.
when_to_use_negative:
- When the two score sets are paired or matched observations rather than independent
  samples (the Mann-Whitney U test used here assumes independence between Known and
  Random pairs).
- When per-pair reaction-likelihood scores cannot be computed because the pretrained
  GNN-SOM state or the deposited Known/Random pair files are unavailable.
- When the goal is pathway reconstruction from transcriptomic-metabolomic data rather
  than validating that a scoring mechanism separates true reactions from random pairs.
tools:
- name: MEANtools
  role: Framework whose createGnnSom/loadGnnSomState functions produce the GNN-SOM
    + ReactionDecoder reaction-likelihood scores that are statistically compared
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: scipy.stats.mannwhitneyu
  role: Runs the two-sided Mann-Whitney U test comparing Known vs Random score distributions
- name: createGnnSom
  role: Instantiates the GNN-SOM scorer used to score each enzyme-substrate pair before
    testing
- name: loadGnnSomState
  role: Restores the pretrained GNN-SOM model state prior to scoring
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:49d0499348a6496767a1d10aad14637565c2aced70dfb46e7723d538c2d80978
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MEANtools
  - createGnnSom
  - loadGnnSomState
  - scipy.stats.mannwhitneyu
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# nonparametric-hypothesis-testing

## Summary

Applies the two-sided Mann-Whitney U test (scipy.stats.mannwhitneyu) to compare distributions of GNN-SOM + ReactionDecoder reaction-likelihood scores between experimentally characterized (Known) and randomly assembled (Random) enzyme-substrate pairs, establishing that the scoring mechanism prioritizes true reactions without distributional assumptions.

## When to use

Use when two independent score distributions (e.g., Known vs Random enzyme-substrate pairs scored via MEANtools createGnnSom) must be compared nonparametrically, with reported statistics available for validation.

## When NOT to use

- When the two score sets are paired or matched observations rather than independent samples (the Mann-Whitney U test used here assumes independence between Known and Random pairs).
- When per-pair reaction-likelihood scores cannot be computed because the pretrained GNN-SOM state or the deposited Known/Random pair files are unavailable.
- When the goal is pathway reconstruction from transcriptomic-metabolomic data rather than validating that a scoring mechanism separates true reactions from random pairs.

## Inputs

- Known enzyme-substrate pair file from the Zenodo deposit (zenodo.org/records/15697913)
- Random enzyme-substrate pair file from the Zenodo deposit (zenodo.org/records/15697913)
- Pretrained GNN-SOM model state loaded via loadGnnSomState

## Outputs

- Per-pair score table gnnsom_scores.csv with columns: set label, enzyme, substrate, score
- mannwhitney_result.json containing U statistic, p-value, and per-set median/mean summary statistics

## How to apply

Score every enzyme-substrate pair in both the Known and Random sets by restoring the pretrained GNN-SOM state with loadGnnSomState, instantiating the scorer via createGnnSom, and combining GNN-SOM site-of-metabolism likelihoods with ReactionDecoder reaction-center predictions. Compute the median and mean of each set's score distribution and confirm they match the reported values (Known: 0.86/0.70; Random: 0.29/0.39). Then run the two-sided Mann-Whitney U test with scipy.stats.mannwhitneyu on the two independent score vectors — chosen because no normality assumption on the skewed score distributions is warranted — and check the result against the reported U=2573.0, p=4.4e-07. Persist the per-pair table (set label, enzyme, substrate, score) to gnnsom_scores.csv and the test statistics plus summary statistics to mannwhitney_result.json.

## Related tools

- **MEANtools** (Framework whose createGnnSom/loadGnnSomState functions produce the GNN-SOM + ReactionDecoder reaction-likelihood scores that are statistically compared) — github.com/IntegrativeBioinformaticsLab/meantools
- **scipy.stats.mannwhitneyu** (Runs the two-sided Mann-Whitney U test comparing Known vs Random score distributions)
- **createGnnSom** (Instantiates the GNN-SOM scorer used to score each enzyme-substrate pair before testing)
- **loadGnnSomState** (Restores the pretrained GNN-SOM model state prior to scoring)

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(known_scores, random_scores, alternative='two-sided')
```

## Evaluation signals

- Known set score distribution must reproduce median/mean of 0.86/0.70.
- Random set score distribution must reproduce median/mean of 0.29/0.39.
- Two-sided Mann-Whitney U test must return U=2573.0 and p=4.4e-07.
- gnnsom_scores.csv must contain exactly the schema: set label, enzyme, substrate, score.
- mannwhitney_result.json must include both test statistics (U, p) and the per-set summary statistics.

## Limitations

- Reproducibility depends on using the exact deposited Known and Random pair files from Zenodo record 15697913 and the correct pretrained GNN-SOM state; deviations in U or p indicate a mismatched model state or pair set.
- The Mann-Whitney U result only demonstrates rank-based separation of the two distributions (Known higher than Random); it does not by itself establish per-pair score calibration.
- The comparison is defined for two independent groups; extending to additional pair sets requires a different test design.

## Evidence

- [other] Run the two-sided Mann-Whitney U test (scipy.stats.mannwhitneyu) comparing Known vs Random scores and check the result against the reported U=2573.0, p=4.4e-07.: [evidence span withheld — anchor status: not_found]
- [other] Reaction-likelihood scores for experimentally characterized enzyme-substrate (Known) pairs are significantly higher than for random pairs, with median/mean 0.86/0.70 versus 0.29/0.39.: [evidence span withheld — anchor status: not_found]
- [readme] Read the documentation here:

https://meantools.readthedocs.io/en/latest/: "Read the documentation here:

https://meantools.readthedocs.io/en/latest/"
