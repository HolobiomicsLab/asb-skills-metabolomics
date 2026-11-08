---
name: enzyme-substrate-pair-scoring
description: Use when you have curated Known enzyme-substrate pairs (plus Random negatives)
  and need to score them with GNN-SOM + ReactionDecoder and test whether the score
  distributions separate.
when_to_use_negative:
- You want to discover biosynthetic pathways from paired transcriptomic-metabolomic
  data (e.g. the tomato falcarindiol case) rather than benchmark an existing scoring
  mechanism against known pairs.
- Your input is uncharacterized mass features without deposited Known/Random pair
  files or a pretrained GNN-SOM model state to restore.
- You need to predict reactions from mass shifts using RetroRules loose/medium/strict
  rulesets rather than validate GNN-SOM + ReactionDecoder scores.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0218
tools:
- name: MEANtools
  role: Host framework providing the GNN-SOM + ReactionDecoder scoring mechanism;
    pretrained model state is loaded and a scorer instantiated from its API.
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: createGnnSom
  role: MEANtools API function used to instantiate the GNN-SOM scorer that computes
    reaction-likelihood scores for each enzyme-substrate pair.
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: loadGnnSomState
  role: MEANtools API function used to restore the pretrained GNN-SOM model state
    before scoring.
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: scipy.stats.mannwhitneyu
  role: Two-sided Mann-Whitney U test comparing Known vs Random score distributions
    (expected U=2573.0, p=4.4e-07).
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:fb12edd15be69fe9c5bead1ca430375174ed70e6a1661e4b7331c9d6dab386f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - MEANtools
  - createGnnSom
  - loadGnnSomState
  - scipy.stats.mannwhitneyu
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# enzyme-substrate-pair-scoring

## Summary

Computes MEANtools reaction-likelihood scores (GNN-SOM site-of-metabolism likelihoods combined with ReactionDecoder reaction-center prediction) for enzyme-substrate pairs and validates the scoring mechanism by showing Known experimentally characterized pairs score significantly higher than Random pairs.

## When to use

Use when you have curated Known enzyme-substrate pairs (plus Random negatives) and need to score them with GNN-SOM + ReactionDecoder and test whether the score distributions separate.

## When NOT to use

- You want to discover biosynthetic pathways from paired transcriptomic-metabolomic data (e.g. the tomato falcarindiol case) rather than benchmark an existing scoring mechanism against known pairs.
- Your input is uncharacterized mass features without deposited Known/Random pair files or a pretrained GNN-SOM model state to restore.
- You need to predict reactions from mass shifts using RetroRules loose/medium/strict rulesets rather than validate GNN-SOM + ReactionDecoder scores.

## Inputs

- Known enzyme-substrate pair files (experimentally characterized pairs) from zenodo.org/records/15697913
- Random enzyme-substrate pair files (randomly assembled negatives) from the same Zenodo deposit
- Pretrained GNN-SOM model state (restored via loadGnnSomState)

## Outputs

- gnnsom_scores.csv — per-pair score table with columns: set label (Known/Random), enzyme, substrate, reaction-likelihood score
- mannwhitney_result.json — Mann-Whitney U statistic (expected U=2573.0, p=4.4e-07) plus per-set median and mean (Known 0.86/0.70; Random 0.29/0.39)

## How to apply

Load the deposited Known and Random enzyme-substrate pair files from the Zenodo deposit (zenodo.org/records/15697913) associated with MEANtools. Restore the pretrained model state with loadGnnSomState and instantiate the scorer via createGnnSom, then compute a GNN-SOM + ReactionDecoder reaction-likelihood score for every pair in both sets. Summarize each distribution's median and mean and check they reproduce the reported values (Known: median/mean 0.86/0.70; Random: 0.29/0.39). Run a two-sided Mann-Whitney U test (scipy.stats.mannwhitneyu) comparing Known vs Random scores, expecting U=2573.0, p=4.4e-07, which confirms the scoring mechanism prioritizes true reactions. Write the per-pair score table (set label, enzyme, substrate, score) to gnnsom_scores.csv and the test statistics plus summary statistics to mannwhitney_result.json.

## Related tools

- **MEANtools** (Host framework providing the GNN-SOM + ReactionDecoder scoring mechanism; pretrained model state is loaded and a scorer instantiated from its API.) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **createGnnSom** (MEANtools API function used to instantiate the GNN-SOM scorer that computes reaction-likelihood scores for each enzyme-substrate pair.) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **loadGnnSomState** (MEANtools API function used to restore the pretrained GNN-SOM model state before scoring.) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **scipy.stats.mannwhitneyu** (Two-sided Mann-Whitney U test comparing Known vs Random score distributions (expected U=2573.0, p=4.4e-07).)

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(known_scores, random_scores, alternative='two-sided')
```

## Evaluation signals

- Known set median and mean scores reproduce 0.86 and 0.70 respectively.
- Random set median and mean scores reproduce 0.29 and 0.39 respectively.
- Two-sided Mann-Whitney U test returns U=2573.0 and p=4.4e-07 for Known vs Random.
- gnnsom_scores.csv contains one row per pair with exactly the fields: set label, enzyme, substrate, score, covering both sets.
- mannwhitney_result.json records both the test statistics and the per-set summary statistics.

## Limitations

- Validation depends on the specific deposited Known/Random pair files and pretrained GNN-SOM state from the Zenodo deposit (zenodo.org/records/15697913); other pair sets or model states are not covered by the reported U=2573.0, p=4.4e-07 benchmark.
- MEANtools reaction-rule machinery is designed for single-substrate reactions only, which limits how multi-substrate conversions are represented in scored pairs.
- Mass spectrometric data does not capture stereoisomers, so stereomeric conversions are filtered out and cannot be distinguished in scored pairs.

## Evidence

- [other] Reaction-likelihood scores for experimentally characterized enzyme-substrate (Known) pairs are significantly higher than for random pairs, with median/mean 0.86/0.70 versus 0.29/0.39: [evidence span withheld — anchor status: not_found]
- [other] Run the two-sided Mann-Whitney U test (scipy.stats.mannwhitneyu) comparing Known vs Random scores and check the result against the reported U=2573.0, p=4.4e-07.: [evidence span withheld — anchor status: not_found]
- [other] Restore the pretrained model state with loadGnnSomState and instantiate the scorer via createGnnSom.: [evidence span withheld — anchor status: not_found]
- [discussion] The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913.: "The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913."
