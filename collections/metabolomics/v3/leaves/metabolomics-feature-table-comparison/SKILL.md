---
name: metabolomics-feature-table-comparison
description: Use when two untargeted LC-MS metabolomics feature tables (samples x
  features with RT, m/z, and optionally FI medians) must be matched, and match counts/quality
  should be compared against metabCombiner.
when_to_use_negative:
- The datasets have not yet been peak-picked into feature tables (raw LC-MS data must
  first be processed with XCMS and QC'd)
- The goal is statistical cross-dataset analysis of co-occuring differential features
  rather than feature correspondence matching (metaXCMS-style meta-analysis)
- Features behave very differently from the modeled inter-dataset shifts of RT, m/z,
  and FI (these are the matches M2S most often misses)
tools:
- name: M2S (Match 2 Sets)
  role: 'Primary matching toolbox: threshold matching, shift estimation, and penalty-score
    selection between two feature lists'
  repo: https://github.com/rjdossan/M2S
- name: metabCombiner
  role: Published alternative method run on the same dataset pairs as benchmark comparator
- name: MATLAB
  role: Interactive environment in which the M2S toolbox and its examples run
- name: XCMS
  role: Used to peak-pick each dataset separately into feature tables prior to matching
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:6cd5692bd056b686e05b19e3c1f20c08f308d25d92c9cd24318a99d890d6c57d
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolomics-feature-table-comparison

## Summary

A method for finding correspondence (matching) between untargeted LC-MS metabolomics feature lists using only each feature's retention time (RT), m/z, and optionally feature intensity (FI) medians, implemented in the open-source MATLAB toolbox M2S (Match 2 Sets), and benchmarked against metabCombiner on three dataset pairs.

## When to use

Use when two untargeted LC-MS metabolomics feature tables (samples x features with RT, m/z, and optionally FI medians) must be matched, and match counts/quality should be compared against metabCombiner.

## When NOT to use

- The datasets have not yet been peak-picked into feature tables (raw LC-MS data must first be processed with XCMS and QC'd)
- The goal is statistical cross-dataset analysis of co-occuring differential features rather than feature correspondence matching (metaXCMS-style meta-analysis)
- Features behave very differently from the modeled inter-dataset shifts of RT, m/z, and FI (these are the matches M2S most often misses)

## Inputs

- Two peak-picked LC-MS feature tables (samples x features), each feature characterized by retention time, m/z, and median feature intensity across samples
- Optionally, feature annotations (e.g. Metabolomics Standards Initiative confidence level 2) for validation
- Dataset pairs such as MESA (reference) and Rotterdam (target) cohorts from Examples S4-S6

## Outputs

- Set of uniquely matched feature pairs between the two datasets
- Side-by-side comparison table of M2S vs metabCombiner per dataset pair (match counts and match quality metrics)
- List of deleted poor matches if Step 3 is applied

## How to apply

Peak-pick each dataset separately with XCMS, apply feature QC (eliminate features with dilution-series response R2 < 0.7) and retention-time trimming (e.g. retain only features eluting before 12 min, or between 0.45 and 9.5 min), then summarize each feature by the medians of RT, m/z, and FI across all samples. Run the M2S workflow: Step 1 match all variables within thresholds (initial large windows, ~1 min RT and ~0.025 Da m/z, optionally a tighter upper FI threshold to remove column-bleeding clusters); Step 2 find unique correspondence using neighbor-based shift estimation with residuals and a weighted penalty score; Step 3 (optional, analyst's decision) find and delete poor matches by tightening thresholds. Run metabCombiner on the same dataset pairs per its published algorithm as a comparator. Assemble a side-by-side table per pair with match counts and match quality (e.g. agreement with MSI confidence-level-2 annotated features) to demonstrate M2S's qualitative advantage.

## Related tools

- **M2S (Match 2 Sets)** (Primary matching toolbox: threshold matching, shift estimation, and penalty-score selection between two feature lists) — https://github.com/rjdossan/M2S
- **metabCombiner** (Published alternative method run on the same dataset pairs as benchmark comparator)
- **MATLAB** (Interactive environment in which the M2S toolbox and its examples run)
- **XCMS** (Used to peak-pick each dataset separately into feature tables prior to matching)

## Evaluation signals

- The comparison table contains rows for all three dataset pairs (S4, S5, S6) and both methods, with match counts and quality metrics populated for each row
- Agreement with annotations: in the main example, 4953 features were uniquely matched and 585 (96.8%) of 604 manually annotated features were correct; in the second example, 79 (90.8%) of 87 annotated features were correct
- On simulated data with 4755 features per dataset, at least 99.6% of matches should be correct
- Correctly matched features should be consistent with the modeled inter-dataset shifts of RT, m/z, and FI; deviations flag poor matches for Step 3 removal

## Limitations

- Threshold selection is subjective, guided by visual inspection of the distance plots resulting from the initial matching
- The optional Step 3 (poor-match deletion) is left to the analyst and may inadvertently delete some correct matches
- Validation by annotated features is limited by the number of annotations available in the datasets
- Most missed annotated matches are between features that behave very differently from the modeled inter-dataset shifts of RT, m/z, and FI

## Evidence

- [abstract] the results of matching three other dataset pairs using our method are compared with a published alternative method, metabCombiner, showing the advantages of our approach: "the results of matching three other dataset pairs using our method are compared with a published alternative method, metabCombiner, showing the advantages of our approach"
- [methods] These are used to compare the results of M2S and an existing method, metabCombiner.: "These are used to compare the results of M2S and an existing method, metabCombiner."
- [methods] Step 1: Match All Variables Within Thresholds: "Step 1: Match All Variables Within Thresholds"
- [discussion] guided by visual inspection of the distance plots resulting from the initial matching: "guided by visual inspection of the distance plots resulting from the initial matching"
- [readme] Matlab package to match untargeted metabolomic features of two LC-MS datasets: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
