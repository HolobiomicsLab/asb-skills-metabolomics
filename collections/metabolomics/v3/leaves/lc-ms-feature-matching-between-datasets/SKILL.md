---
name: lc-ms-feature-matching-between-datasets
description: Use when two LC–MS feature tables (XCMS peak-picked, post-QC, each with
  RT, m/z, and FI medians per feature) must be matched across batches or cohorts,
  and Step 1 candidate matching leaves features in clusters with multiple matches
  requiring unique correspondence.
when_to_use_negative:
- Datasets have not been separately peak-picked and QC-filtered first — the method
  expects post-QC feature tables (e.g., features failing the R2 < 0.7 dilution-series
  QC already removed)
- When RT, m/z, or FI values are unavailable or unreliable for one of the datasets,
  since the method requires only these three variables and behaves very differently
  for features that deviate from modeled inter-dataset shifts
- When an existing validated merger for the same data is available (e.g., metabCombiner
  was run as the published alternative comparison); consider benchmarking both rather
  than defaulting to M2S
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: M2S (Match 2 Sets)
  role: Free, open-source MATLAB toolbox implementing the three-step feature correspondence
    method (candidate matching, unique correspondence, poor-match removal)
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Environment in which M2S runs; used to load feature tables and compute neighbor
    definitions, residuals, and penalization scores
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Used to separately peak-pick both datasets, producing the per-feature RT/m/z/FI
    tables that are the input to matching
- name: metabCombiner
  role: Published alternative method used to benchmark M2S results on three other
    dataset pairs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:37e223d7a5bb4ced76689fd3ab63331bc4b7a374ca0e65fdda5ea47f612161cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  - XCMS
  - metabCombiner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lc-ms-feature-matching-between-datasets

## Summary

Finds feature correspondence between two untargeted LC–MS metabolomics datasets using only retention time (RT), m/z (MZ), and feature intensity (FI) values, via the three-step M2S method (candidate matching within thresholds, unique correspondence resolution, and optional poor-match removal) implemented in a free, open-source MATLAB toolbox.

## When to use

Use when two LC–MS feature tables (XCMS peak-picked, post-QC, each with RT, m/z, and FI medians per feature) must be matched across batches or cohorts, and Step 1 candidate matching leaves features in clusters with multiple matches requiring unique correspondence.

## When NOT to use

- Datasets have not been separately peak-picked and QC-filtered first — the method expects post-QC feature tables (e.g., features failing the R2 < 0.7 dilution-series QC already removed)
- When RT, m/z, or FI values are unavailable or unreliable for one of the datasets, since the method requires only these three variables and behaves very differently for features that deviate from modeled inter-dataset shifts
- When an existing validated merger for the same data is available (e.g., metabCombiner was run as the published alternative comparison); consider benchmarking both rather than defaulting to M2S

## Inputs

- Post-QC feature table for the reference dataset (e.g., MESA) with RT, m/z, and FI medians per feature, peak-picked separately with XCMS
- Post-QC feature table for the target dataset (e.g., Rotterdam) with the same columns
- Analyst-chosen RT/MZ/FI matching thresholds for Step 1, guided by visual inspection of distance plots

## Outputs

- A table of unique feature correspondences (feature pairs) between the two datasets
- Per-candidate-match penalization scores (weighted sum of squares of median + 3×MAD-normalized RT, MZ, log10FI residuals)
- Unique match counts per dataset pair (e.g., Dataset 1: 5365; Dataset 2: 2486; Dataset 3: 2728)
- Optionally, a list of deleted poor matches from Step 3

## How to apply

Load the post-QC feature tables (RT, m/z, and FI medians per feature) for the reference dataset (e.g., MESA, 2656 samples) and target dataset (e.g., Rotterdam, 1057 samples), which were separately peak-picked with XCMS and QC-filtered using a dilution series (R2 < 0.7 removed). In Step 1, match all variables within RT, MZ, and FI thresholds. In Step 2, resolve multiple-match clusters: define neighbors with the 'cross' method using 1% of the total number of features in the reference dataset, robustly determine the expected inter-dataset shift, compute residuals in RT, MZ, and log10FI, normalize residuals per dimension using median + 3 × MAD, apply weights W_RT,MZ,FI = [1, 1, 0.2], compute penalization scores as the weighted sum of squared residuals, and recursively select best matches until every feature has a single match. Optionally apply Step 3 to detect and delete poor matches by tightening thresholds — a decision the article explicitly leaves to the analyst. Threshold selection is guided by visual inspection of the distance plots from the initial matching.

## Related tools

- **M2S (Match 2 Sets)** (Free, open-source MATLAB toolbox implementing the three-step feature correspondence method (candidate matching, unique correspondence, poor-match removal)) — https://github.com/rjdossan/M2S
- **MATLAB** (Environment in which M2S runs; used to load feature tables and compute neighbor definitions, residuals, and penalization scores) — https://github.com/rjdossan/M2S
- **XCMS** (Used to separately peak-pick both datasets, producing the per-feature RT/m/z/FI tables that are the input to matching)
- **metabCombiner** (Published alternative method used to benchmark M2S results on three other dataset pairs)

## Evaluation signals

- Unique match counts fall in the reported ranges: 4953 and 2324 uniquely matched features in the main examples; 5365, 2486, and 2728 unique matches for Datasets 1–3
- Of manually annotated features, 585 of 604 (96.8%) were correctly matched in the first example and 79 of 87 (90.8%) in the second
- Residuals are normalized per dimension with median + 3 × MAD and combined with weights [1, 1, 0.2] — verify the penalization score equals the weighted sum of squared normalized residuals
- Neighbor definition uses the 'cross' method with k set to 1% of the total number of reference-dataset features
- On simulated data with 4755 features per dataset, 99.6% of matches were correct — sanity-check synthetic benchmark accuracy at this level

## Limitations

- Threshold selection is subjective, guided by visual inspection of the distance plots resulting from the initial matching
- The decision on whether to use Step 3 (poor-match deletion) rests with the analyst, and this optional step may inadvertently delete some correct matches
- Validation by annotated features is limited by the number of annotations in the datasets
- Most missed annotated matches are between features that behave very differently from the modeled inter-dataset shifts of RT, MZ, and FI
- No software version is stated for M2S

## Evidence

- [abstract] The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox"
- [abstract] 4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct: "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
- [methods] Both datasets were (separately) peak-picked using XCMS,13: "Both datasets were (separately) peak-picked using XCMS,13"
- [methods] a QC dilution series14 was used to eliminate features whose response to dilution was worse than a threshold of R2 <0.7: "a QC dilution
series14 was used to eliminate features
whose response to dilution was worse than a threshold of R2 <0.7."
- [discussion] guided by visual inspection of the distance plots resulting from the initial matching: "guided by
visual inspection of the distance plots resulting from the initial
matching"
- [readme] Matlab package to match untargeted metabolomic features of two LC-MS datasets: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
