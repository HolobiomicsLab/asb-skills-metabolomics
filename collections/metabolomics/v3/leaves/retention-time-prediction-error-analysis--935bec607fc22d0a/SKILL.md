---
name: retention-time-prediction-error-analysis
description: Use when you have true vs. predicted retention times for a fixed test
  set of molecules (e.g. SMRT) and need error-distribution coverage statistics and
  a ±2 SD exclusion-threshold false negative rate.
when_to_use_negative:
- Your goal is to train or fine-tune RT-Transformer rather than evaluate prediction
  error and filter thresholds (use train.py/transfer.py workflows instead).
- Your retention time predictions come from a different chromatographic method/dataset
  and the error SD (52.80 s) and coverage thresholds have not been re-derived for
  that setting.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: RT-Transformer
  role: Deep neural network (graph attention network + 1D-Transformer) that generates
    the predicted retention times being evaluated; test.py runs its validation/test
    predictions
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:79e3e06ce5e488c73684cdbba242e4ce779cf629fbf1fafbf35165d4fc3b8697
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# retention-time-prediction-error-analysis

## Summary

Quantifies RT-Transformer retention time prediction error on a held-out test set by computing absolute-error coverage at fixed thresholds (60 s, 120 s) and evaluating a ±2 SD filter threshold for false negative control during metabolite annotation.

## When to use

Use when you have true vs. predicted retention times for a fixed test set of molecules (e.g. SMRT) and need error-distribution coverage statistics and a ±2 SD exclusion-threshold false negative rate.

## When NOT to use

- Your goal is to train or fine-tune RT-Transformer rather than evaluate prediction error and filter thresholds (use train.py/transfer.py workflows instead).
- Your retention time predictions come from a different chromatographic method/dataset and the error SD (52.80 s) and coverage thresholds have not been re-derived for that setting.

## Inputs

- CSV/table of true and RT-Transformer-predicted retention times for the 7,790-molecule SMRT retained test set (e.g. InChI/RT/predicted_RT columns)
- Error SD value for the prediction model (52.80 s on the SMRT retained test set)

## Outputs

- smrt_retained_test_error_stats.csv with percentage of molecules with absolute error <60 s (89.84%), <120 s (95.82%), and false negative rate at the ±2 SD (52.80 s) filter (4.93%)

## How to apply

Load the retained-test-set table holding true and RT-Transformer-predicted retention times for the 7,790 SMRT retained molecules and compute the absolute error |predicted - true| per molecule. Compute coverage fractions at 60 s and 120 s absolute-error cutoffs (on the SMRT retained test set these are 89.84% and 95.82%, i.e. 6306 and 6726 of 7,790 molecules). For annotation filtering, derive the ±2 SD threshold from the error standard deviation (SD = 52.80 s, giving a 52.80 s filter) and compute the false negative rate of true matches incorrectly excluded (4.93% in the reference evaluation). Write the three statistics to a CSV (e.g. smrt_retained_test_error_stats.csv). Rationale: RT prediction error filtering assists metabolite identification by excluding candidates whose predicted RT deviates too much from the observed peak, so the FNR at the chosen SD multiple is the key decision metric.

## Related tools

- **RT-Transformer** (Deep neural network (graph attention network + 1D-Transformer) that generates the predicted retention times being evaluated; test.py runs its validation/test predictions) — https://github.com/01dadada/RT-Transformer

## Examples

```
python ./test.py
```

## Evaluation signals

- Molecule count consistency: 6306/7790 (89.84%) and 6726/7790 (95.82%) for the 60 s and 120 s cutoffs on the SMRT retained test set
- Coverage must be monotonic: the fraction with absolute error <120 s must exceed the fraction <60 s
- False negative rate at the ±2 SD threshold (SD = 52.80 s) should approximate 4.93% on this test set
- Absolute errors are non-negative and computed element-wise as |predicted - true| with no missing molecules relative to the 7,790-row test table
- Output CSV contains exactly the three statistics (coverage <60 s, coverage <120 s, FNR at SD filter)

## Limitations

- The 52.80 s SD threshold and 4.93% FNR are specific to the SMRT retained test set; transfer to other chromatographic conditions requires re-deriving these statistics.
- Performance differs between retained and full data: RT-Transformer's average absolute error was 27.30 after removing not-retained molecules but 33.41 when no samples were removed.
- Filtering by RT deviation discards true matches; the false negative rate must be weighed against the annotation gain.

## Evidence

- [results] On the SMRT retained test set, 6306 molecules (89.84%) have absolute RT prediction error <60 s and 6726 molecules (95.82%) <120 s: [evidence span withheld — anchor status: not_found]
- [results] applying the ±2 SD threshold (SD = 52.80 s) reduces the false negative rate to 4.93%: [evidence span withheld — anchor status: not_found]
- [abstract] the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [readme] Run test.py by `python ./test.py `: [evidence span withheld — anchor status: external_doc]
