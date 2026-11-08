---
name: average-distance-modification-site-evaluation
description: Use when benchmarking atom-level modification site likelihood scores
  (e.g., MF-R/MF-O outputs, random baselines, CFM-ID analog permutations) on analog
  pairs and you need a metric that rewards both closeness to the true site and score
  informativeness.
when_to_use_negative:
- When the task is compound identification rather than modification site localization
  — MS/MS library search cosine similarity, not Average-distance over atoms, is the
  appropriate metric.
- When only a single top-ranked atom is needed with no score distribution — the paper
  explicitly notes top-1 accuracy would reward a constant-score algorithm that assigns
  every atom the same score.
- When no 2D molecular graph of the S-known compound is available, since the metric
  requires 2D graph distances and the graph diameter.
tools:
- name: ModiFinder
  role: Produces the per-atom likelihood distributions (MF-R/MF-O) that are scored
    with the Average-distance evaluation method
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: CFM-ID
  role: Alternative approach benchmarked under the same metric; scores permuted analogs
    by MS/MS prediction cosine similarity in a docker container with prob_thresh 0.001
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:4c1965cfa6c0b03ade44ca0b886d65ceec1da3cc2b729ac052b366f0019cd093
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - CFM-ID
  - ModiFinder
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# average-distance-modification-site-evaluation

## Summary

An evaluation metric for modification site localization (the paper's 'Evaluation Score') that scores an atom-level likelihood distribution by exponential decay over 2D molecular graph distance from high-scoring atoms to the true modification site, normalized by graph diameter. It balances proximity cover and ambiguity cover, unlike top-1 accuracy.

## When to use

Use when benchmarking atom-level modification site likelihood scores (e.g., MF-R/MF-O outputs, random baselines, CFM-ID analog permutations) on analog pairs and you need a metric that rewards both closeness to the true site and score informativeness.

## When NOT to use

- When the task is compound identification rather than modification site localization — MS/MS library search cosine similarity, not Average-distance over atoms, is the appropriate metric.
- When only a single top-ranked atom is needed with no score distribution — the paper explicitly notes top-1 accuracy would reward a constant-score algorithm that assigns every atom the same score.
- When no 2D molecular graph of the S-known compound is available, since the metric requires 2D graph distances and the graph diameter.

## Inputs

- Per-atom likelihood distributions over the S-known compound (ModiFinder MF-R/MF-O scores after threshold 0.5 and power-of-4 post-processing)
- True modification site atom(s) for each analog pair
- 2D molecular graph of the S-known compound (for graph-distance and diameter computation)
- Baseline likelihood distributions: Random Choice (uniform atom), Random Distribution (Normal(0,1) per atom), CFM-ID cosine similarities from permuted analog predictions

## Outputs

- Per-pair Average-distance Evaluation Scores for each method
- Average deltas vs baselines (MF-R vs RC/RD, MF-O vs RC/RD, MF-R vs CFM-ID)
- Pairwise win fractions and baseline_comparison_results.csv

## How to apply

For each benchmark analog pair, take the per-atom likelihood distribution over the S-known compound (after ModiFinder post-processing where values below 0.5 are set to 0 and all values are raised to the power of 4). Score the distribution with the Average-distance method: weight each atom by its likelihood and compute an exponentially decaying function of the 2D graph distance between that atom and the true modification site, normalized by the graph diameter. Apply the identical scorer to Random Choice (uniform atom pick), Random Distribution (per-atom Normal(0,1) scores), and the CFM-ID alternative (cosine similarity of each permuted-analog MS/MS prediction against MS2-unknown). Aggregate deltas relative to baselines (e.g., MF-R vs RC +0.181, MF-O vs RC +0.208) and pairwise win fractions (MF-R beats RC/RD in ~81%/80% of pairs, MF-O in 85%) to verify reproduction against the reported values.

## Related tools

- **ModiFinder** (Produces the per-atom likelihood distributions (MF-R/MF-O) that are scored with the Average-distance evaluation method) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **CFM-ID** (Alternative approach benchmarked under the same metric; scores permuted analogs by MS/MS prediction cosine similarity in a docker container with prob_thresh 0.001) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
docker run wishartlab/cfmid cfm-predict --prob_thresh 0.001 <analog_permutation.mgf>  # then score cosine similarities vs MS2-unknown with Average-distance
```

## Evaluation signals

- Reproduced MF-O vs Random Choice average delta falls within tolerance of +0.208 and MF-O vs Random Distribution within +0.207.
- MF-R deltas fall within tolerance of +0.181/+0.180 with win rates near 81%/80% of pairs.
- MF-R vs CFM-ID average delta approximates +0.266, with the CFM-ID approach scoring worse than RC and RD on average.
- Constant-score distributions (all atoms equal) are not rewarded, distinguishing Average-distance from top-1 accuracy.
- All per-pair scores, deltas, and win rates are written to baseline_comparison_results.csv and match reported values.

## Limitations

- Requires a valid 2D molecular graph and known true modification site, so it only applies to labeled benchmark analog pairs (precursor < 2000 Da, mass difference < 50% of the smaller compound, shared M+H adduct, structures differing at exactly one modification site).
- Scores depend on the likelihood distribution's post-processing (0.5 threshold, power of 4), so raw unpost-processed scores are not directly comparable.
- Random Distribution baselines draw scores from Normal(0,1), so stochastic replication requires fixing the sampling procedure.

## Evidence

- [methods] Average-distance metric rationale: [evidence span withheld — anchor status: not_found]
- [methods] Metric computation detail: [evidence span withheld — anchor status: not_found]
- [methods] Likelihood post-processing before evaluation: "any value below 0.5 is set to 0 and to further highlight the differences, especially in the high-scoring atoms, all the values are raised to the power of 4"
- [intro] Evaluation scope of the paper: [evidence span withheld — anchor status: not_found]
- [abstract] Per-atom scoring output: "ModiFinder synthesizes this information together and scores the likelihood for each atom in the known molecule to be the modification site."
