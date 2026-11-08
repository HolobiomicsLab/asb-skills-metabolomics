---
name: mutual-information-estimation
description: Use when you need a non-linear similarity measure between pairs of expression
  features (e.g., gene rows x sample columns in a numeric matrix) where Pearson/Spearman
  correlation is insufficient.
when_to_use_negative:
- Input is already a precomputed correlation or distance matrix rather than raw feature-level
  expression values
- The analysis goal is a purely linear association, where Pearson/Spearman correlation
  (as used elsewhere in MOG, e.g., Spearman >= 0.60 thresholds) is sufficient and
  simpler
- Working interactively with large expression datasets inside MOG where its built-in
  Mutual Information module already provides the Daub et al. estimator
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MetaOmGraph (MOG)
  role: Java workbench whose Mutual Information module implements the Daub et al.
    B-spline MI estimator for statistical association analyses
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: Java
  role: Language in which MOG and its Mutual Information module are implemented; used
    to reimplement the B-spline MI estimator
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:dae83b5d360d8c3192941abc4474a764ebcf8ea627749729029d9fcf9436b2c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - Java
  - MOG
  - MetaOmGraph
  - MetaOmGraph (MOG)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# mutational-information-estimation via B-splines (Daub et al.) as implemented in MetaOmGraph

## Summary

Estimates pairwise mutual information between features of an expression matrix using the Daub et al. B-spline density-estimation MI estimator, the method behind MetaOmGraph's Mutual Information module for statistical association analyses.

## When to use

Use when you need a non-linear similarity measure between pairs of expression features (e.g., gene rows x sample columns in a numeric matrix) where Pearson/Spearman correlation is insufficient.

## When NOT to use

- Input is already a precomputed correlation or distance matrix rather than raw feature-level expression values
- The analysis goal is a purely linear association, where Pearson/Spearman correlation (as used elsewhere in MOG, e.g., Spearman >= 0.60 thresholds) is sufficient and simpler
- Working interactively with large expression datasets inside MOG where its built-in Mutual Information module already provides the Daub et al. estimator

## Inputs

- Small numeric expression matrix (feature rows x sample columns, e.g., gene expression values)
- List of feature pairs to evaluate (or all pairs)

## Outputs

- Pairwise mutual information values written in CSV format to mi_matrix.csv
- Validation report comparing B-spline MI estimates against an independent reference MI implementation

## How to apply

Load a numeric expression matrix (feature rows x sample columns) and implement the Daub et al. estimator: model each feature's marginal density with B-splines and evaluate I(X;Y) for every feature pair, exploiting the property that I(X;Y)=0 exactly when X and Y are statistically independent. This is the method MetaOmGraph (a Java software) uses in its Mutual Information module for its co-expression/association analyses. Validate by computing MI for the same pairs with an independent reference MI implementation, then confirm sanity properties: MI values are non-negative, and simulated-random or statistically independent feature pairs yield MI near zero. Quantify value-level agreement between the B-spline estimator and the reference implementation within a stated tolerance.

## Related tools

- **MetaOmGraph (MOG)** (Java workbench whose Mutual Information module implements the Daub et al. B-spline MI estimator for statistical association analyses) — https://github.com/urmi-21/MetaOmGraph/
- **Java** (Language in which MOG and its Mutual Information module are implemented; used to reimplement the B-spline MI estimator)

## Examples

```
git clone https://github.com/urmi-21/MetaOmGraph.git
```

## Evaluation signals

- All MI values in mi_matrix.csv are non-negative
- Pairs of simulated-random or statistically independent features yield MI values near zero
- Value-level agreement between the B-spline estimator and the independent reference MI implementation across all pairs within a stated tolerance
- mi_matrix.csv parses as a symmetric matrix over the same feature pairs analyzed by both implementations
- Validation report records the comparison summary for every evaluated pair

## Limitations

- Requires B-spline density estimation choices (order/knot placement per Daub et al.); poor spline settings can bias estimates
- Near-zero MI for independent features is exact only under the estimator's ideal properties; finite-sample estimates may deviate slightly from zero
- MOG is a Java standalone application requiring JRE 8+ (R 3.4+ optional), so programmatic reuse means reimplementing rather than scripting MOG directly

## Evidence

- [readme] Daub B-spline MI attribution: [evidence span withheld — anchor status: external_doc]
- [readme] MOG purpose: "MetaOmGraph (MOG) is a Java software to interactively explore and visualize large datasets."
- [intro] MOG Java software claim: "we present MetaOmGraph (MOG), a Java software, to interactively explore and visualize large expression datasets"
- [results] MOG statistical analyses: "The statistical analyses and visualizations shown were generated exclusively using MOG."
