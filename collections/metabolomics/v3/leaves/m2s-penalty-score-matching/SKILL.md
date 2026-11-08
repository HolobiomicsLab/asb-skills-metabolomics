---
name: m2s-penalty-score-matching
description: Use when combining two separately peak-picked untargeted LC–MS metabolomics
  datasets (e.g. two batches or two cohorts) that have already gone through QC and
  peak-picking, and you need feature-level correspondence using only RT, m/z, and
  FI values.
when_to_use_negative:
- If the inputs are raw LC–MS spectra that have not yet been peak-picked into a feature
  table — the method operates on feature lists, and XCMS peak-picking must be done
  first.
- If the two datasets have very different sample types or lack comparable RT/mz/FI
  behavior, making inter-dataset shift modeling unreliable.
- If you need metabolite-level identification from scratch — this skill only establishes
  feature correspondence and does not perform annotation itself.
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: M2S (Match 2 Sets)
  role: MATLAB toolbox implementing the three-step feature correspondence workflow
    (threshold matching, unique correspondence via shift modeling and penalty score,
    poor-match deletion).
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Execution environment for M2S; the toolbox ships with examples in the highly
    interactive, graphically capable MATLAB environment.
- name: XCMS
  role: Used upstream to peak-pick each dataset separately, yielding the feature tables
    that M2S consumes.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:902faa87024b838cbab20de66b234f15266227764b327921f0b253c4c60282be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  - XCMS
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# m2s-penalty-score-matching

## Summary

A three-step feature correspondence method (implemented in the open-source MATLAB toolbox M2S) that finds matches between untargeted LC–MS metabolomics feature lists using only retention time, m/z, and feature intensity, with a penalty-score selection step for unique correspondence and an optional third step that deletes poor matches.

## When to use

Use when combining two separately peak-picked untargeted LC–MS metabolomics datasets (e.g. two batches or two cohorts) that have already gone through QC and peak-picking, and you need feature-level correspondence using only RT, m/z, and FI values.

## When NOT to use

- If the inputs are raw LC–MS spectra that have not yet been peak-picked into a feature table — the method operates on feature lists, and XCMS peak-picking must be done first.
- If the two datasets have very different sample types or lack comparable RT/mz/FI behavior, making inter-dataset shift modeling unreliable.
- If you need metabolite-level identification from scratch — this skill only establishes feature correspondence and does not perform annotation itself.

## Inputs

- Two separately peak-picked (e.g. via XCMS) untargeted LC–MS metabolomics feature tables with retention time, m/z, and (optionally) feature intensity per feature
- Feature-level medians of RT, m/z, and FI across samples for each dataset
- QC dilution-series response information (e.g. R2 per feature) used for prior quality control of features

## Outputs

- A set of uniquely matched feature pairs between the two datasets
- Distance plots from the initial matching used for visual threshold inspection
- Optionally, a pruned match set after deletion of poor matches in step 3

## How to apply

In MATLAB, run the M2S toolbox on two feature tables (medians of RT, m/z, and FI across samples per feature) produced by separate XCMS peak-picking of the datasets. Step 1 matches all variables within RT, m/z, and FI thresholds, with threshold selection guided by visual inspection of the resulting distance plots; the article's example retained only features eluting before 12 min and excluded features with a QC dilution-series response worse than R2 < 0.7. Step 2 finds unique correspondence between datasets by modeling inter-dataset shifts of RT, m/z, and FI and choosing matches by penalty score. Step 3 is an optional tightening of thresholds — poor matches are detected and deleted using nonlinear thresholds at median + 3×MAD — and the decision on whether to apply it rests with the analyst; e.g. a cluster of column-bleeding features at differing FI can be removed with a tighter upper FI threshold in the initial candidate matching. The result is a list of uniquely matched feature pairs that can be further annotated to confidence level 2 (Metabolomics Standards Initiative) against LIPID MAPS, METLIN, HMDB, GNPS, and MassBank.

## Related tools

- **M2S (Match 2 Sets)** (MATLAB toolbox implementing the three-step feature correspondence workflow (threshold matching, unique correspondence via shift modeling and penalty score, poor-match deletion).) — https://github.com/rjdossan/M2S
- **MATLAB** (Execution environment for M2S; the toolbox ships with examples in the highly interactive, graphically capable MATLAB environment.)
- **XCMS** (Used upstream to peak-pick each dataset separately, yielding the feature tables that M2S consumes.)

## Examples

```
In MATLAB: % after peak-picking with XCMS, load the two feature tables and run the M2S toolbox
% e.g. from https://github.com/rjdossan/M2S
```

## Evaluation signals

- On simulated Dataset 3 (4755 features per dataset), 99.6% of the matches were correct — compare your run's match-correctness fraction to this benchmark when the same simulated data is used.
- In the main MESA–Rotterdam example, 4953 features were uniquely matched, and 585 (96.8%) of the 604 manually annotated features were correct — verify your match count and annotated-match correctness against these numbers.
- In a second example, 2324 features were uniquely matched with 79 (90.8%) of 87 annotated features correctly matched — a comparable accuracy target for that dataset pair.
- Check that features eluting before the retention-time trim (before 12 min in the main example; 0.45–9.5 min in another example) were excluded before matching.
- Verify that step 3 (poor-match deletion) was applied only where justified: the decision rests with the analyst and the step may inadvertently delete some correct matches.

## Limitations

- Threshold selection is subjective, guided by visual inspection of the distance plots resulting from the initial matching.
- The decision on when to use step 3 to detect and delete poor matches rests with the analyst, and this final/optional step may inadvertently delete some correct matches.
- Validation by annotated features is limited by the number of annotations in the datasets.
- Most of the missed annotated matches are between features that behave very differently from modeled inter-dataset shifts of RT, MZ, and FI.
- No software version is stated for M2S in the article.

## Evidence

- [abstract] a method to find feature correspondence between two similar LC–MS metabolomics experiments or batches using only the features’ RT, m/z, and FI: "a method to find feature correspondence between two similar LC–MS metabolomics experiments or batches using only the features’ RT, m/z, and FI"
- [abstract] The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox, available at https://github.com/rjdossan/M2S"
- [abstract] In a third example with simulated data with 4755 features per dataset, 99.6% of the matches were correct: "In a third example with simulated data with 4755 features per dataset, 99.6% of the matches were correct"
- [methods] a QC dilution series14 was used to eliminate features whose response to dilution was worse than a threshold of R2 <0.7.: "a QC dilution series14 was used to eliminate features whose response to dilution was worse than a threshold of R2 <0.7."
- [methods] Step 3: Find Poor Matches (Tighten Thresholds): "Step
3: Find Poor Matches (Tighten Thresholds)"
- [discussion] This cluster was removed by setting a tighter upper threshold in the FI domain in the initial candidate matching: "This cluster was
removed by setting a tighter upper threshold in the FI domain in the
initial candidate matching"
- [readme] Matlab package to match untargeted metabolomic features of two LC-MS datasets: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
