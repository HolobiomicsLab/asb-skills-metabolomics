---
name: cross-dataset-feature-matching
description: Use when integrating two untargeted LC-MS metabolomics peak tables (e.g.
  MESA reference vs Rotterdam target) that share RT, m/z, and FI dimensions but were
  acquired in separate runs, and you need unique feature correspondence without MS/MS
  or annotation-level matching.
when_to_use_negative:
- When the two datasets are peak lists from the same acquisition run (within-batch
  feature grouping), not inter-batch/inter-study alignment
- When features lack reliable intensity (e.g. pure MS1 presence/absence only) and
  the FI-dimension residual-driven uniqueness resolution is not informative
- When MS/MS spectral matching or known-annotation-based correspondence is available
  and preferred over RT/MZ/FI shift modeling
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: M2S (Match 2 Sets)
  role: Primary MATLAB toolbox implementing the three-step matching (candidate matching,
    cross/circle neighbor shift modeling, unique correspondence, optional poor-match
    deletion)
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Execution environment for M2S; the toolbox is distributed as an open-source
    MATLAB package
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Upstream peak-picking of each dataset separately to produce the sample-by-feature
    tables that M2S consumes
- name: metabCombiner
  role: Published alternative method used as a benchmark comparison in the paper's
    validation on three other dataset pairs
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:3eaae101289d97887ac6ee85cd34483bdb1bdfb1be8b72595bfcf4a0861d64f9
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
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# cross-dataset-feature-matching

## Summary

Match untargeted LC-MS metabolomic features across two similar datasets or batches using only retention time, m/z, and optionally feature intensity (FI), via the M2S three-step procedure (candidate matching, robust inter-dataset shift modeling with neighbor definitions, unique correspondence resolution).

## When to use

Use when integrating two untargeted LC-MS metabolomics peak tables (e.g. MESA reference vs Rotterdam target) that share RT, m/z, and FI dimensions but were acquired in separate runs, and you need unique feature correspondence without MS/MS or annotation-level matching.

## When NOT to use

- When the two datasets are peak lists from the same acquisition run (within-batch feature grouping), not inter-batch/inter-study alignment
- When features lack reliable intensity (e.g. pure MS1 presence/absence only) and the FI-dimension residual-driven uniqueness resolution is not informative
- When MS/MS spectral matching or known-annotation-based correspondence is available and preferred over RT/MZ/FI shift modeling

## Inputs

- Two peak-picked untargeted LC-MS feature tables (reference and target) with per-feature sample intensities
- Per-feature medians of RT, m/z, and log10FI across all samples
- User-defined matching thresholds for RT (e.g. ±1 min), MZ (e.g. ±0.025 Da), and FI
- Choice of neighbor definition for shift modeling: 'cross' or 'circle'
- Optional annotation table (Metabolomics Standards Initiative confidence level 2) for validation

## Outputs

- Candidate match list with per-pair RT, MZ, and FI residuals
- Inter-dataset shift curves (smoothed shift as function of RTref, and MZref under 'circle')
- Unique feature correspondence table (reference feature -> target feature)
- Optional post-Step-3 cleaned match list with poor matches removed
- Validation metrics (e.g. % of annotated features correctly matched)

## How to apply

Peak-pick each dataset separately (e.g. with XCMS) and compute per-feature medians of RT, m/z, and log10FI across all samples; apply quality control such as a QC dilution-series R^2 >= 0.7 filter and retention-time trimming (e.g. only features eluting 0.45–12 min, dataset-dependent). In Step 1, pair every reference feature with every target feature within user-defined thresholds on all three dimensions (paper example: RT within 1 min, MZ within 0.025 Da, plus an FI threshold), yielding candidate matches (e.g. 5426 candidates for ESI+ Dataset 1). In Step 2, model the inter-dataset shift in RT, MZ, and FI using robust local regression over candidate matches — with either the 'cross' neighbor definition (per-dimension k-nearest neighbors giving one smoothed shift per RTref) or the 'circle' definition (normalized Euclidean RT+MZ distance allowing multiple distinct shifts at the same RTref) — and resolve to unique correspondences by minimizing residuals. Optionally, Step 3 tightens thresholds to detect and delete poor matches (e.g. clusters at divergent FI suggesting column bleeding), recognizing this may inadvertently delete correct matches. Gauge matching quality against level-2 MSI annotations (e.g. 96.8% of 604 annotated features correct in Dataset 1) and compare against metabCombiner where feasible.

