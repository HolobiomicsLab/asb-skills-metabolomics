---
name: prediction-error-threshold-coverage-evaluation
description: Use when you have paired true vs. predicted retention times for a held-out
  test set and need to quantify model coverage at error cutoffs and the FDR impact
  of an RT-based candidate filter.
when_to_use_negative:
- Input is a prediction table without paired ground-truth retention times — coverage
  and false negative rate cannot be computed
- Evaluating on the full SMRT dataset including non-retained molecules — the article
  reports that removing not retained molecules changes the average absolute error
  from 33.41 to 27.30, so full-data evaluation is not comparable
- Predictions were made under a chromatographic method the model was not transferred/fine-tuned
  to — RT-Transformer was pre-trained on SMRT and transferred to 5 PredRet datasets
  by transfer learning
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: RT-Transformer
  role: Produces the retention time predictions whose absolute errors, threshold coverage,
    and ±2 SD filter false negative rate are computed on the SMRT retained test set
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:bb890d0fee8e581aff1041e97232a84c4d8ad67ac28fc4ab128e7aa33a2ccfa0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# prediction-error-threshold-coverage-evaluation

## Summary

Evaluates a retention time prediction model by computing the distribution of absolute prediction errors on a held-out test set (e.g. 7,790 SMRT retained molecules) and quantifying coverage at fixed error thresholds (60 s, 120 s) plus the false negative rate induced by a ±2 SD filter (52.80 s).

## When to use

Use when you have paired true vs. predicted retention times for a held-out test set and need to quantify model coverage at error cutoffs and the FDR impact of an RT-based candidate filter.

## When NOT to use

- Input is a prediction table without paired ground-truth retention times — coverage and false negative rate cannot be computed
- Evaluating on the full SMRT dataset including non-retained molecules — the article reports that removing not retained molecules changes the average absolute error from 33.41 to 27.30, so full-data evaluation is not comparable
- Predictions were made under a chromatographic method the model was not transferred/fine-tuned to — RT-Transformer was pre-trained on SMRT and transferred to 5 PredRet datasets by transfer learning

## Inputs

- Retained-test-set table (CSV) with true and RT-Transformer-predicted retention times for the 7,790 SMRT molecules
- RT-Transformer model predictions (from test.py or the pretrained best_state_download_dict.pth model)
- SD of prediction errors (52.80 s) defining the ±2 SD filter threshold

## Outputs

- smrt_retained_test_error_stats.csv containing coverage under 60 s, coverage under 120 s, and false negative rate at the ±2 SD threshold
- Per-molecule absolute error column (|predicted - true|)

## How to apply

Load the retained-test-set table holding true and predicted retention times for the 7,790 SMRT molecules and compute the absolute error |predicted - true| per molecule. Compute the coverage fractions of molecules with absolute error < 60 s and < 120 s. Then apply the ±2 SD filter threshold (SD = 52.80 s, i.e. ±105.60 s around the prediction) and compute the false negative rate of true matches incorrectly excluded. On the SMRT retained test set the expected results are 6306/7790 molecules (89.84%) with error <60 s, 6726 (95.82%) <120 s, and a 4.93% false negative rate under the ±2 SD filter. Write the three statistics to smrt_retained_test_error_stats.csv.

## Related tools

- **RT-Transformer** (Produces the retention time predictions whose absolute errors, threshold coverage, and ±2 SD filter false negative rate are computed on the SMRT retained test set) — https://github.com/01dadada/RT-Transformer

## Examples

```
python ./test.py
```

## Evaluation signals

- Coverage at <60 s should be 89.84% (6306 of 7790) and at <120 s should be 95.82% (6726 of 7790) on the SMRT retained test set
- False negative rate at the ±2 SD (52.80 s) threshold should be 4.93%
- Row count of the error table must equal 7,790 retained-test molecules
- Average absolute error on the retained set should be near the reported 27.30 (vs 33.41 without removing non-retained molecules)

## Limitations

- The 89.84%/95.82% coverage figures are specific to the SMRT retained test set after removing not retained molecules; full-data error distribution differs (average absolute error 33.41 vs 27.30)
- The ±2 SD threshold (SD = 52.80 s) was calibrated on RT-Transformer's SMRT error distribution and may not transfer to other models or chromatographic methods
- RT-Transformer requires Python 3.9 with torch, rdkit-pypi, torch_geometric, torch-scatter/sparse/cluster; transfer-learning datasets must be CSVs with 'InChI' and 'RT' columns

## Evidence

- [abstract] Average absolute error on SMRT: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [abstract] Threshold rationale: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods"
- [readme] Test invocation: [evidence span withheld — anchor status: external_doc]
- [readme] Pretrained retained-data model: [evidence span withheld — anchor status: external_doc]
