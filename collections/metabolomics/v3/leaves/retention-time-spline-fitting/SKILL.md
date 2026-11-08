---
name: retention-time-spline-fitting
description: Use when a pair of untargeted LC-MS metabolomics data sets acquired under
  different gradient elution methods must be combined and RT drift must be corrected
  before ranking feature pair alignments by mapped retention time.
when_to_use_negative:
- The two data sets were acquired with excessively disparate chromatographic techniques
  (e.g. RPLC vs HILIC) where elution order of metabolites is not mostly preserved
  — metabCombiner states these are not suitable
- The feature pair alignments have already been scored and the combined table reduced;
  spline fitting is only used for RT mapping before scoring
- The data sets are from different ionization modes or the samples are not biologically
  similar — prerequisites for the metabCombiner workflow that the spline serves
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metabCombiner
  role: Provides selectAnchors for identifying RT ordered pairs and fit_gam/fit_loess
    for spline fitting; the workflow is implemented in the metabCombiner R package
  repo: https://github.com/hhabra/metabCombiner
- name: fit_gam
  role: Modified form of mgcv's gam used to fit the RT mapping spline; default family
    'scat' (scaled t) for heavy-tailed RT data
  repo: https://github.com/hhabra/metabCombiner
- name: fit_loess
  role: Alternative loess-based spline fitter for the same RT mapping task
  repo: https://github.com/hhabra/metabCombiner
- name: mgcv
  role: R package underlying fit_gam; the article notes fit_gam is a modified form
    of the gam function implemented in mgcv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c03693
    title: '<i>metabCombiner</i>: Paired Untargeted LC-HRMS Metabolomics Feature Matching
      and Concatenation of Disparately Acquired Data Sets'
  source_content_hash: sha256:93a59beb522480a49ac0334e350921dbd6e7856ec42fa9b714ed8c530190115e
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
  - GAM
  - R
  - fit_gam
  - fit_loess
  - metabCombiner
  - mgcv
  - selectAnchors
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6086956521739131
n_claims: 23
---

# retention-time-spline-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits a retention-time mapping spline (GAM via fit_gam, scaled-t family, or loess via fit_loess) through a set of selected retention time ordered pairs to map RTs between two disparately acquired untargeted LC-MS metabolomics data sets, correcting for retention time drift before feature alignment scoring.

## When to use

Use when a pair of untargeted LC-MS metabolomics data sets acquired under different gradient elution methods must be combined and RT drift must be corrected before ranking feature pair alignments by mapped retention time.

## When NOT to use

- The two data sets were acquired with excessively disparate chromatographic techniques (e.g. RPLC vs HILIC) where elution order of metabolites is not mostly preserved — metabCombiner states these are not suitable
- The feature pair alignments have already been scored and the combined table reduced; spline fitting is only used for RT mapping before scoring
- The data sets are from different ionization modes or the samples are not biologically similar — prerequisites for the metabCombiner workflow that the spline serves

## Inputs

- metabCombiner anchor table of retention-time ordered pairs produced by selectAnchors from two peak-picked, conventionally aligned untargeted LC-MS data sets (e.g. plasma20 and plasma30)
- known shared compounds (e.g. 68 metabolites in the plasma example) split into training and test anchor sets

## Outputs

- Fitted retention-time mapping spline object (from fit_gam or fit_loess)
- Mapped retention-time values used to rank feature pair alignments by m/z, mapped RT, and relative abundance similarity
- Mean absolute RT prediction error and per-compound RT residuals for held-out test anchors (≈0.06 min mean, >50/68 within 0.1 min on the plasma pair)

## How to apply

First run metabCombiner's selectAnchors to identify retention-time ordered pairs (anchors) whose RT ordering is consistent across the two data sets; the plasma20/plasma30 example uses 68 known shared compounds as anchors. Fit the spline with fit_gam (a modified form of mgcv's gam) — metabCombiner's default 'family' argument is set to scat (scaled t-family for heavy-tailed data), which helps eliminate the influence of outlier anchor points. Alternatively, fit_loess provides a loess-based spline for the same purpose. The fitted spline maps retention times from the second data set (e.g. plasma30, the longer gradient) onto the first (e.g. plasma20), and is then used to add mapped RT values to feature pair alignments during scoring. Validate by predicting RTs for held-out test anchors: the plasma benchmark reports a mean absolute RT prediction error of roughly 0.06 min with >50 of 68 test compounds predicted within 0.1 min.

## Related tools

- **metabCombiner** (Provides selectAnchors for identifying RT ordered pairs and fit_gam/fit_loess for spline fitting; the workflow is implemented in the metabCombiner R package) — https://github.com/hhabra/metabCombiner
- **fit_gam** (Modified form of mgcv's gam used to fit the RT mapping spline; default family 'scat' (scaled t) for heavy-tailed RT data) — https://github.com/hhabra/metabCombiner
- **fit_loess** (Alternative loess-based spline fitter for the same RT mapping task) — https://github.com/hhabra/metabCombiner
- **mgcv** (R package underlying fit_gam; the article notes fit_gam is a modified form of the gam function implemented in mgcv)

## Examples

```
rt.fit <- fit_gam(anchorData, family="scat", method=1, factor=8, spans="adaptive")
```

## Evaluation signals

- Predict RTs for held-out test anchors and check that mean absolute prediction error is roughly 0.06 min on the plasma20/plasma30 pair
- Verify that more than 50 of 68 test compounds are predicted within 0.1 min on the plasma benchmark
- Confirm that semisupervised (known-metabolite) anchors give a slight RT error advantage over unsupervised anchor selection
- Check that mapped RT values are assigned to feature pair alignments before the scoring step that ranks alignments by m/z, mapped RT, and relative abundance similarity
- Inspect the fitted spline for unfiltered anchor-identity outliers — metabCombiner relies on the scaled-t family of fit_gam to damp their influence

## Limitations

- Anchor-identity outliers must not be filtered during selectAnchors if the paper's protocol is followed; the scaled-t family in fit_gam is the mechanism that down-weights them
- Data sets acquired from HILIC methods are more difficult to align than those by RPLC methods, which raises RT-mapping error
- The paper does not state the version of the metabCombiner R package used; the missing signals section flags this as an unstated parameter
- Preprocessing errors such as incomplete or multipeak integration, low signal-to-noise features, and missed peaks propagate into anchor quality and thus spline accuracy

## Evidence

- [readme] Installation of metabCombiner from GitHub for R ≥4.0: [evidence span withheld — anchor status: external_doc]
