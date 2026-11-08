---
name: lcms-feature-matching
description: Use when integrating two similar untargeted LC–MS metabolomics feature
  tables (e.g. two batches or two cohorts peak-picked with XCMS) that need feature-level
  correspondence based solely on RT, MZ, and FI values.
when_to_use_negative:
- You already have feature-level correspondence (e.g. identical feature IDs across
  batches) or the two datasets were acquired in a single run — re-matching with RT/MZ/FI
  alone would discard existing mapping.
- You need feature-to-metabolite annotation rather than cross-dataset feature matching
  — use annotation tools/databases (LIPID MAPS, METLIN, HMDB, GNPS, MassBank) instead.
- The two LC–MS datasets were acquired with substantially different chromatographic/mass-spectrometric
  methods such that inter-dataset RT and MZ shifts cannot be modeled robustly — the
  method relies on modeling inter-dataset shifts of RT, MZ, and FI.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: M2S (Match 2 Sets)
  role: Primary implementation of the three-step matching method (candidate matching,
    unique correspondence, poor-match detection) for RT/MZ/FI-based feature correspondence.
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Used to peak-pick each dataset separately, producing the feature tables (RT,
    MZ, FI) that serve as input to matching.
- name: MATLAB
  role: Runtime environment for the M2S toolbox.
- name: metabCombiner
  role: Published alternative feature-matching method used as a comparison baseline
    for M2S results on three dataset pairs.
- name: metaXCMS
  role: Earlier meta-analysis software for combining peak lists from multiple datasets;
    context/alternative to M2S.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:93e1df022f63b901a687d10649b32eaed824b3409460953d8d6b82986da227c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  - XCMS
  - metaXCMS
  - metabCombiner
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# lcms-feature-matching

## Summary

Find correspondence between features in two untargeted LC–MS metabolomics datasets using only retention time, m/z, and feature intensity (RT, MZ, FI), via the three-step M2S method (candidate matching, unique correspondence, poor-match deletion). This enables batch combination, discovery validation, and multicohort integration of LC–MS metabolomics data.

## When to use

Use when integrating two similar untargeted LC–MS metabolomics feature tables (e.g. two batches or two cohorts peak-picked with XCMS) that need feature-level correspondence based solely on RT, MZ, and FI values.

## When NOT to use

- You already have feature-level correspondence (e.g. identical feature IDs across batches) or the two datasets were acquired in a single run — re-matching with RT/MZ/FI alone would discard existing mapping.
- You need feature-to-metabolite annotation rather than cross-dataset feature matching — use annotation tools/databases (LIPID MAPS, METLIN, HMDB, GNPS, MassBank) instead.
- The two LC–MS datasets were acquired with substantially different chromatographic/mass-spectrometric methods such that inter-dataset RT and MZ shifts cannot be modeled robustly — the method relies on modeling inter-dataset shifts of RT, MZ, and FI.

## Inputs

- Two untargeted LC–MS metabolomics feature tables (one reference, one target) each with per-feature RT, MZ, and (optionally) FI values across samples, e.g. XCMS peak-picking output
- Per-dataset QC-filtered feature lists (e.g. features surviving a QC dilution-series R2 < 0.7 filter and RT trimming)
- User-specified matching thresholds for RT, MZ, and log10FI (selected via visual inspection of distance plots)

## Outputs

- List of uniquely matched feature pairs (one-to-one correspondence) between reference and target datasets
- Optionally, the poor-match deletion log: count of matches flagged and deleted at Step 3 (e.g. Dataset 1: 4953 of 5365 retained; Dataset 3: 2682 of 2728 retained)
- Distance/diagnostic plots of RT, MZ, and FI residuals for threshold selection and poor-match review

## How to apply

First peak-pick each dataset separately (e.g. with XCMS) and run proper QC — e.g. a QC dilution series eliminating features with R2 < 0.7 response to dilution — plus optional RT trimming (e.g. retaining features eluting before 12 min or between 0.45 and 9.5 min). Characterize each feature by the medians of RT, MZ, and FI across all samples, then run Step 1 to match all variables within thresholds on RT, MZ, and optionally log10FI, and Step 2 to resolve these into unique (many-to-many → one-to-one) correspondences. Optionally apply Step 3 poor-match detection: compute residuals of each match relative to the robustly estimated inter-dataset RT, MZ, and log10FI shifts (using the 'cross' neighbor method with ~1% of reference features as neighbors), normalize residuals per dimension using median + 3 × MAD, and flag matches via a nonlinear threshold method (e.g. 'scores' at median + 3 × MAD, weights W_RT, W_MZ, W_FI = [1, 1, 0.2]) for deletion. Threshold selection is guided by visual inspection of the distance plots resulting from the initial matching; the decision on whether to use Step 3 rests with the analyst. The method is implemented in M2S (Match 2 Sets), a free, open-source MATLAB toolbox available at https://github.com/rjdossan/M2S.

