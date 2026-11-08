---
name: grid-search-weight-region-verification
description: Use when tuning or validating the A, B, C weight parameters of the metabCombiner
  similarity-scoring function for LC-HRMS feature alignment, and you have a training
  set of shared identified compounds with known true pairings.
when_to_use_negative:
- Input datasets were acquired with disparate chromatographic techniques (e.g. RPLC
  vs HILIC), where elution order is not preserved and weight tuning cannot compensate.
- No shared identified compounds exist between the two datasets, so no 'true pairing'
  labels are available to define the separation objective.
- Abundance values have been normalized or transformed in a way that removes ranked
  abundance order information, breaking the abundance-similarity term of the scoring
  function.
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metabCombiner
  role: Provides the scoring function and evaluateParams() objective used to sweep
    and verify the (A, B, C) weight grid on training compounds.
  repo: https://github.com/hhabra/metabCombiner
- name: R
  role: Runtime environment in which the metabCombiner package and its evaluateParams()
    weight-optimization routine are executed.
  repo: https://github.com/hhabra/metabCombiner
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c03693
    title: '<i>metabCombiner</i>: Paired Untargeted LC-HRMS Metabolomics Feature Matching
      and Concatenation of Disparately Acquired Data Sets'
  source_content_hash: sha256:bc053ea923a9671ef29e16ea63e2aea9e89b6019f23d4626396f9b8e11c946e6
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
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# grid-search-weight-region-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct the metabCombiner evaluateParams() procedure by sweeping a grid of feature-pair-alignment scoring weights (A, B, C) on the package's paired human plasma datasets (plasma20/plasma30), maximizing the positive score separation between true FPAs of shared identified compounds and their misaligned pairings.

## When to use

Use when tuning or validating the A, B, C weight parameters of the metabCombiner similarity-scoring function for LC-HRMS feature alignment, and you have a training set of shared identified compounds with known true pairings.

## When NOT to use

- Input datasets were acquired with disparate chromatographic techniques (e.g. RPLC vs HILIC), where elution order is not preserved and weight tuning cannot compensate.
- No shared identified compounds exist between the two datasets, so no 'true pairing' labels are available to define the separation objective.
- Abundance values have been normalized or transformed in a way that removes ranked abundance order information, breaking the abundance-similarity term of the scoring function.

## Inputs

- Paired peak-picked untargeted LC-HRMS datasets plasma20 and plasma30 (same ionization mode, overlapping metabolite composition)
- Training-set table of shared identified compounds with m/z, retention time, and abundance values in both datasets
- Candidate grid of (A, B, C) weight triples for the metabCombiner scoring function

## Outputs

- Full grid of positive score-separation values for each (A, B, C) weight triple
- Identified optimal weight region (expected to contain A≈50–70, B≈14–15, C≈0.3–0.4)
- Verification result confirming the paper's defaults (A=100, B=15, C=0.3) lie in a near-maximal separation area

## How to apply

Load the plasma20 and plasma30 human plasma metabolomics datasets shipped with the metabCombiner R package together with the training-set table of identified compounds and their m/z, retention time, and abundance values. For each candidate weight triple (A, B, C) on the paper's grid, compute the weighted feature pair alignment (FPA) score for each training compound's true pairing and for its candidate misaligned pairings using the metabCombiner scoring function, which ranks alignments by m/z, mapped retention time, and relative abundance similarity. For each triple, calculate the positive score difference between the true FPAs of shared identified compounds and their misaligned pairings. Sweep the grid to identify the (A, B, C) region maximizing this separation, then verify the recovered region contains the vignette's reported optimum (A≈50–70, B≈14–15, C≈0.3–0.4) and that the paper's chosen values (A=100, B=15, C=0.3) yield a near-maximal separation among training compounds. Rationale: evaluateParams() optimizes this objective function so that true FPAs of shared identified compounds score clearly above their misaligned competitors, which is what makes the downstream combined table reduction reliable.

## Related tools

- **metabCombiner** (Provides the scoring function and evaluateParams() objective used to sweep and verify the (A, B, C) weight grid on training compounds.) — https://github.com/hhabra/metabCombiner
- **R** (Runtime environment in which the metabCombiner package and its evaluateParams() weight-optimization routine are executed.) — https://github.com/hhabra/metabCombiner

## Examples

```
evaluateParams(object, A = seq(0, 100, 10), B = seq(0, 20, 1), C = seq(0, 1, 0.05), training = training_compounds)
```

## Evaluation signals

- The recovered optimal (A, B, C) region contains the vignette's reported region A≈50–70, B≈14–15, C≈0.3–0.4.
- The paper's chosen values (A=100, B=15, C=0.3) yield a near-maximal positive score separation among training compounds.
- For every training compound, the true FPA score is strictly higher than its misaligned pairing competitors at the optimum.
- Mean absolute retention time prediction error on the plasma20/plasma30 pair is roughly 0.06 min after spline mapping.
- Weighted per-compound matching accuracy on the plasma dataset pair is approximately 90%.

## Limitations

- The optimal weights are dataset-specific; the preprocessing method and choice of parameters should be considered carefully for each data set.
- Datasets acquired from HILIC methods are more difficult to align than those by RPLC methods, which degrades score separation.
- Preprocessing errors such as incomplete or multipeak integration, low signal-to-noise features, and missed peaks propagate into the training labels and can distort the objective.
- The paper does not state the specific version of the metabCombiner R package used.

## Evidence

- [methods] evaluateParams() objective function and grid semantics: [evidence span withheld — anchor status: not_found]
