---
name: chromatographic-retention-matching-window-analysis
description: Use when a predicted-versus-true retention time table exists and you
  need to set an RT matching window (e.g., ±2 SD) for metabolite identification and
  quantify how many true matches that window would exclude.
when_to_use_negative:
- The dataset contains non-retained molecules and you have not removed them — error
  statistics will be inflated (33.41 s vs 27.30 s average absolute error when not-retained
  molecules are excluded)
- Chromatographic conditions differ from those the model was transferred to — RT-Transformer
  must first be fine-tuned on a dataset for the specific chromatographic method before
  error distributions can be trusted
- You need per-candidate metabolite scoring rather than RT-window statistics — this
  skill characterizes the error distribution, it does not rank candidate structures
tools:
- name: RT-Transformer
  role: Predicts retention times under any chromatographic method (graph attention
    network + 1D-Transformer); its predictions on the SMRT retained test set provide
    the error distribution used to derive matching windows and FNR.
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:d6b91a031913b818cca6c0c338fbacc40ed41df0e2d281320d5b9d7163c74424
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# chromatographic-retention-matching-window-analysis

## Summary

Quantifies RT-Transformer retention-time prediction error on a retained-molecule test set (e.g., the SMRT retained set of 7,790 molecules) and derives candidate-matching windows (±2 SD = 52.80 s) with associated false negative rates, so that RT filters can be used to prune metabolite annotation candidates.

## When to use

Use when a predicted-versus-true retention time table exists and you need to set an RT matching window (e.g., ±2 SD) for metabolite identification and quantify how many true matches that window would exclude.

## When NOT to use

- The dataset contains non-retained molecules and you have not removed them — error statistics will be inflated (33.41 s vs 27.30 s average absolute error when not-retained molecules are excluded)
- Chromatographic conditions differ from those the model was transferred to — RT-Transformer must first be fine-tuned on a dataset for the specific chromatographic method before error distributions can be trusted
- You need per-candidate metabolite scoring rather than RT-window statistics — this skill characterizes the error distribution, it does not rank candidate structures

## Inputs

- Retained-molecule test table with true and RT-Transformer-predicted retention times (e.g., 7,790 SMRT molecules, RT in seconds)
- SMRT dataset (from https://doi.org/10.1038/s41467-019-13680-7) with 'InChI' and 'RT' columns for prediction runs
- Pre-trained RT-Transformer model weights (best_state_download_dict.pth for retained data; best_state_dict.pth for full data)

## Outputs

- smrt_retained_test_error_stats.csv with: % molecules |error| < 60 s, % molecules |error| < 120 s, SD of errors, ±2 SD threshold, and false negative rate
- Per-molecule absolute error column derived from predicted minus true RT

## How to apply

Load the retained-test-set table holding true and RT-Transformer-predicted retention times (SMRT retained set: 7,790 molecules) and compute absolute error |predicted - true| per molecule. Report coverage statistics, e.g. the fraction of molecules with absolute error <60 s (6306 molecules, 89.84%) and <120 s (6726 molecules, 95.82%). Compute the SD of the prediction errors (52.80 s on this set) and apply a ±2 SD matching window (52.80 s), then count true matches incorrectly excluded to obtain the false negative rate (4.93% on the SMRT retained set). Rationale: RT-Transformer achieved an average absolute error of 27.30 s after removing non-retained molecules (33.41 s with no samples removed), so SD-based windows sized from the error distribution keep FNR low while still filtering candidates. Write all three statistics to smrt_retained_test_error_stats.csv. Use the pre-trained model trained on the 80,038-molecule small molecule retention time dataset, transferred/fine-tuned to the chromatographic conditions of interest.

## Related tools

- **RT-Transformer** (Predicts retention times under any chromatographic method (graph attention network + 1D-Transformer); its predictions on the SMRT retained test set provide the error distribution used to derive matching windows and FNR.) — https://github.com/01dadada/RT-Transformer

## Examples

```
python ./test.py
```

## Evaluation signals

- Coverage check: on the SMRT retained set (7,790 molecules), |error| < 60 s should cover ~89.84% (6,306 molecules) and <120 s ~95.82% (6,726 molecules)
- Threshold check: SD of errors should be 52.80 s; the ±2 SD filter should yield a false negative rate of 4.93%
- Output file smrt_retained_test_error_stats.csv exists and contains all three statistics (60 s coverage, 120 s coverage, FNR)
- Per-molecule error column satisfies |predicted - true| >= 0 and matches the row count of the retained test set (7,790)
- Predictions must come from the correct model weights: best_state_download_dict.pth (retained-data model) vs best_state_dict.pth (full-data model)

## Limitations

- Error statistics and the ±2 SD window (52.80 s) are specific to the SMRT retained test set and its chromatographic conditions; other datasets require recomputing the distribution
- Non-retained molecules inflate error (average absolute error 33.41 s with no samples removed vs 27.30 s after removal), so the retained/not-retained split must be handled consistently
- The RT filter trades coverage for specificity: even the ±2 SD window excludes 4.93% of true matches as false negatives

## Evidence

- [abstract] the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [readme] Run test.py by `python ./test.py`: [evidence span withheld — anchor status: external_doc]
- [readme] The SMRT dataset is collect from this paper: [evidence span withheld — anchor status: external_doc]
- [readme] best_state_download_dict.pth The Best model of RT-Transformer train from retained data.: [evidence span withheld — anchor status: external_doc]
