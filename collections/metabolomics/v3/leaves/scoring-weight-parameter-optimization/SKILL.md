---
name: scoring-weight-parameter-optimization
description: Use when tuning metabCombiner's similarity-scoring weights (A for m/z,
  B for mapped retention time, C for abundance) before finalizing the combined feature
  table, using a training table of identified compounds shared between two datasets.
when_to_use_negative:
- Input datasets were acquired with different chromatographic techniques such as RPLC
  vs HILIC, where elution order is not preserved.
- Input datasets contain normalized or transformed abundances that remove ranked abundance-order
  information.
- The datasets have not yet been filtered, grouped by m/z, and spline-fitted for retention
  time mapping, i.e. the FPA candidates are not yet available.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: metabCombiner
  role: R package that implements the evaluateParams() objective-function method for
    selecting the A, B, C weight parameters of the feature pair alignment scoring
    function.
  repo: https://github.com/hhabra/metabCombiner
- name: R
  role: Runtime environment in which the metabCombiner package is installed and the
    weight-grid sweep and objective-function evaluation are executed.
  repo: https://github.com/hhabra/metabCombiner
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c03693
    title: '<i>metabCombiner</i>: Paired Untargeted LC-HRMS Metabolomics Feature Matching
      and Concatenation of Disparately Acquired Data Sets'
  source_content_hash: sha256:7a004bdf413566fcc4c0fa623eaa6e8f9ee25d37f458d995e495891a7aacb8ae
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
  - metabCombiner
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# scoring-weight-parameter-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Optimize the metabCombiner weighted feature pair alignment (FPA) score parameters A, B, C by sweeping a weight grid against a training set of shared identified compounds, maximizing the positive score difference between true and misaligned pairings.

## When to use

Use when tuning metabCombiner's similarity-scoring weights (A for m/z, B for mapped retention time, C for abundance) before finalizing the combined feature table, using a training table of identified compounds shared between two datasets.

## When NOT to use

- Input datasets were acquired with different chromatographic techniques such as RPLC vs HILIC, where elution order is not preserved.
- Input datasets contain normalized or transformed abundances that remove ranked abundance-order information.
- The datasets have not yet been filtered, grouped by m/z, and spline-fitted for retention time mapping, i.e. the FPA candidates are not yet available.

## Inputs

- Paired peak-picked LC-MS metabolomics datasets (e.g. plasma20 and plasma30) shipped with the metabCombiner R package
- Training-set table of identified compounds shared between the two datasets, with m/z, retention-time, and abundance values
- Candidate grid of weight triples (A, B, C) for the metabCombiner similarity score

## Outputs

- Full grid of separation scores (positive difference between true and misaligned pairings) for each candidate (A, B, C) triple
- Identified optimal weight region (A, B, C) maximizing the separation between true and misaligned FPAs
- Final selected weight values for the metabCombiner scoring function

## How to apply

Load the paired plasma datasets (plasma20 and plasma30) shipped with the metabCombiner R package and read the training-set table of identified compounds with their m/z, retention-time, and abundance values. For each candidate weight triple (A, B, C) on the grid, compute the weighted FPA score for each training compound's true pairing and for its candidate misaligned pairings, ranking alignments by m/z, mapped retention time, and relative abundance similarity. For each triple, calculate the positive score difference between the true FPAs of shared identified compounds and their misaligned pairings. Sweep the grid to identify the (A, B, C) region maximizing this separation, comparing against the vignette's reported optimum (A≈50–70, B≈14–15, C≈0.3–0.4) and the paper's chosen values (A=100, B=15, C=0.3). Write the full grid of separation scores and the identified optimal weight region to a results table. Validation: verify the recovered optimal region contains the reported vignette region and that the paper's defaults yield a near-maximal separation among training compounds.

## Related tools

- **metabCombiner** (R package that implements the evaluateParams() objective-function method for selecting the A, B, C weight parameters of the feature pair alignment scoring function.) — https://github.com/hhabra/metabCombiner
- **R** (Runtime environment in which the metabCombiner package is installed and the weight-grid sweep and objective-function evaluation are executed.) — https://github.com/hhabra/metabCombiner

## Examples

```
library(metabCombiner); comb <- metabCombine(input1, input2, method="rt", fit="gam")
```

## Evaluation signals

- The recovered optimal (A, B, C) region must contain the vignette's reported region (A≈50–70, B≈14–15, C≈0.3–0.4).
- The paper's chosen values (A=100, B=15, C=0.3) must yield a near-maximal separation score when evaluated on the training compounds.
- The separation score (positive difference between true and misaligned FPAs) must be maximized over the swept grid, not merely positive.
- The final combined feature table produced with the selected weights should achieve the paper's reported weighted per-compound matching accuracy of approximately 90% on the plasma dataset pair.
- Retention-time prediction error on the training compounds should be near the paper's reported mean absolute error of roughly 0.06 min.

## Limitations

- Weight optimization is only meaningful on a training set of shared identified compounds; without such a table, the objective function cannot distinguish true from misaligned pairings.
- HILIC-acquired datasets are more difficult to align than RPLC datasets, which can reduce the robustness of the recovered optimal weight region.
- The paper does not state the specific version of the metabCombiner R package used, so exact parameter defaults may differ across versions.
- Preprocessing errors such as incomplete or multipeak integration, low signal-to-noise features, and missed peaks can distort the training-set pairing and therefore the recovered weights.

## Evidence

- [methods] metabCombiner selects A, B, C weights via the evaluateParams() method, which optimizes an objective function maximizing the positive difference between scores of true FPAs of shared identified compounds and their respective misaligned pairings.: [evidence span withheld — anchor status: not_found]
