---
name: numerical-implementation-validation
description: Use when porting or verifying a published statistical estimator (e.g.
  B-spline MI as in MOG) and you need ground-truth comparison, invariants, and a validation
  report before trusting the implementation.
when_to_use_negative:
- The goal is to compute MI for exploratory analysis directly rather than to validate
  a re-implementation — use an existing trusted MI library instead.
- The features under comparison are known to be dependent, so the I(X;Y)=0-for-independence
  invariant cannot serve as a sanity check.
- The estimator being validated is not a published, citable numerical method with
  an available reference implementation.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3577
tools:
- name: Java
  role: Implementation language used for MetaOmGraph, in which the B-spline MI estimator
    being validated is written
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: MetaOmGraph
  role: Reference software whose Mutual Information module applies the Daub et al.
    B-spline MI estimator
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: MOG
  role: Statistical analysis workbench whose pairwise MI computation is the behavior
    under validation
  repo: https://github.com/urmi-21/MetaOmGraph/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:46dacc99f7d2b4e3394f76a646bd369c848a31c7da5f3d1c47428b537acbd30c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - Java
  - MOG
  - MetaOmGraph
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# numerical-implementation-validation

## Summary

Re-implements a published numerical estimator — here the Daub et al. B-spline mutual information estimator used in MetaOmGraph's Mutual Information module — from scratch, then validates it against an independent reference implementation using sanity invariants (non-negativity, near-zero MI for independent features) and pairwise value agreement.

## When to use

Use when porting or verifying a published statistical estimator (e.g. B-spline MI as in MOG) and you need ground-truth comparison, invariants, and a validation report before trusting the implementation.

## When NOT to use

- The goal is to compute MI for exploratory analysis directly rather than to validate a re-implementation — use an existing trusted MI library instead.
- The features under comparison are known to be dependent, so the I(X;Y)=0-for-independence invariant cannot serve as a sanity check.
- The estimator being validated is not a published, citable numerical method with an available reference implementation.

## Inputs

- small numeric expression matrix (feature rows x sample columns, CSV)
- published estimator specification (Daub et al. B-spline MI method)
- independent reference MI implementation for ground truth

## Outputs

- pairwise MI values in CSV format (mi_matrix.csv)
- validation report with comparison summary (agreement within tolerance, sanity-check results)

## How to apply

Load a small numeric expression matrix (feature rows x sample columns) into the analysis environment, then implement the Daub et al. estimator: estimate each feature's marginal density with B-splines and evaluate I(X;Y) for every feature pair, exploiting the property that I(X;Y)=0 exactly when X and Y are statistically independent. Compute MI between the same feature pairs with an independent reference MI implementation to serve as ground truth. Check sanity properties: MI values must be non-negative, and pairs of simulated random/independent features must yield MI near zero. Quantify agreement between the B-spline estimator and the reference across all pairs at value-level within a stated tolerance. Write the pairwise MI values to mi_matrix.csv and record the comparison summary in a validation report. This mirrors how MOG itself grounds its Mutual Information module in the Daub et al. B-spline approach.

## Related tools

- **Java** (Implementation language used for MetaOmGraph, in which the B-spline MI estimator being validated is written) — https://github.com/urmi-21/MetaOmGraph/
- **MetaOmGraph** (Reference software whose Mutual Information module applies the Daub et al. B-spline MI estimator) — https://github.com/urmi-21/MetaOmGraph/
- **MOG** (Statistical analysis workbench whose pairwise MI computation is the behavior under validation) — https://github.com/urmi-21/MetaOmGraph/

## Examples

```
python validate_mi.py --expression expression.csv --estimator bspline --reference sklearn_mi --tolerance 1e-6 --output mi_matrix.csv --report validation_report.md
```

## Evaluation signals

- All computed MI values are non-negative across every feature pair.
- Pairs of simulated random / statistically independent features yield MI values near zero.
- B-spline estimator values agree with the independent reference MI implementation at value level within the stated tolerance for all pairs.
- mi_matrix.csv is well-formed and contains one MI value per selected feature pair.
- The validation report records both the agreement summary and the sanity-check outcomes.

## Limitations

- B-spline MI is a density-estimation-based similarity measure; agreement with a reference implementation depends on matching the estimator's parameters (e.g. number of B-splines/bins) to the published Daub et al. specification.
- Validation on a small numeric matrix only exercises the estimator on limited data regimes; behavior on MOG-scale massive expression datasets is not covered by this check.
- Near-zero MI on simulated random features verifies the independence property but does not guarantee correctness for strongly dependent feature pairs.

## Evidence

- [other] Implement the Daub et al. MI estimator: estimate each feature's marginal density with B-splines and evaluate I(X;Y) for every feature pair: [evidence span withheld — anchor status: not_found]
- [results] MOG computes mutual information using B-splines density estimation, following the approach described in Daub et al.: [evidence span withheld — anchor status: not_found]
- [readme] If you use Mutual Information module in MetaOmGraph, please also cite [Daub et al. Estimating mutual information using B-spline functions: [evidence span withheld — anchor status: external_doc]
- [intro] we present MetaOmGraph (MOG), a Java software, to interactively explore and visualize large expression datasets: "we present MetaOmGraph (MOG), a Java software, to interactively explore and visualize large expression datasets"
- [other] exploiting the property that I(X;Y)=0 exactly when X and Y are statistically independent: [evidence span withheld — anchor status: not_found]
