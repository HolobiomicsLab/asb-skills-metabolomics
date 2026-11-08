---
name: false-negative-rate-estimation-for-rt-filtering
description: 'Use when evaluating or calibrating an RT-based metabolite-annotation
  filter: you have true vs. RT-Transformer-predicted retention times on a retained
  test set and need the fraction of molecules within error bounds (e.g. <60 s, <120
  s) and the FNR a ±2 SD threshold introduces.'
when_to_use_negative:
- Your test set still contains 'not retained' molecules — the article reports separate
  error regimes (27.30 s after removing not-retained molecules vs. 33.41 s with none
  removed), so retained-set statistics do not transfer
- The prediction model is not RT-Transformer trained/transferred under the same chromatographic
  conditions — the SD (52.80 s) and coverage fractions are model- and dataset-specific
- You need metabolite structural annotation itself rather than retention-time filter
  error characterization
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3172
tools:
- name: RT-Transformer
  role: Deep neural network (graph attention network + 1D-Transformer) that generates
    the predicted retention times whose absolute errors and ±2 SD filter statistics
    this skill computes
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:850de744ec6dd16207b435c9d5d44b1592064d6cab8d7086715715534126f9fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# false-negative-rate-estimation-for-rt-filtering

## Summary

Quantifies retention-time prediction error coverage and the false negative rate induced by an RT-Transformer ±2 SD (52.80 s) retention-time filter on the SMRT retained-molecule test set (7,790 molecules), so that metabolite-annotation filtering decisions are grounded in measured error statistics rather than assumed cutoffs.

## When to use

Use when evaluating or calibrating an RT-based metabolite-annotation filter: you have true vs. RT-Transformer-predicted retention times on a retained test set and need the fraction of molecules within error bounds (e.g. <60 s, <120 s) and the FNR a ±2 SD threshold introduces.

## When NOT to use

- Your test set still contains 'not retained' molecules — the article reports separate error regimes (27.30 s after removing not-retained molecules vs. 33.41 s with none removed), so retained-set statistics do not transfer
- The prediction model is not RT-Transformer trained/transferred under the same chromatographic conditions — the SD (52.80 s) and coverage fractions are model- and dataset-specific
- You need metabolite structural annotation itself rather than retention-time filter error characterization

## Inputs

- Retained-test-set table (CSV) with true retention times and RT-Transformer-predicted retention times for the 7,790 SMRT retained molecules
- Pre-trained RT-Transformer model file (best_state_download_dict.pth for retained-data predictions)
- Filter specification: ±2 SD threshold (SD = 52.80 s)

## Outputs

- Coverage statistics: fraction of molecules with absolute RT error <60 s (89.84%) and <120 s (95.82%)
- False negative rate under the ±2 SD (52.80 s) filter (4.93%)
- smrt_retained_test_error_stats.csv containing the computed statistics

## How to apply

Load the retained-test-set table holding true and RT-Transformer-predicted retention times for the 7,790 SMRT molecules (test.py workflow; SMRT retained molecules as used for the best_state_download_dict.pth model). Compute the absolute error |predicted − true| per molecule, then report coverage fractions: on this benchmark, 6306/7790 molecules (89.84%) have absolute error <60 s and 6726/7790 (95.82%) <120 s. Derive the filter threshold as ±2 SD of the error distribution (SD = 52.80 s), apply it to exclude molecules whose true RT falls outside the predicted window, and compute the false negative rate — true matches incorrectly excluded by the filter — which equals 4.93% here. Write all three statistics to a CSV (e.g. smrt_retained_test_error_stats.csv) for downstream filter-tuning comparisons. The rationale is that RT-Transformer's reported average absolute error (27.30 s on SMRT retained molecules) is distributional, so filter thresholds should be derived from the error SD, not the mean alone.

## Related tools

- **RT-Transformer** (Deep neural network (graph attention network + 1D-Transformer) that generates the predicted retention times whose absolute errors and ±2 SD filter statistics this skill computes) — https://github.com/01dadada/RT-Transformer

## Examples

```
python ./test.py
```

## Evaluation signals

- Coverage invariants: 6306/7790 molecules (89.84%) with |error| <60 s and 6726/7790 (95.82%) with |error| <120 s
- FNR under the ±2 SD (52.80 s) threshold must equal 4.93% on this benchmark
- Error-column schema check: |predicted − true| computed for all 7,790 molecules with no missing values
- Mean absolute error sanity check against the reported 27.30 s for RT-Transformer on the SMRT dataset after removing not-retained molecules
- Output file smrt_retained_test_error_stats.csv contains all three statistics (coverage <60 s, coverage <120 s, FNR)

## Limitations

- Statistics are specific to the SMRT retained-molecule test set (7,790 molecules); performance varies across chromatographic conditions, and RT-Transformer was only transferred to 5 PredRet-style datasets in the paper
- The ±2 SD threshold assumes an approximately well-behaved error distribution; molecules with |error| between 60 s and 120 s (about 6% of the set) show the filter's coverage degrades with looser or tighter bounds
- The workflow defines the threshold from the error SD (52.80 s); if the model is fine-tuned to new chromatographic methods, the SD and resulting FNR must be re-estimated

## Evidence

- [abstract] average absolute error on SMRT retained molecules: [evidence span withheld — anchor status: not_found]
- [abstract] predict retention times under any chromatographic methods: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods"
- [readme] test.py invocation: [evidence span withheld — anchor status: external_doc]
- [readme] SMRT dataset provenance: [evidence span withheld — anchor status: external_doc]
- [readme] retained-data model checkpoint: [evidence span withheld — anchor status: external_doc]
