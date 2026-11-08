---
name: feature-intensity-shift-characterization
description: Use when preparing independently peak-picked XCMS feature tables from
  two LC-MS metabolomics batches (e.g. MESA reference vs Rotterdam target) for feature
  correspondence matching, since the matching algorithm operates on per-feature medians
  of RT, m/z, and FI rather than raw sample-level values.
when_to_use_negative:
- Input is a raw LC-MS data file (e.g. mzML/NetCDF) rather than an already peak-picked,
  QC-filtered feature table — peak-picking and QC (dilution-series R2 < 0.7 removal,
  RT trimming) must be done first with XCMS
- The two datasets were acquired on dissimilar platforms or with different chromatographic
  methods, since the method relies on modeled inter-dataset shifts of RT, MZ, and
  FI and only requires RT, MZ, and optionally FI values
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: M2S (Match 2 Sets)
  role: MATLAB toolbox implementing the full matching workflow (Step 1 all-possible-matches,
    Step 2 unique correspondence, Step 3 poor-match removal) that consumes the median
    RT/MZ/FI characterizations
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Environment in which M2S runs; used to compute per-feature medians and execute
    the threshold-based candidate matching
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Peak-picks each dataset separately to produce the raw feature tables that
    are then QC-filtered and characterized by medians
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:0ae13395f697aa85d1fb814bfe4dae91a7200ff82a0bb4eeaad9c6778e52e7b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  - XCMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Feature intensity shift characterization

## Summary

Characterizes each feature in an untargeted LC-MS metabolomics dataset by computing the median retention time (RT), m/z (MZ), and log10-transformed feature intensity (FI) across all samples, producing the per-feature reference values that the M2S matching workflow uses for cross-dataset correspondence.

## When to use

Use when preparing independently peak-picked XCMS feature tables from two LC-MS metabolomics batches (e.g. MESA reference vs Rotterdam target) for feature correspondence matching, since the matching algorithm operates on per-feature medians of RT, m/z, and FI rather than raw sample-level values.

## When NOT to use

- Input is a raw LC-MS data file (e.g. mzML/NetCDF) rather than an already peak-picked, QC-filtered feature table — peak-picking and QC (dilution-series R2 < 0.7 removal, RT trimming) must be done first with XCMS
- The two datasets were acquired on dissimilar platforms or with different chromatographic methods, since the method relies on modeled inter-dataset shifts of RT, MZ, and FI and only requires RT, MZ, and optionally FI values

## Inputs

- QC'd XCMS peak-picked feature table (samples × features) from the reference dataset (e.g. MESA, 10427 features in Dataset 1)
- QC'd XCMS peak-picked feature table from the target dataset (e.g. Rotterdam, 14097 features in Dataset 1)
- Per-dataset retention-time trimming bounds and candidate-matching thresholds (RT window, MZ tolerance, FI threshold)

## Outputs

- Per-feature characterization table: median RT, median MZ, and median log10(FI) for every feature in each dataset
- Step 1 candidate match counts per dataset pair (e.g. 5426 candidates for Dataset 1 MESA–Rotterdam ESI+)
- Summary table of unique matches and clusters of multiple matches after uniqueness resolution

## How to apply

Load the QC'd feature tables (already passed the QC dilution series R2 ≥ 0.7 filter and retention-time trimming, e.g. features eluting after 12 min retained for Dataset 1, or 0.45–9.5 min for Dataset 2). For each dataset, compute per-feature medians of RT, MZ, and log10(FI) across all samples; this median-based characterization makes each feature a single point in RT–MZ–FI space. These median RT/MZ/FI triples become the coordinates on which the M2S Step 1 all-possible-matches candidate matching is performed using user-defined thresholds (e.g. RT ±1 min, MZ ±0.025 Da, plus an FI threshold for Dataset 1). A tighter upper threshold on the FI domain can subsequently remove clusters of features exhibiting column-bleeding-like behavior. The decision on whether to invoke the optional Step 3 poor-match deletion rests with the analyst, guided by visual inspection of distance plots from the initial matching.

## Related tools

- **M2S (Match 2 Sets)** (MATLAB toolbox implementing the full matching workflow (Step 1 all-possible-matches, Step 2 unique correspondence, Step 3 poor-match removal) that consumes the median RT/MZ/FI characterizations) — https://github.com/rjdossan/M2S
- **MATLAB** (Environment in which M2S runs; used to compute per-feature medians and execute the threshold-based candidate matching) — https://github.com/rjdossan/M2S
- **XCMS** (Peak-picks each dataset separately to produce the raw feature tables that are then QC-filtered and characterized by medians)

## Examples

```
matlab -r "addpath('M2S'); % load QC'd feature tables; medRT = median(RT,2); medMZ = median(MZ,2); medFI = median(log10(FI),2);"
```

## Evaluation signals

- Step 1 candidate match count for Dataset 1 (MESA vs Rotterdam, ESI+) should equal 5426 before uniqueness resolution; comparable Stage-1 counts are recorded for the ESI− (Dataset 2) and simulated (Dataset 3) examples
- Dataset 1 after QC should contain 10427 MESA features and 14097 Rotterdam features; Dataset 2 should contain 6793 MESA and 6315 Rotterdam features
- After Step 2 unique correspondence, Dataset 1 should yield 4953 uniquely matched features, with 585 (96.8%) of 604 manually annotated features correct
- Per-feature medians should be reproducible: re-running the characterization on the same QC'd table must return identical RT/MZ/log10(FI) values, and features failing the dilution-series R2 < 0.7 threshold must be absent

## Limitations

- Threshold selection is subjective, guided by visual inspection of distance plots from the initial matching
- The optional Step 3 poor-match deletion may inadvertently delete some correct matches; its use rests with the analyst
- Validation by annotated features is limited by the number of annotations in the datasets; most missed annotated matches are between features that behave very differently from modeled inter-dataset shifts of RT, MZ, and FI

## Evidence

- [methods] per-feature medians of RT, m/z, and FI across all samples: "the medians of RT, m/z, and FI across all samples were used."
- [methods] Step 1: Match all variables within thresholds: "Step 1: Match All Variables Within Thresholds"
- [methods] QC dilution series threshold R2 < 0.7: "a QC dilution series14 was used to eliminate features whose response to dilution was worse than a threshold of R2 <0.7."
- [discussion] a cluster of features removed by tighter FI threshold (column bleeding): "This cluster was removed by setting a tighter upper threshold in the FI domain in the initial candidate matching"
- [discussion] threshold selection guided by visual inspection: "guided by visual inspection of the distance plots resulting from the initial matching"
