---
name: weighted-alignment-score-computation
description: Use when candidate feature pair alignments (FPAs) between two LC-MS feature
  tables already exist — grouped by m/z with a fitted RT spline — and each alignment
  must be ranked and validated by a similarity score.
when_to_use_negative:
- Datasets acquired from excessively disparate chromatographic techniques (e.g. RPLC
  vs HILIC) — elution order is not preserved and the scoring model breaks
- Datasets not acquired in the same ionization mode (positive or negative)
- Abundance values already normalized or transformed in a way that removed ranked
  abundance order information
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: metabCombiner
  role: R package providing m/z grouping, anchor selection, spline fitting, and the
    scoring function used to rank feature pair alignments
  repo: https://github.com/hhabra/metabCombiner
- name: fit_gam
  role: Modified GAM (mgcv) spline-fitting function used to map retention time between
    datasets before scoring
  repo: https://github.com/hhabra/metabCombiner
- name: evaluateParams
  role: Used to guide selection of weight parameters A, B, C on training set compounds
    before scoring
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c03693
    title: '<i>metabCombiner</i>: Paired Untargeted LC-HRMS Metabolomics Feature Matching
      and Concatenation of Disparately Acquired Data Sets'
  source_content_hash: sha256:606659895b90b4784a78d8dcd4fc0147287ea67a2b2430610feb6b743722cbe6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hhabra/metabCombiner
  repo_url: https://github.com/hhabra/metabCombiner
  tools:
  - R
  - evaluateParams
  - fit_gam
  - metabCombiner
  - mgcv
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# weighted-alignment-score-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute metabCombiner's composite similarity score for each candidate feature pair alignment in a pair of disparately-acquired LC-HRMS data sets, ranking matches by m/z difference, mapped retention time difference (after spline fitting), and relative abundance difference with tunable weights A, B, C (e.g. A=100, B=15, C=0.3).

## When to use

Use when candidate feature pair alignments (FPAs) between two LC-MS feature tables already exist — grouped by m/z with a fitted RT spline — and each alignment must be ranked and validated by a similarity score.

## When NOT to use

- Datasets acquired from excessively disparate chromatographic techniques (e.g. RPLC vs HILIC) — elution order is not preserved and the scoring model breaks
- Datasets not acquired in the same ionization mode (positive or negative)
- Abundance values already normalized or transformed in a way that removed ranked abundance order information

## Inputs

- Peak-picked, conventionally aligned LC-MS feature tables (m/z, RT, abundance) from both datasets
- A metabCombiner object with fitted RT mapping spline
- Selected anchor retention time ordered pairs
- Known/annotated compounds for accuracy evaluation

## Outputs

- Ranked feature pair alignments with composite similarity scores
- Weighted per-compound average alignment scores
- Per-alignment and per-compound score table (e.g. calc_scores.csv)

## How to apply

After grouping features by m/z and fitting an RT mapping spline (fit_gam or fit_loess) through selected anchor ordered pairs, call calcScores() on the metabCombiner object, passing weight parameters A, B and C that control the contribution of m/z, mapped RT, and abundance differences respectively (the plasma evaluation used A=100, B=15, C=0.3). Scores are weighted by relative abundance; compute the weighted per-compound average score over known compounds to validate the fit. In the plasma20/plasma30 case, applying metabCombiner's similarity scoring with weights guided by evaluateParams on training-set compounds, most test-set compounds achieved the highest alignment score for all of their variants, with weighted per-compound average scores between 0.85 and 0.9. Select weights on a training subset via evaluateParams before scoring the full data set.

## Related tools

- **metabCombiner** (R package providing m/z grouping, anchor selection, spline fitting, and the scoring function used to rank feature pair alignments) — https://github.com/hhabra/metabCombiner
- **fit_gam** (Modified GAM (mgcv) spline-fitting function used to map retention time between datasets before scoring) — https://github.com/hhabra/metabCombiner
- **evaluateParams** (Used to guide selection of weight parameters A, B, C on training set compounds before scoring)

## Examples

```
mc <- metabCombiner::calcScores(object = mc, A = 100, B = 15, C = 0.3); write.csv(mc@calcScores, file = "calc_scores.csv")
```

## Evaluation signals

- Weighted per-compound matching accuracy of approximately 90% on the plasma data set pair
- Weighted per-compound average alignment scores between 0.85 and 0.9
- Most test-set compounds achieve the highest alignment score for all of their variants
- Mean absolute retention time prediction error of roughly 0.06 min (plasma datasets, different gradient elution)
- All scored alignments have valid m/z group membership and mapped RT within spline fit

## Limitations

- Data sets acquired from HILIC methods are more difficult to align than those by RPLC methods
- Preprocessing errors (incomplete or multipeak integration, low signal-to-noise features, missed peaks) can undermine scoring accuracy
- The preprocessing method and choice of parameters should be considered carefully for each data set
- Weight parameters must be tuned; defaults may not transfer to other datasets or acquisition methods

## Evidence