## Related tools

- **M2S (Match 2 Sets)** (Primary MATLAB toolbox implementing the three-step matching (candidate matching, cross/circle neighbor shift modeling, unique correspondence, optional poor-match deletion)) — https://github.com/rjdossan/M2S
- **MATLAB** (Execution environment for M2S; the toolbox is distributed as an open-source MATLAB package) — https://github.com/rjdossan/M2S
- **XCMS** (Upstream peak-picking of each dataset separately to produce the sample-by-feature tables that M2S consumes)
- **metabCombiner** (Published alternative method used as a benchmark comparison in the paper's validation on three other dataset pairs)

## Examples

```
Clone the toolbox and run the M2S MATLAB example in rjdossan/M2S (e.g. `>> addpath(genpath('M2S')); run('examples/dataset1_run.m')`) to execute the three-step match on the bundled ESI+ MESA–Rotterdam example data.
```

## Evaluation signals

- Step-1 candidate counts match the paper's scale (e.g. 5426 candidates for ESI+ Dataset 1; post-QC feature counts 10427 MESA / 14097 Rotterdam, 6793 MESA / 6315 Rotterdam for Dataset 2)
- Post-Step-2, a single unique correspondence is assigned per reference feature, and residuals from cross and circle neighbor definitions are consistent and small
- Percentage of manually annotated (MSI level-2) features correctly matched reaches ~90–100% (paper: 96.8% of 604 in Dataset 1; 90.8% of 87 in Dataset 2; 99.6% in simulated Dataset 3)
- Missed annotated matches, if any, correspond to features behaving very differently from the modeled inter-dataset RT/MZ/FI shifts
- The 'cross' method yields one smoothed shift per RTref, while the 'circle' method yields multiple distinct shifts at the same RTref (distinguishing behavior per Figure 3)

## Limitations

- Threshold selection is subjective and guided by visual inspection of distance plots from the initial matching; no automatic calibration is given
- The optional Step 3 (poor-match deletion) is left to the analyst's judgment and may inadvertently delete correct matches
- Validation by annotated features is limited by the number of annotations available in the datasets
- No software version is stated for M2S; the method is demonstrated on a single annotated cohort (MESA/Rotterdam) and one simulated dataset

## Evidence

- [abstract] a method to find feature correspondence between two similar LC–MS metabolomics experiments or batches using only the features’ RT, m/z, and FI: "a method to find feature correspondence between two similar LC–MS metabolomics experiments or batches using only the features’ RT, m/z, and FI"
- [methods] the medians of RT, m/z, and FI across all samples were used.: "the medians of RT, m/z, and FI across all samples were used."
- [other] The 'cross' method defines k neighbors separately per dimension to produce a smoothed curve giving one inter-dataset shift for all matches sharing an RTref, while the 'circle' method uses normalized Euclidean distance over RT and MZ (not FI) so matches at the same RTref but different MZref can have distinct shifts.: [evidence span withheld — anchor status: not_found]
- [methods] a QC dilution series was used to eliminate features whose response to dilution was worse than a threshold of R2 <0.7.: "a QC dilution series14 was used to eliminate features whose response to dilution was worse than a threshold of R2 <0.7."
- [abstract] The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox, available at https://github.com/rjdossan/M2S: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox, available at https://github.com/rjdossan/M2S"
- [abstract] 4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct: "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
