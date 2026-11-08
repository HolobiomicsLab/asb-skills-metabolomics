---
name: weighted-penalty-score-match-selection
description: Use when two post-QC LC–MS feature tables (RT, m/z, FI medians) have
  been candidate-matched but some features remain in clusters with multiple candidate
  matches and need unique one-to-one correspondence.
when_to_use_negative:
- Features already have unique one-to-one correspondences after Step 1 candidate matching
  (no multi-match clusters to resolve)
- Datasets were acquired under very different conditions such that a single expected
  inter-dataset RT/mz/FI shift model is inappropriate — consider metabCombiner-style
  RT shift modeling instead
- The goal is raw peak-picking or annotation rather than correspondence between two
  already peak-picked feature tables (use XCMS or annotation databases like METLIN/HMDB)
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: M2S (Match 2 Sets)
  role: MATLAB toolbox implementing the recursive weighted-penalty-score selection
    of unique feature correspondences
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Execution environment for M2S; neighbor definition with 'cross' method and
    scoring computation
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Upstream separate peak-picking of both datasets to produce the feature tables
    that Step 2 consumes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:5d80699a75e98f24b4fc6466a38d6d3b6c750f2932d943dcdcd4c1f88d048a61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  - XCMS
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# weighted-penalty-score-match-selection

## Summary

M2S Step 2 resolves multi-match feature clusters from candidate inter-dataset matching by scoring each candidate pair with a weighted sum of squared, MAD-normalized residuals in RT, m/z, and log10 FI, then recursively selecting the best match until every feature has a single correspondence.

## When to use

Use when two post-QC LC–MS feature tables (RT, m/z, FI medians) have been candidate-matched but some features remain in clusters with multiple candidate matches and need unique one-to-one correspondence.

## When NOT to use

- Features already have unique one-to-one correspondences after Step 1 candidate matching (no multi-match clusters to resolve)
- Datasets were acquired under very different conditions such that a single expected inter-dataset RT/mz/FI shift model is inappropriate — consider metabCombiner-style RT shift modeling instead
- The goal is raw peak-picking or annotation rather than correspondence between two already peak-picked feature tables (use XCMS or annotation databases like METLIN/HMDB)

## Inputs

- Post-QC feature tables from reference (e.g., MESA) and target (e.g., Rotterdam) datasets with per-feature medians of retention time, m/z, and feature intensity
- Step 1 candidate matches (features in clusters with multiple candidate correspondences across thresholds)
- Number of features in the reference dataset (to set k = 1% nearest neighbors)
- Dimension weights W_RT,MZ,FI = [1, 1, 0.2]

## Outputs

- Unique one-to-one feature correspondence table (matched reference–target feature pairs)
- Per-candidate penalization scores (weighted sum of squares of normalized RT/MZ/log10FI residuals)
- step2_unique_matches.csv recording unique match counts per dataset pair (e.g., 5365, 2486, 2728)

## How to apply

In MATLAB with the M2S toolbox, define neighbors using the 'cross' method with k = 1% of the total number of features in the reference dataset to estimate the expected inter-dataset shift in RT, MZ, and log10 FI. Compute residuals of each candidate match against the robustly determined expected shift and normalize them per dimension using median + 3 × MAD. Apply dimension weights W_RT,MZ,FI = [1, 1, 0.2] and compute a penalization score per candidate as the weighted sum of squares of the normalized residuals. Recursively select the best match (lowest penalization score) and re-score until every feature in the cluster has a single match, then record unique match counts (e.g., 5365, 2486, 2728 for Datasets 1, 2, 3). Optionally run Step 3 tightening of thresholds to delete poor matches, a decision left to the analyst.

## Related tools

- **M2S (Match 2 Sets)** (MATLAB toolbox implementing the recursive weighted-penalty-score selection of unique feature correspondences) — https://github.com/rjdossan/M2S
- **MATLAB** (Execution environment for M2S; neighbor definition with 'cross' method and scoring computation) — https://github.com/rjdossan/M2S
- **XCMS** (Upstream separate peak-picking of both datasets to produce the feature tables that Step 2 consumes)

## Examples

```
matlab -r "addpath(genpath('M2S')); M2S_run_step2('reference_mesa.csv','target_rotterdam.csv','cross',0.01,[1 1 0.2])"
```

## Evaluation signals

- Every feature in the reference and target datasets has exactly one matched counterpart after Step 2 (no multi-match clusters remain)
- Unique match counts fall in expected ranges for the published dataset pairs (5365 for Dataset 1, 2486 for Dataset 2, 2728 for Dataset 3)
- On annotated features, correctness should approach reported rates (96.8% of 604 annotated features correct in the main example; 90.8% of 87 in a second example)
- On simulated data with 4755 features per dataset, ~99.6% of matches should be correct
- Normalized residuals per RT/MZ/log10FI dimension use median + 3 × MAD scaling, and the FI dimension weight is 0.2 vs 1.0 for RT and MZ — verify the score is the weighted sum of squared residuals

## Limitations

- Threshold selection is subjective and guided by visual inspection of the distance plots resulting from the initial matching
- The decision on when to use Step 3 to detect and delete poor matches rests with the analyst, and this optional step may inadvertently delete some correct matches
- Validation by annotated features is limited by the number of annotations in the datasets
- Most missed annotated matches are between features that behave very differently from modeled inter-dataset shifts of RT, MZ, and FI
- No software version stated for M2S

## Evidence

- [other] M2S defines the expected inter-dataset RT/MZ/FI shift from k nearest feature neighbors, normalizes residuals, computes a weighted penalization score per candidate match: [evidence span withheld — anchor status: not_found]
- [methods] weights and neighbor fraction: [evidence span withheld — anchor status: not_found]
- [methods] weights and score definition: [evidence span withheld — anchor status: not_found]
- [abstract] toolbox availability: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox"
- [abstract] missed matches caveat: "Most of the missed annotated matches are between features that behave very differently from modeled inter-dataset shifts of RT, MZ, and FI"
- [discussion] analyst discretion on step 3: "The decision on when to use step 3 to detect and delete poor matches rests with the analyst"
- [readme] readme: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
