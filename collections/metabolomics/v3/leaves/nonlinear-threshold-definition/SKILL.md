---
name: nonlinear-threshold-definition
description: Use when a two-dataset LC-MS metabolomics feature correspondence already
  contains unique matches (post Step 2) but the plain box thresholds in RT, m/z, and
  FI are too tight for features under large local shift trends, so candidate matches
  need to be re-scored and poor matches deleted.
when_to_use_negative:
- Unique matching has not yet been performed (only the initial candidate matching
  exists) — run Steps 1 and 2 first, since this skill tightens thresholds on already-matched
  pairs
- The analyst wants to keep borderline matches that may be biologically valid — the
  article notes this optional step 'may inadvertently delete some correct matches',
  so use it only when deletion is acceptable
- The task is the initial candidate matching with plain RT/m/z/FI box thresholds rather
  than re-defining thresholds from residual trends
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: M2S (Match 2 Sets)
  role: 'MATLAB toolbox whose functions implement Step 3 poor-match detection: residual
    computation against modeled shifts, median+MAD normalization, nonlinear threshold
    application, and match deletion'
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Environment in which the M2S Step 3 functions are executed and the threshold
    redefinition is scripted
  repo: https://github.com/rjdossan/M2S
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:aa9492d2c49d865a14a5335ee4864b4adedbc0697fa948de8635d01f0b410ed2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# nonlinear-threshold-definition

## Summary

Redefines RT, MZ, and log10FI match thresholds as nonlinear functions of the reference/target feature positions by normalizing per-dimension residuals with median + F·MAD and flagging matches far from the modeled inter-dataset shift trends in the M2S toolbox (Step 3 of the feature-correspondence workflow).

## When to use

Use when a two-dataset LC-MS metabolomics feature correspondence already contains unique matches (post Step 2) but the plain box thresholds in RT, m/z, and FI are too tight for features under large local shift trends, so candidate matches need to be re-scored and poor matches deleted.

## When NOT to use

- Unique matching has not yet been performed (only the initial candidate matching exists) — run Steps 1 and 2 first, since this skill tightens thresholds on already-matched pairs
- The analyst wants to keep borderline matches that may be biologically valid — the article notes this optional step 'may inadvertently delete some correct matches', so use it only when deletion is acceptable
- The task is the initial candidate matching with plain RT/m/z/FI box thresholds rather than re-defining thresholds from residual trends

## Inputs

- Unique match table (post Step 2) linking reference and target features by RT, m/z, and FI, e.g. MESA (10427 features) vs Rotterdam (14097 features) after XCMS peak-picking
- Feature RT, m/z, and log10 FI values for both datasets after QC, trimming, and unique correspondence
- Neighbor specification (e.g. 'cross' method with 1% of reference features as neighbors) and per-dimension weights (e.g. [1, 1, 0.2])
- Normalization factor F for MAD (e.g. 3) and threshold method name ('scores')

## Outputs

- Match list with flagged poor matches (matches far from the modeled shift trends)
- Retained unique match list after deletion of poor matches
- Match counts per dataset, e.g. matches_before / matches_deleted / matches_after per dataset
- Retained/expected ratio, e.g. 2682/2717 = 98.71% on the simulated dataset

## How to apply

Run Step 3 of the M2S workflow after the unique-match step: compute each match's residuals relative to the robustly estimated inter-dataset shifts of RT, m/z, and log10 FI using the 'cross' neighbor method with 1% of reference features as neighbors. Normalize residuals per dimension using median + F·MAD (F=3 in the main example) and apply the 'scores' nonlinear threshold method at median + 3·MAD to flag matches far from the shift trends, using weights W_RT,MZ,FI = [1, 1, 0.2] that downweight intensity deviations. Delete flagged poor matches from the unique match lists. On the MESA–Rotterdam pair this retained 4953 of 5365 matches, and on simulated data 2682 of 2728; the retained/expected ratio on Dataset 3 was 98.71% (2682/2717).

## Related tools

- **M2S (Match 2 Sets)** (MATLAB toolbox whose functions implement Step 3 poor-match detection: residual computation against modeled shifts, median+MAD normalization, nonlinear threshold application, and match deletion) — https://github.com/rjdossan/M2S
- **MATLAB** (Environment in which the M2S Step 3 functions are executed and the threshold redefinition is scripted) — https://github.com/rjdossan/M2S

## Examples

```
% M2S Step 3 (MATLAB): recompute thresholds from residual trends and delete flagged poor matches
% (scripted via the M2S toolbox; see https://github.com/rjdossan/M2S for exact call signatures)
```

## Evaluation signals

- Retained match counts match reported values within ±2: Dataset 1 = 4953 of 5365, Dataset 3 = 2682 of 2728
- Retained/expected ratio on Dataset 3 equals 2682/2717 = 98.71%
- Step 3 reduces the match list but keeps a majority of unique matches (4953 retained of 5365 in the main example)
- Flagged poor matches lie far from the modeled RT/mz/log10FI shift trends when visualized in the residual distance plots
- Per-dimension residual normalization uses median + 3·MAD consistently across RT, MZ, and log10FI dimensions

## Limitations

- The decision on whether to apply Step 3 rests with the analyst; it is optional and the method does not prescribe when to use it
- This optional step may inadvertently delete some correct matches
- Threshold selection is subjective, guided by visual inspection of the distance plots resulting from the initial matching
- Validation by annotated features is limited by the number of annotations available in the datasets

## Evidence

- [methods] Step 3 tightens thresholds on already-unique matches: "Step
3: Find Poor Matches (Tighten Thresholds)"
- [discussion] Toolbox provides Step 3 deletion capability: "step 3 to detect and delete poor matches"
- [discussion] Optional step may remove true matches: "this final/optional
step may inadvertently delete some correct matches"
- [discussion] Analyst decides when to apply Step 3: "The decision on when to use step
3 to detect and delete poor matches rests with the analyst"
- [abstract] M2S is the MATLAB toolbox for the workflow: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox"
- [abstract] Main example match count after Step 3: "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
- [methods] Dataset feature counts before matching: "After QC, the MESA dataset contained 10 427
features and the Rotterdam dataset contained 14 097."
