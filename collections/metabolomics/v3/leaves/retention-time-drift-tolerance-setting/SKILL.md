---
name: retention-time-drift-tolerance-setting
description: Use when matching two untargeted LC–MS metabolomics feature tables whose
  RT, MZ, and FI values drift between experiments, and you must choose initial match
  thresholds before Step 2 uniqueness resolution.
when_to_use_negative:
- Input is a single feature table with no paired second dataset — tolerance setting
  presumes a reference/target pair (e.g. MESA vs Rotterdam)
- Features have not passed QC (e.g. QC dilution series R2 < 0.7 filter and RT trimming,
  12 min or 0.45–9.5 min windows); poor-quality features will inflate Step 1 candidate
  counts and distort threshold choice
- You need fully automated threshold selection — the article states threshold choice
  is subjective and rests on visual inspection of distance plots
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: M2S (Match 2 Sets)
  role: MATLAB toolbox that performs Step 1 all-possible-matches within the user-set
    RT/MZ/FI tolerances and produces the distance plots used for visual threshold
    selection
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Execution environment for M2S; the matching is run interactively so distance
    plots from the initial matching can be inspected to adjust tolerances
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Peak-picks each dataset separately beforehand so the feature tables fed into
    tolerance-based matching have RT, MZ, and FI medians to compare
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:7abbd593f93a1cc0c84fc37c0aad1da3dd120f6b61f1a12c839b5dbbba84f9ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  - XCMS
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# retention-time-drift-tolerance-setting

## Summary

Set user-defined tolerance windows on retention time (RT), m/z (MZ), and feature intensity (FI) in the M2S Step 1 all-possible-matches stage so that inter-dataset drifts between LC–MS feature tables are accommodated without admitting spurious matches.

## When to use

Use when matching two untargeted LC–MS metabolomics feature tables whose RT, MZ, and FI values drift between experiments, and you must choose initial match thresholds before Step 2 uniqueness resolution.

## When NOT to use

- Input is a single feature table with no paired second dataset — tolerance setting presumes a reference/target pair (e.g. MESA vs Rotterdam)
- Features have not passed QC (e.g. QC dilution series R2 < 0.7 filter and RT trimming, 12 min or 0.45–9.5 min windows); poor-quality features will inflate Step 1 candidate counts and distort threshold choice
- You need fully automated threshold selection — the article states threshold choice is subjective and rests on visual inspection of distance plots

## Inputs

- QC'd peak-picked feature tables (e.g. from XCMS) for a reference dataset and a target dataset, one row per feature with per-feature medians of RT, MZ, and log10FI across all samples
- Annotated features (confidence level 2 per the Metabolomics Standards Initiative) used to inspect tolerance effects on match correctness
- Candidate tolerance values for RT (min), MZ (Da), and FI domains

## Outputs

- A Step 1 candidate-match list for the dataset pair (e.g. 5426 candidates for Dataset 1) annotated with RT, MZ, and FI distances within tolerance
- Distance plots (RT, MZ, FI) from the initial matching used for visual threshold selection
- Counts of unique matches and clusters of multiple matches before Step 2 uniqueness resolution
- A tightened-threshold candidate list when Step 3-style FI tightening is applied

## How to apply

In M2S Step 1 ('Match All Variables Within Thresholds'), pair every reference feature with every target feature falling within user-defined tolerance windows on all three dimensions simultaneously; for Dataset 1 (MESA vs Rotterdam, ESI+) the initial matching used RT ±1 min, MZ ±0.025 Da, plus an FI threshold, yielding 5426 candidate matches. Thresholds are selected subjectively, guided by visual inspection of the distance plots produced by the initial matching rather than by an automated criterion. For Dataset 2 (ESI−, retention-time window 0.45–9.5 min was used at QC), analogous per-dataset tolerances were applied; for the simulated Dataset 3, added systematic and random variability on Dataset 2 required the same tolerance-setting exercise. If a cluster of multiple matches persists (e.g. features at different FI consistent with column bleeding), tighten the upper FI threshold in the initial candidate matching to remove it, and only proceed to Step 2 uniqueness resolution once the tolerance windows are defensible.

## Related tools

- **M2S (Match 2 Sets)** (MATLAB toolbox that performs Step 1 all-possible-matches within the user-set RT/MZ/FI tolerances and produces the distance plots used for visual threshold selection) — https://github.com/rjdossan/M2S
- **MATLAB** (Execution environment for M2S; the matching is run interactively so distance plots from the initial matching can be inspected to adjust tolerances) — https://github.com/rjdossan/M2S
- **XCMS** (Peak-picks each dataset separately beforehand so the feature tables fed into tolerance-based matching have RT, MZ, and FI medians to compare)

## Evaluation signals

- For Dataset 1 (MESA vs Rotterdam, ESI+) with RT ±1 min and MZ ±0.025 Da plus FI threshold, Step 1 must reproduce 5426 candidate matches before uniqueness resolution
- After Step 2, uniquely matched features should approach the published figure (4953 uniquely matched on Dataset 1), indicating tolerances were neither too loose nor too tight
- Of 604 manually annotated matches, ≥96.8% (585) should be correct — excess annotation errors suggest tolerances admit spurious pairs
- No residual cluster of multiple matches differing mainly in FI (possible column bleeding) should survive after tightening the FI upper threshold
- Analogous Stage-1 candidate counts for Dataset 2 and the simulated Dataset 3 should be reproducible within the SI-recorded ranges

## Limitations

- Threshold selection is subjective, guided by visual inspection of the distance plots resulting from the initial matching
- The decision on when to use step 3 to detect and delete poor matches rests with the analyst
- This final/optional step may inadvertently delete some correct matches
- Validation by annotated features is limited by the number of annotations in the datasets
- Most of the missed annotated matches are between features that behave very differently from modeled inter-dataset shifts of RT, MZ, and FI

## Evidence

- [methods] Step 1 tolerances on Dataset 1: [evidence span withheld — anchor status: not_found]
- [results] Step 1 candidate count: [evidence span withheld — anchor status: not_found]
- [discussion] Threshold selection is subjective: [evidence span withheld — anchor status: not_found]
- [methods] Step 1 heading: "Step 1: Match All Variables Within Thresholds"
- [methods] QC dilution series filter: [evidence span withheld — anchor status: not_found]
- [discussion] FI tightening to remove bleeding cluster: "This cluster was removed by setting a tighter upper threshold in the FI domain in the initial candidate matching"
- [readme] M2S toolbox availability: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
