---
name: simulated-data-validation
description: 'Use when you need an annotation-free accuracy estimate of a feature-matching
  method: simulate two LC–MS feature tables with known true pairs, run matching, and
  score match correctness, retention (e.g. 2682/2728), and precision (99.6%).'
when_to_use_negative:
- The task is annotation-based validation on real biological datasets where manual
  or database annotations (confidence level 2 per Metabolomics Standards Initiative)
  already provide ground truth — simulated data adds little.
- Feature tables lack FI (intensity) values or the m/z/RT structure needed to embed
  realistic shift trends.
- The goal is annotation-coverage assessment (e.g. how many features can be annotated
  to LIPID MAPS, METLIN, HMDB, GNPS, or MassBank) rather than matching-accuracy estimation.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: M2S (Match 2 Sets)
  role: Runs the matching pipeline (Steps 1–3) on the simulated dataset pair and produces
    the match tables whose counts and accuracy are validated
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Environment in which M2S and the simulation/validation scripts are executed
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Separate peak-picking of the two simulated datasets before matching, mirroring
    the real-data workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:d7e126b24ae15557e2891d47404bcf90de394f0f95c5edb224cc26acabe6b02c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  - XCMS
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Simulated-data validation

## Summary

Generates synthetic paired LC–MS feature lists (4755 features per dataset) whose true correspondences are known, to quantify matching accuracy (99.6% correct) of M2S without relying on metabolite annotations. It also validates published-match-count ratios such as 2682/2717 = 98.71% after Step 3 poor-match filtering.

## When to use

Use when you need an annotation-free accuracy estimate of a feature-matching method: simulate two LC–MS feature tables with known true pairs, run matching, and score match correctness, retention (e.g. 2682/2728), and precision (99.6%).

## When NOT to use

- The task is annotation-based validation on real biological datasets where manual or database annotations (confidence level 2 per Metabolomics Standards Initiative) already provide ground truth — simulated data adds little.
- Feature tables lack FI (intensity) values or the m/z/RT structure needed to embed realistic shift trends.
- The goal is annotation-coverage assessment (e.g. how many features can be annotated to LIPID MAPS, METLIN, HMDB, GNPS, or MassBank) rather than matching-accuracy estimation.

## Inputs

- Simulated or real paired LC–MS feature tables with RT, m/z, and FI columns (e.g. Dataset 3: 4755 features per dataset)
- Ground-truth correspondence mapping between simulated reference and target features
- Post-Step-2 unique match tables for Dataset 1 (MESA–Rotterdam, 10427 × 14097 features) and Dataset 3

## Outputs

- match_counts_step3.tsv with dataset, matches_before, matches_deleted, matches_after columns
- Correct-match proportion from simulated ground truth (expected 99.6% for Dataset 3)
- Retained-match ratio per dataset (Dataset 3: 2682/2717 = 98.71% after Step 3; Dataset 1: 4953/5365 retained)

## How to apply

Construct a simulated reference–target pair where each dataset contains 4755 features with RT, m/z, and FI values, embedding a known ground-truth correspondence between them. Run the matching pipeline on the simulated pair: peak-pick with XCMS, compute robust inter-dataset RT, MZ, and log10FI shifts using the 'cross' neighbor method (1% of reference features as neighbors), match within thresholds (Steps 1–2), and optionally apply Step 3 poor-match filtering with residuals normalized via median + 3 × MAD ('scores' nonlinear threshold method, W_RT,MZ,FI = [1, 1, 0.2]). Score results against the embedded ground truth: check that retained matches equal the expected counts (Dataset 3 = 2682 from 2728 within ±2) and that the correct-match proportion is 99.6%. Compare with real-data behavior (Dataset 1 MESA–Rotterdam retains 4953 of 5365 after Step 3) to confirm the method handles both synthetic and biological noise structures.

## Related tools

- **M2S (Match 2 Sets)** (Runs the matching pipeline (Steps 1–3) on the simulated dataset pair and produces the match tables whose counts and accuracy are validated) — https://github.com/rjdossan/M2S
- **MATLAB** (Environment in which M2S and the simulation/validation scripts are executed) — https://github.com/rjdossan/M2S
- **XCMS** (Separate peak-picking of the two simulated datasets before matching, mirroring the real-data workflow)

## Examples

```
% MATLAB: run simulated Dataset 3 through M2S Steps 1–3 and score vs ground truth
 run('M2S_run_Dataset3.m'); validateStep3Counts('match_counts_step3.tsv', expectedAfter=2682, tol=2);
```

## Evaluation signals

- Retained matches after Step 3 for Dataset 3 = 2682 (from 2728) within ±2 matches tolerance
- Retained/expected ratio for Dataset 3 = 2682/2717 = 98.71%
- Correct-match proportion on simulated ground truth = 99.6%
- Dataset 1 (MESA–Rotterdam) retains 4953 of 5365 unique matches after Step 3, matching the abstract's main-example count
- match_counts_step3.tsv columns are populated for every dataset in the simulation study

## Limitations

- Step 3 poor-match deletion is optional and analyst-dependent; applying it may inadvertently delete some correct matches.
- Threshold selection is subjective, guided by visual inspection of the distance plots from the initial matching.
- Validation by annotated features is limited by the number of annotations in the datasets, which is precisely why simulated ground truth is useful but cannot capture all real-data complexity.
- Simulated 4755-feature datasets do not reproduce the full feature-count scale of real data (e.g. 10427 MESA features, 14097 Rotterdam features).

## Evidence

- [methods] Simulated dataset validation: "In a third example with simulated data with 4755 features per dataset, 99.6% of the matches were correct"
- [methods] Step 3 match retention on Dataset 3: [evidence span withheld — anchor status: not_found]
- [methods] Step 3 match retention on Dataset 1: [evidence span withheld — anchor status: not_found]
- [methods] Normalization and threshold method for Step 3: [evidence span withheld — anchor status: not_found]
- [abstract] M2S toolbox availability: "M2S (Match 2 Sets), a free, open-source MATLAB toolbox, available at https://github.com/rjdossan/M2S"
- [discussion] Limitation of annotation-based validation: "while being limited by the number of annotations in our datasets"
- [discussion] Optional nature of Step 3 and risk of deleting correct matches: "this final/optional step may inadvertently delete some correct matches"