## Related tools

- **M2S (Match 2 Sets)** (Primary implementation of the three-step matching method (candidate matching, unique correspondence, poor-match detection) for RT/MZ/FI-based feature correspondence.) — https://github.com/rjdossan/M2S
- **XCMS** (Used to peak-pick each dataset separately, producing the feature tables (RT, MZ, FI) that serve as input to matching.)
- **MATLAB** (Runtime environment for the M2S toolbox.)
- **metabCombiner** (Published alternative feature-matching method used as a comparison baseline for M2S results on three dataset pairs.)
- **metaXCMS** (Earlier meta-analysis software for combining peak lists from multiple datasets; context/alternative to M2S.)

## Examples

```
matlab -r "addpath(genpath('M2S')); matchFeatures('reference','mesa','target','rotterdam')
```

## Evaluation signals

- Compare uniquely matched counts to the paper's reported values within a small tolerance: main example (MESA–Rotterdam, Dataset 1) 4953 uniquely matched features of 5365 unique matches after Step 3; second example 2324 uniquely matched features; Dataset 3 (simulated, 4755 features/dataset) 99.6% correct matches.
- Validate correctness against annotated features: in the main example, 585 of 604 manually annotated features (96.8%) correctly matched; in the second example, 79 of 87 annotated features (90.8%) correctly matched.
- Check that each retained match is unique — Step 2 must yield one-to-one (not many-to-many) correspondence between reference and target features.
- If Step 3 is applied, verify that flagged poor matches are those whose residuals deviate from the robustly modeled inter-dataset RT/MZ/log10FI shifts (median + 3 × MAD normalization); e.g. for Dataset 3, retained/expected ratio = 2682/2717 = 98.71%.
- Cross-check against metabCombiner on a held-out dataset pair to confirm the method's results remain competitive or better.

## Limitations

- Threshold selection is subjective and guided by visual inspection of the distance plots resulting from the initial matching — there is no objective automatic rule.
- The decision on when to use step 3 to detect and delete poor matches rests with the analyst, and this optional step may inadvertently delete some correct matches.
- Validation by annotated features is limited by the number of annotations in the datasets.
- Most missed annotated matches are between features that behave very differently from the modeled inter-dataset shifts of RT, MZ, and FI (e.g. a cluster of features at different FI from possible column bleeding may require a tighter FI threshold at the initial candidate matching).
- Proper quality control of features prior to matching is essential to avoid poor matches.

## Evidence

- [abstract] a method to find feature correspondence between two similar LC–MS metabolomics experiments or batches using only the features’ RT, m/z, and FI: "a method to find feature correspondence between two similar LC–MS metabolomics experiments or batches using only the features’ RT, m/z, and FI"
- [methods] After QC, the MESA dataset contained 10 427 features and the Rotterdam dataset contained 14 097.: "After QC, the MESA dataset contained 10 427 features and the Rotterdam dataset contained 14 097."
- [methods] a QC dilution series14 was used to eliminate features whose response to dilution was worse than a threshold of R2 <0.7.: "a QC dilution series14 was used to eliminate features whose response to dilution was worse than a threshold of R2 <0.7."
- [methods] Step 1: Match All Variables Within Thresholds: "Step 1: Match All Variables Within Thresholds"
- [methods] Step 2: Find Unique Correspondence: "Step 2: Find Unique Correspondence"
- [methods] Step 3: Find Poor Matches (Tighten Thresholds): "Step 3: Find Poor Matches (Tighten Thresholds)"
- [abstract] 4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct: "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
- [abstract] The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox"
- [abstract] available at https://github.com/rjdossan/M2S: "available at https://github.com/rjdossan/M2S"
- [methods] Both datasets were (separately) peak-picked using XCMS,13: "Both datasets were (separately) peak-picked using XCMS,13"
- [methods] the medians of RT, m/z, and FI across all samples were used.: "the medians of RT, m/z, and FI across all samples were used."
- [discussion] guided by visual inspection of the distance plots resulting from the initial matching: "guided by visual inspection of the distance plots resulting from the initial matching"
- [discussion] The decision on when to use step 3 to detect and delete poor matches rests with the analyst: "The decision on when to use step 3 to detect and delete poor matches rests with the analyst"
- [discussion] this final/optional step may inadvertently delete some correct matches: "this final/optional step may inadvertently delete some correct matches"
- [intro] will aid the integration of untargeted LC–MS metabolomics data in a wide variety of applications such as batch combination, discovery validation, and multicohort integration: "will aid the integration of untargeted LC–MS metabolomics data in a wide variety of applications such as batch combination, discovery validation, and multicohort integration"
- [readme]  Matlab package to match untargeted metabolomic features of two LC-MS datasets: " Matlab package to match untargeted metabolomic features of two LC-MS datasets"
