---
name: correlated-feature-pattern-scoring
description: Use when validating matched feature pairs between two untargeted LC-MS
  metabolomics datasets (reference and target) after matching on RT, m/z, and FI,
  and you need an orthogonal, annotation-free quality metric computed from each feature's
  correlated neighbors.
when_to_use_negative:
- The two feature tables have not yet been matched (no correspondence table exists)
  — pattern scoring is a post-matching validation metric, not a matching method
- Only a handful of features per RT region exist, so correlated-feature neighborhoods
  are too sparse to distinguish correct from incorrect matches
- Intensity matrices lack sufficient shared sample structure for meaningful Spearman
  correlation (e.g., unmatched sample sets)
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: M2S (Match 2 Sets)
  role: MATLAB toolbox that produced the match tables being validated; its validation
    functions support correlated-feature pattern scoring
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Environment in which M2S and the patternScore computation are executed
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Used upstream to peak-pick each dataset separately, yielding the feature tables
    and intensity matrices consumed by this validation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:7fadfc84702c62b25f188fdbca6034c8e314042d6d1c0fa0aa4b8b483501e031
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  - MESA
  - Rotterdam
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# correlated-feature-pattern-scoring

## Summary

A validation strategy from the M2S feature-matching method that scores the quality of matched LC-MS feature pairs by comparing the patterns of highly correlated co-eluting features each matched feature has in its own dataset. Agreement in these correlated-feature neighborhoods (patternScore) indicates the correspondence reflects genuine chemistry rather than a chance RT/m/z/FI alignment.

## When to use

Use when validating matched feature pairs between two untargeted LC-MS metabolomics datasets (reference and target) after matching on RT, m/z, and FI, and you need an orthogonal, annotation-free quality metric computed from each feature's correlated neighbors.

## When NOT to use

- The two feature tables have not yet been matched (no correspondence table exists) — pattern scoring is a post-matching validation metric, not a matching method
- Only a handful of features per RT region exist, so correlated-feature neighborhoods are too sparse to distinguish correct from incorrect matches
- Intensity matrices lack sufficient shared sample structure for meaningful Spearman correlation (e.g., unmatched sample sets)

## Inputs

- Final M2S match table linking reference (e.g., MESA) and target (e.g., Rotterdam) features after step 3 poor-match removal, with RT, m/z, and median FI per feature
- Per-sample feature intensity matrices for both datasets (to compute Spearman correlations across samples)
- Feature tables with retention times (features eluting before 12 min for Dataset 1; 0.45–9.5 min window for Dataset 2)

## Outputs

- Per-match patternScore values based on common highly correlated features (Spearman > 0.7 within a 0.25 s RT window) between matched pairs
- Rows contributing to the six-metric summary validation table (CSV/TSV) for each dataset

## How to apply

For each matched feature pair from the final M2S match tables (after step 3 poor-match removal), work separately in each dataset: within a 0.25 s RT window around the feature, select features whose Spearman correlation with it exceeds 0.7. Then compare the two correlated-feature sets across the matched pair and compute the patternScore from the common highly correlated features, i.e., how consistently the same correlated features are shared between the reference and target neighborhoods. This exploits the fact that a true correspondence should carry the same co-eluting, co-varying adduct/isotopologue/in-source-fragment neighborhood in both datasets (features were peak-picked separately with XCMS and characterized by medians of RT, m/z, and FI across samples). Report the resulting per-match patternScores alongside the other five validation metrics (annotation agreement, log10FI correlation, covariate-association agreement, detection-sample counts, and unique-vs-multiple-match cluster ratios) in a summary metrics table. Elevated patternScores across matches support matching quality without requiring manual annotations, which are limited in number.

## Related tools

- **M2S (Match 2 Sets)** (MATLAB toolbox that produced the match tables being validated; its validation functions support correlated-feature pattern scoring) — https://github.com/rjdossan/M2S
- **MATLAB** (Environment in which M2S and the patternScore computation are executed) — https://github.com/rjdossan/M2S
- **XCMS** (Used upstream to peak-pick each dataset separately, yielding the feature tables and intensity matrices consumed by this validation)

## Examples

```
matlab -r "addpath(genpath('M2S')); patternScore = validateCorrelatedFeatures(matchTable, refIntensity, tgtIntensity, refRT, tgtRT, 'RTwindow', 0.25, 'corrThreshold', 0.7);"
```

## Evaluation signals

- Every scored match used a 0.25 s RT window and Spearman correlation threshold of exactly 0.7 for neighbor selection — verify parameters in the scoring code/outputs
- patternScore is computed for each entry of the final match table (post step-3 poor-match removal), and the column is present in the summary metrics table alongside all six metric categories
- Sanity check on known-correct matches: matches validated by manual annotation agreement (e.g., Dataset 1: 585/604 = 96.8% correct) should show comparatively high patternScores
- Cross-dataset neighborhoods: the correlated-feature sets compared must come from the reference and target datasets respectively, not from the same dataset twice

## Limitations

- Validation by annotated features is limited by the number of annotations in the datasets; patternScore partially mitigates this but still depends on locally dense, well-correlated features
- Threshold selection in M2S (including window and correlation cutoffs) is subjective, guided by visual inspection of distance plots
- Features behave very differently from modeled inter-dataset RT, MZ, and FI shifts (the main source of missed annotated matches) may also have atypical correlated-feature neighborhoods, deflating patternScores

## Evidence

- [methods] select correlated features within a 0.25 s RT window at Spearman correlation > 0.7 and compute the patternScore of common correlated features: [evidence span withheld — anchor status: not_found]
- [readme] Matlab package to match untargeted metabolomic features: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
- [abstract] six orthogonal validation strategies to gauge the matching quality: "using six orthogonal validation strategies to gauge the matching quality"
- [methods] the medians of RT, m/z, and FI across all samples were used: "the medians of RT, m/z, and FI across all samples were used."
