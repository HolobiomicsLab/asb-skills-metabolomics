---
name: mz-matching-threshold-tuning
description: Use when merging two peak-picked untargeted LC–MS metabolomics feature
  tables (e.g., from XCMS) where datasets were processed identically but run in different
  batches or cohorts, and you must decide RT, MZ, and FI tolerance windows before
  aligning features.
when_to_use_negative:
- The two datasets come from different experimental designs or different analytical
  methods — the article states M2S requires datasets 'processed in the same way' with
  matching feature tables
- You need metabolite-to-metabolite annotation rather than dataset-to-dataset feature
  correspondence — use annotation tools (LIPID MAPS, METLIN, HMDB, GNPS, MassBank)
  instead
- Feature tables have not been QC'd or RT-trimmed — the article warns that improper
  QC prior to matching leads to poor matches
tools:
- name: M2S (Match 2 Sets)
  role: The MATLAB toolbox that implements the three-step threshold-based matching
    (Step 1 all-possible-matches, Step 2 unique correspondence, Step 3 delete poor
    matches) and produces the distance plots used for threshold tuning
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Environment in which M2S is run; provides interactive, graphically capable
    displays for inspecting distance plots during threshold selection
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Used upstream to peak-pick each dataset separately and produce the feature
    tables whose RT, MZ, and FI are then matched
- name: metabCombiner
  role: Alternative published method whose matching results are compared against M2S
    on three other dataset pairs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:3d5e861dc714fbc25283a9e58b4b777718ef070b5c83a1c2ebcb7b2bd63e8bc7
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mz-matching-threshold-tuning

## Summary

Tunes retention-time, m/z, and (optionally) feature-intensity thresholds so that a reference feature from one LC–MS metabolomics dataset is matched to the corresponding target feature in another dataset using the M2S method. Thresholds are set via visual inspection of distance plots from the initial matching, then tightened in a final step to remove poor matches.

## When to use

Use when merging two peak-picked untargeted LC–MS metabolomics feature tables (e.g., from XCMS) where datasets were processed identically but run in different batches or cohorts, and you must decide RT, MZ, and FI tolerance windows before aligning features.

## When NOT to use

- The two datasets come from different experimental designs or different analytical methods — the article states M2S requires datasets 'processed in the same way' with matching feature tables
- You need metabolite-to-metabolite annotation rather than dataset-to-dataset feature correspondence — use annotation tools (LIPID MAPS, METLIN, HMDB, GNPS, MassBank) instead
- Feature tables have not been QC'd or RT-trimmed — the article warns that improper QC prior to matching leads to poor matches

## Inputs

- Two separately peak-picked LC–MS feature tables (e.g., XCMS output) with per-feature RT, m/z, and FI values, processed identically in both datasets (reference and target)
- Median RT, m/z, and FI per feature across all samples in each dataset
- User-defined matching thresholds for RT, m/z, and (optionally) FI (e.g., RT ±1 min, MZ ±0.025 Da)
- Optional QC'd feature tables (QC dilution-series filtered, R2 < 0.7 removed; RT-trimming applied)

## Outputs

- Matched feature correspondence table (unique matches between reference and target features)
- Initial candidate-match distance plots (per threshold dimension) for visual threshold tuning
- Counts of candidate matches, unique matches, and clusters of multiple matches
- List of poor matches flagged for deletion (if Step 3 is applied)

## How to apply

Load the two XCMS peak-picked feature tables and characterize each feature by the medians of RT, m/z, and FI across all samples (e.g., Dataset 1: 10,427 MESA features vs 14,097 Rotterdam features). Apply the Step 1 'all possible matches' stage by pairing every reference feature with every target feature that falls within user-defined thresholds on all three dimensions — for the MESA/Rotterdam Dataset 1 example, RT ±1 min and MZ ±0.025 Da plus an FI threshold. Threshold selection is subjective and guided by visual inspection of the distance plots produced by this initial matching; the M2S MATLAB toolbox provides these plots as decision aids. Decide whether to run Step 3 (delete poor matches via tightened thresholds); this step may inadvertently delete some correct matches, so the decision rests with the analyst. Re-run matching with adjusted windows (e.g., a tighter upper FI threshold removed a cluster attributed to possible column bleeding). Proper QC of features (e.g., a QC dilution series eliminating features with dilution-response R2 < 0.7, and RT trimming) prior to matching avoids poor matches.

## Related tools

- **M2S (Match 2 Sets)** (The MATLAB toolbox that implements the three-step threshold-based matching (Step 1 all-possible-matches, Step 2 unique correspondence, Step 3 delete poor matches) and produces the distance plots used for threshold tuning) — https://github.com/rjdossan/M2S
- **MATLAB** (Environment in which M2S is run; provides interactive, graphically capable displays for inspecting distance plots during threshold selection) — https://github.com/rjdossan/M2S
- **XCMS** (Used upstream to peak-pick each dataset separately and produce the feature tables whose RT, MZ, and FI are then matched)
- **metabCombiner** (Alternative published method whose matching results are compared against M2S on three other dataset pairs)

## Evaluation signals

- For Dataset 1 (MESA vs Rotterdam, ESI+), the Step 1 all-possible-matches stage should yield 5426 candidate matches before uniqueness resolution
- After Step 2, 4953 features should be uniquely matched, and of 604 manually annotated features, 585 (96.8%) should be correct
- Annotated-feature check: for the second example, 2324 unique matches with 79 of 87 annotated features correct (90.8%)
- On the simulated Dataset 3 (4755 features per dataset), 99.6% of matches should be correct
- Distance plots from the initial matching should show the inter-dataset shifts of RT, MZ, and FI; missed annotated matches should predominantly be features that behave very differently from these modeled shifts

## Limitations

- Threshold selection is subjective, guided by visual inspection of the distance plots resulting from the initial matching
- The decision on whether to use Step 3 (poor-match deletion) rests with the analyst and is not automatic
- The optional Step 3 may inadvertently delete some correct matches
- Validation by annotated features is limited by the number of annotations in the datasets
- M2S requires the two datasets to be processed in the same way (same peak-picking, same QC)

## Evidence

- [methods] Step 1: Match All Variables Within Thresholds: "Step 1: Match All Variables Within Thresholds"
- [discussion] Threshold selection is subjective: "guided by
visual inspection of the distance plots resulting from the initial
matching"
- [discussion] Step 3 optional deletion caveat: "this final/optional
step may inadvertently delete some correct matches"
- [discussion] Proper QC avoids poor matches: "Proper
quality control of features prior to matching avoids poor matches"
- [methods] Dataset 1 feature counts after QC: "After QC, the MESA dataset contained 10 427
features and the Rotterdam dataset contained 14 097."
- [readme] M2S toolbox description: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
- [abstract] Unique match validation result: "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
