---
name: lc-ms-cross-dataset-feature-matching
description: Use when two separately peak-picked LC–MS metabolomics feature tables
  (samples × features with RT, m/z, FI) must be combined or compared — e.g., batch
  combination, discovery validation, or multicohort integration.
when_to_use_negative:
- The input is raw LC–MS data rather than separately peak-picked feature tables with
  RT, m/z, and FI per feature (peak-picking, e.g. with XCMS, must be done first).
- The two datasets are not similar LC–MS experiments or batches of the same type of
  samples — the method models inter-dataset shifts in RT, MZ, and FI and assumes features
  behave consistently with those shifts.
- Features have not undergone QC or RT trimming — the paper notes that improper QC
  prior to matching leads to poor matches.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: M2S (Match 2 Sets)
  role: Free, open-source MATLAB toolbox that implements the three-step matching workflow
    (threshold matching, unique correspondence, poor-match deletion)
  repo: https://github.com/rjdossan/M2S
- name: metabCombiner
  role: 'Published alternative method used as a comparator: the results of matching
    three dataset pairs with M2S are compared against metabCombiner to show M2S''s
    advantages'
- name: XCMS
  role: Used to peak-pick each dataset separately, producing the samples × features
    input tables
- name: MATLAB
  role: Environment in which M2S runs; the toolbox is graphically capable and ships
    with examples
  repo: https://github.com/rjdossan/M2S
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:82fe8a22262c17c4081a64de4e0ff4bac4f53757ab5cfc71dd4a032703430f22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S
  - M2S (Match 2 Sets)
  - MATLAB
  - XCMS
  - metabCombiner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# lc-ms-cross-dataset-feature-matching

## Summary

Method for finding feature correspondence between two untargeted LC–MS metabolomics experiments or batches using only features' retention time (RT), m/z (MZ), and optionally feature intensity (FI) values, implemented in the M2S (Match 2 Sets) MATLAB toolbox.

## When to use

Use when two separately peak-picked LC–MS metabolomics feature tables (samples × features with RT, m/z, FI) must be combined or compared — e.g., batch combination, discovery validation, or multicohort integration.

## When NOT to use

- The input is raw LC–MS data rather than separately peak-picked feature tables with RT, m/z, and FI per feature (peak-picking, e.g. with XCMS, must be done first).
- The two datasets are not similar LC–MS experiments or batches of the same type of samples — the method models inter-dataset shifts in RT, MZ, and FI and assumes features behave consistently with those shifts.
- Features have not undergone QC or RT trimming — the paper notes that improper QC prior to matching leads to poor matches.

## Inputs

- Two preprocessed untargeted LC–MS metabolomics feature tables (samples × features), one reference and one target, each with per-feature retention time (RT), m/z (MZ), and feature intensity (FI) values
- Optionally, a list of annotated features (e.g., METLIN/HMDB/LIPID MAPS/GNPS/MassBank confidence level 2 annotations) for match-quality validation

## Outputs

- A set of uniquely matched feature pairs between the reference and target datasets
- A side-by-side comparison table of match counts and match-quality metrics (e.g., % of annotated matches that are correct) versus alternative methods such as metabCombiner

## How to apply

Peak-pick each dataset separately (the paper used XCMS), apply feature QC (e.g., a QC dilution series removing features with dilution-response R2 < 0.7) and RT trimming (e.g., retain only features eluting before 12 min, or between 0.45 and 9.5 min for some datasets), then characterize each feature by the medians of RT, m/z, and FI across all samples. Run the three-step M2S workflow: Step 1, match all variables within initial thresholds (e.g., large RT windows of ~1 min and m/z windows of ~0.025 Da, guided by visual inspection of distance plots); Step 2, find unique correspondence; Step 3 (optional, analyst's decision), detect and delete poor matches by tightening thresholds, which may also remove clusters at different FI such as column-bleeding features. Assess match quality against independently annotated features (Metabolomics Standards Initiative confidence level 2) and compare against alternative combiners such as metabCombiner on the same dataset pairs.

## Related tools

- **M2S (Match 2 Sets)** (Free, open-source MATLAB toolbox that implements the three-step matching workflow (threshold matching, unique correspondence, poor-match deletion)) — https://github.com/rjdossan/M2S
- **metabCombiner** (Published alternative method used as a comparator: the results of matching three dataset pairs with M2S are compared against metabCombiner to show M2S's advantages)
- **XCMS** (Used to peak-pick each dataset separately, producing the samples × features input tables)
- **MATLAB** (Environment in which M2S runs; the toolbox is graphically capable and ships with examples) — https://github.com/rjdossan/M2S

## Evaluation signals

- For the main MESA (reference, 2656 samples, 10427 features after QC) vs Rotterdam (target, 1057 samples) example, expect ~4953 uniquely matched features, with ~585 (96.8%) of 604 manually annotated features matched correctly.
- For the second example (MESA 6793 features vs Rotterdam 6315 after QC), expect ~2324 uniquely matched features with 79 (90.8%) of 87 annotated features correctly matched.
- On simulated data with 4755 features per dataset, expect ~99.6% of matches to be correct.
- The optional step 3 should reduce poor matches without removing correct ones — most missed annotated matches should be features that behave very differently from the modeled inter-dataset shifts of RT, MZ, and FI.
- A comparison table against metabCombiner should contain rows for all tested dataset pairs with match counts and quality metrics populated, reproducing M2S's qualitative advantage.

## Limitations

- Initial threshold selection is subjective, guided by visual inspection of the distance plots resulting from the initial matching.
- The decision on whether to use step 3 (poor-match deletion) rests with the analyst, and this optional step may inadvertently delete some correct matches.
- Validation by annotated features is limited by the number of annotations available in the datasets.
- Most missed annotated matches are between features that behave very differently from the modeled inter-dataset shifts of RT, MZ, and FI.
- No software version is stated for M2S in the article.

## Evidence

- [abstract] a method to find feature correspondence between two similar LC–MS metabolomics experiments or batches using only the features’ RT, m/z, and FI: "a method to find feature correspondence between two similar LC–MS metabolomics experiments or batches using only the features’ RT, m/z, and FI"
- [abstract] The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox"
- [methods] QC dilution series threshold: "a QC dilution
series14 was used to eliminate features
whose response to dilution was worse than a threshold of R2 <0.7."
- [methods] Step 1 thresholds: "Step 1: Match All Variables Within Thresholds"
- [methods] feature characterization by medians: "the medians of RT, m/z, and FI across all samples were used."
- [discussion] step 3 decision rests with the analyst: "The decision on when to use step
3 to detect and delete poor matches rests with the analyst"
- [abstract] comparison with metabCombiner: "the results of matching three other dataset pairs using our method are compared with a published alternative method, metabCombiner, showing the advantages of our approach"
- [readme] README purpose statement: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
