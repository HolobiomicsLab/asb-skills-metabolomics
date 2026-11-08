---
name: permutation-significance-testing
description: Use when you have trained the SARTRE random forest classifier (e.g. on
  stitch_yeast with shadow-price or 128-bit fingerprint features) and need to verify
  that observed AUC/metrics are statistically significant rather than artifacts of
  label structure or feature information.
when_to_use_negative:
- You have only computed shadow prices with optimization.py or generated fingerprints
  with generate_fp.py but have not yet trained/evaluated the classifier via evaluate.py
- You are evaluating SARTRE on specific tasks (shared-pair exclusion via shared.py,
  subsystem exclusion via subsys.py, media-composition analysis) where the workflow
  defines its own comparisons rather than permutation P-values
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: SARTRE
  role: Framework whose permutation_tests.py script implements label_permutation and
    feature_permutation options on gold-standard datasets
  repo: https://github.com/fayazsoleymani/SARTRE
- name: scikit-learn
  role: Source of the random forest classifier and evaluation metrics that the permutation
    tests assess
- name: Python
  role: Runtime for executing permutation_tests.py in each gold-standard directory
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:ab66fc71786689e22af78f772fc4e1b0c05a8f27c66ffaa3638650cebfd30657
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fayazsoleymani/SARTRE
  tools:
  - Bio.Entrez
  - GUROBI
  - MATLAB
  - Python
  - RDKit
  - SARTRE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# permutation-significance-testing

## Summary

Statistical validation of SARTRE metabolite-protein interaction classifiers via label permutation and feature permutation tests, executed with the repository's permutation_tests.py script to obtain P-values confirming that classification performance (e.g. AUC ~0.85 on stitch_yeast) exceeds chance.

## When to use

Use when you have trained the SARTRE random forest classifier (e.g. on stitch_yeast with shadow-price or 128-bit fingerprint features) and need to verify that observed AUC/metrics are statistically significant rather than artifacts of label structure or feature information.

## When NOT to use

- You have only computed shadow prices with optimization.py or generated fingerprints with generate_fp.py but have not yet trained/evaluated the classifier via evaluate.py
- You are evaluating SARTRE on specific tasks (shared-pair exclusion via shared.py, subsystem exclusion via subsys.py, media-composition analysis) where the workflow defines its own comparisons rather than permutation P-values

## Inputs

- Trained SARTRE random forest classifier and constructed dataset for a gold standard (e.g. stitch_yeast, stitch_ecoli, piazza, reznik)
- Metabolite feature files produced in preprocessing: met_sps_t80_replaced.csv (shadow prices) and met_fp128.csv (128-bit RDKit fingerprints)
- STITCH confidence score threshold (150: low, 400: medium, 700: high, 900: highest) for STITCH-derived gold standards

## Outputs

- Permutation-test P-values for label permutation and feature permutation on the chosen gold standard
- Null-hypothesis rejection decision (e.g. P < 1e-7 expected for the stitch_yeast reproduction)

## How to apply

After training and evaluating the classifier with evaluate.py (e.g. `evaluate.py sp` for shadow prices or `evaluate.py fp` for fingerprints, adding a STITCH confidence score of 150/400/700/900 for STITCH gold standards), change into the gold-standard directory and run permutation_tests.py with either the label_permutation or feature_permutation option. Label permutation destroys the MPI/non-MPI label association; feature permutation tests the importance of the metabolite feature block (shadow prices vs fingerprints). For STITCH-derived datasets the confidence score is passed as the first argument before the permutation option. For the S. cerevisiae/stitch_yeast reproduction, the result is accepted as correct only if both permutation tests reject the null hypothesis with P < 1e-7, complementing a mean AUC of approximately 0.85 for both feature types.

## Related tools

- **SARTRE** (Framework whose permutation_tests.py script implements label_permutation and feature_permutation options on gold-standard datasets) — https://github.com/fayazsoleymani/SARTRE
- **scikit-learn** (Source of the random forest classifier and evaluation metrics that the permutation tests assess)
- **Python** (Runtime for executing permutation_tests.py in each gold-standard directory)

## Examples

```
cd stitch_yeast && python3 permutation_tests.py feature_permutation
```

## Evaluation signals

- Both label permutation and feature permutation tests reject the null hypothesis with P < 1e-7 on stitch_yeast
- Mean AUC of ~0.85 is obtained for both shadow-price and fingerprint features on stitch_yeast before permutation testing
- Script is run from the correct gold-standard directory (e.g. cd stitch_yeast) with the permutation option spelled exactly as label_permutation or feature_permutation
- For STITCH datasets the confidence score is passed as the first argument before the permutation option (e.g. 400 feature_permutation)
- Feature-importance ranking confirms the key role of shadow prices in predicting metabolite-protein interactions

## Limitations

- STITCH-derived datasets require an explicit confidence-score argument; omitting it will break the invocation
- Null-hypothesis rejection at P < 1e-7 is dataset-specific (stitch_yeast reproduction); other gold standards may yield different significance levels
- In the S. cerevisiae analysis, shadow-price and fingerprint features showed comparable performance (Table 4), so permutation significance does not imply shadow prices outperform fingerprints for a given gold standard
- Archives containing optimization results must be unzipped before running the pipeline steps

## Evidence

- [readme] Permutation tests workflow: "We examined the performance of SARTRE with two permutation tests of label permutation and feature permutation."
- [readme] STITCH confidence-score invocation: "For STITCH derived datasets, run the commands with desired confidence score as a first argument, e.g. stitch_ecoli with medium confidence score(400) and feature permutation test:"
- [other] stitch_yeast acceptance criterion: [evidence span withheld — anchor status: not_found]
