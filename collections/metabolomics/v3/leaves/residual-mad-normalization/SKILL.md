---
name: residual-mad-normalization
description: 'Use when matching two untargeted LC–MS metabolomics feature tables:
  residuals from an estimated inter-dataset RT/mz/FI shift must be normalized robustly
  (median + F×MAD) before scoring or threshold tightening.'
when_to_use_negative:
- The datasets are already harmonized or the shift is known a priori — residual normalization
  would add noise rather than adapt thresholds.
- The comparison is between a feature table and a database annotation (e.g. LIPID
  MAPS, HMDB, METLIN) rather than two experimental LC–MS feature lists; M2S matching
  is designed for dataset-to-dataset correspondence.
- Feature intensities are absent or unreliable and you cannot compute log10FI residuals,
  yet FI-domain thresholds are required — MAD normalization of FI residuals needs
  valid FI medians.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: M2S (Match 2 Sets)
  role: MATLAB toolbox implementing the neighbor-based shift estimation, median +
    3×MAD residual normalization, and Step 3 nonlinear threshold redefinition for
    poor-match detection
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Runtime environment in which the M2S residual normalization and scoring functions
    are executed
  repo: https://github.com/rjdossan/M2S
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:f19bb27596010872bc31660206ba901dd381c2ec034e4106c081129d4685d1e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  - XCMS
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Residual MAD normalization

## Summary

Robustly normalizes the residuals of each candidate feature match (in RT, m/z, and log10FI) relative to the expected inter-dataset shift, using median + F×MAD scaling, so that a weighted penalization score can select unique correspondences and flag poor matches.

## When to use

Use when matching two untargeted LC–MS metabolomics feature tables: residuals from an estimated inter-dataset RT/mz/FI shift must be normalized robustly (median + F×MAD) before scoring or threshold tightening.

## When NOT to use

- The datasets are already harmonized or the shift is known a priori — residual normalization would add noise rather than adapt thresholds.
- The comparison is between a feature table and a database annotation (e.g. LIPID MAPS, HMDB, METLIN) rather than two experimental LC–MS feature lists; M2S matching is designed for dataset-to-dataset correspondence.
- Feature intensities are absent or unreliable and you cannot compute log10FI residuals, yet FI-domain thresholds are required — MAD normalization of FI residuals needs valid FI medians.

## Inputs

- Post-QC feature tables (medians of RT, m/z, FI per feature) for a reference dataset (e.g. MESA, 10427 features) and a target dataset (e.g. Rotterdam, 14097 features)
- Candidate match lists from Step 1 or unique matches from Step 2
- Neighbor definitions ('cross' method, 1% of reference features)

## Outputs

- Normalized residuals in RT, MZ, and log10FI per candidate match
- Weighted penalization score per candidate match
- Redefined nonlinear RT/MZ/log10FI thresholds (median + F×MAD) for poor-match detection in Step 3
- Updated unique match lists (e.g. Dataset 1: 4953 from 5365; Dataset 3: 2682 from 2728)

## How to apply

Define neighbors with the 'cross' method using 1% of the number of reference features, then estimate the expected inter-dataset shift in RT, MZ, and log10FI for each match. Compute residuals of each candidate match against that shift and normalize each dimension using median + 3×MAD, which resists outliers that would distort ordinary z-score normalization. Combine normalized residuals into a weighted penalization score with weights W_RT,MZ,FI = [1, 1, 0.2], applied both to select the best unique match per cluster (Step 2) and, in Step 3, to re-derive nonlinear RT, MZ, and log10FI thresholds from median + F×MAD of the residuals for poor-match detection. The MAD normalization is what lets threshold redefinition adapt to the actual residual spread of each dataset rather than fixed tolerances.

## Related tools

- **M2S (Match 2 Sets)** (MATLAB toolbox implementing the neighbor-based shift estimation, median + 3×MAD residual normalization, and Step 3 nonlinear threshold redefinition for poor-match detection) — https://github.com/rjdossan/M2S
- **MATLAB** (Runtime environment in which the M2S residual normalization and scoring functions are executed) — https://github.com/rjdossan/M2S

## Examples

```
matlab -r "M2S('cross', 0.01, 'scores', 3, [1 1 0.2])"
```

## Evaluation signals

- Retained match counts after Step 3 must reproduce reported values within ±2 matches (Dataset 1: 4953 of 5365; Dataset 3: 2682 of 2728).
- The retained/expected ratio for Dataset 3 should equal 2682/2717 ≈ 98.71%.
- Normalized residual distributions per dimension should be centered near the median with MAD-scaled spread, not dominated by outliers.
- Poor matches flagged via the median + F×MAD 'scores' threshold method should correspond to features far from the modeled inter-dataset RT/MZ/FI shift trends.
- Correctly matched annotated features (Dataset 1: 585/604, 96.8%; Dataset 2: 79/87, 90.8%) should largely survive Step 3 deletion.

## Limitations

- Threshold selection via MAD scaling is partly subjective and guided by visual inspection of distance plots from the initial matching.
- The decision on whether to apply Step 3 poor-match deletion rests with the analyst, and this optional step may inadvertently delete some correct matches.
- MAD normalization assumes residuals follow a roughly symmetric distribution around the expected shift; features that behave very differently from the modeled shifts can be missed or over-flagged.

## Evidence

- [methods] residuals median + F·MAD normalization: [evidence span withheld — anchor status: not_found]
- [methods] median + 3 × MAD: [evidence span withheld — anchor status: not_found]
- [methods] weights W_RT,MZ,FI = [1, 1, 0.2]: [evidence span withheld — anchor status: not_found]
- [methods] Dataset 1 after QC contained 10427 features in MESA and 14097 in Rotterdam: "After QC, the MESA dataset contained 10 427 features and the Rotterdam dataset contained 14 097."
- [discussion] Optional step 3 may delete correct matches: "this final/optional step may inadvertently delete some correct matches"
