---
name: relative-error-metrics-computation
description: Use when evaluating predicted vs. measured retention times from a chromatographic
  RT-prediction model (e.g. GNN-RT), to quantify accuracy as mean/median relative
  error.
when_to_use_negative:
- Comparing retention-time predictions against a dataset with no measured retention
  times — the metric requires ground-truth measured RTs.
- Evaluating model components other than prediction accuracy, such as the GNN graph
  representation itself.
- A scenario where the target chromatographic system differs in type from the training
  system, since the paper's transfer result applies only to 'the same type of chromatographic
  systems'.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3452
tools:
- name: Anaconda for python 3.6
  role: Environment in which the GNN-RT training/transfer scripts and metric computation
    are run.
- name: pytorch
  role: Framework for the GNN-RT model whose predicted RTs are evaluated with the
    relative-error metrics.
- name: rdkit
  role: Used in GNN-RT preprocessing to build molecular graphs whose retention-time
    predictions are scored with the metrics.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c04071
    title: Prediction of Liquid Chromatographic Retention Time with Graph Neural Networks
      to Assist in Small Molecule Identification
  source_content_hash: sha256:cb1ceccd146451677f24bb4534de06373e18f5f6e244e275f6e5da4a570c881f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Anaconda for python 3.6
  - pytorch
  - rdkit
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# relative-error-metrics-computation

## Summary

Computes mean and median relative error between predicted and measured LC retention times, the accuracy metrics the GNN-RT paper uses to benchmark graph-neural-network RT prediction and transfer learning on the METLIN SMRT dataset.

## When to use

Use when evaluating predicted vs. measured retention times from a chromatographic RT-prediction model (e.g. GNN-RT), to quantify accuracy as mean/median relative error.

## When NOT to use

- Comparing retention-time predictions against a dataset with no measured retention times — the metric requires ground-truth measured RTs.
- Evaluating model components other than prediction accuracy, such as the GNN graph representation itself.
- A scenario where the target chromatographic system differs in type from the training system, since the paper's transfer result applies only to 'the same type of chromatographic systems'.

## Inputs

- Predicted retention times from a trained GNN-RT model
- Measured (experimental) LC retention times for the same molecules
- Trained model checkpoint from Train.py
- Processed target-system data from Preprocess.py

## Outputs

- Mean relative error (%)
- Median relative error (%)
- transfer_learning_metrics.csv (mean and median relative errors for the transferred system)

## How to apply

After a trained GNN-RT model produces predicted retention times for a set of small molecules with measured LC retention times, compute the relative error per molecule as |predicted − measured| / measured, then average over the dataset (mean relative error) and take the median (median relative error). The paper's benchmark on the METLIN small molecule retention time (SMRT) dataset is a mean relative error of 4.9% and median relative error of 3.2%; treat deviations far above these levels as a sign the model or preprocessing (e.g. Preprocess.py output) is misapplied. The same computation is applied when evaluating the SMRT-trained model after transfer to another chromatographic system via Transferlearning.py, so the metric serves both in-distribution and transfer-learning evaluation.

## Related tools

- **Anaconda for python 3.6** (Environment in which the GNN-RT training/transfer scripts and metric computation are run.)
- **pytorch** (Framework for the GNN-RT model whose predicted RTs are evaluated with the relative-error metrics.)
- **rdkit** (Used in GNN-RT preprocessing to build molecular graphs whose retention-time predictions are scored with the metrics.)

## Examples

```
python Transferlearning.py  # then record mean and median relative errors to transfer_learning_metrics.csv
```

## Evaluation signals

- Mean relative error ≈ 4.9% on the METLIN SMRT dataset matches the paper's reported accuracy.
- Median relative error ≈ 3.2% matches the paper's reported median accuracy.
- Mean and median relative errors are written to transfer_learning_metrics.csv after running Transferlearning.py.
- Relative errors are computed consistently for both the original SMRT model and the transferred model.
- Predicted RT values come from a GNN-RT checkpoint produced by Train.py, not from unrelated predictors.

## Limitations

- Transfer-learning performance is validated only for chromatographic systems of the same type; accuracy may drop for dissimilar systems.
- The reported 4.9% mean / 3.2% median errors are tied to the METLIN SMRT dataset and may not transfer to in-house data of different composition.
- The README does not specify hyperparameter tuning details or a changelog, so the exact evaluation pipeline may be sensitive to preprocessing choices.
- No quantitative threshold is given for when a transferred-system error is considered acceptable beyond comparison to the reported ~4.9% mean / ~3.2% median.

## Evidence

- [abstract] GNN-RT achieved the highest predicting accuracy with a mean relative error of 4.9% and a median relative error of 3.2%: "GNN-RT achieved the highest predicting accuracy with a mean relative error of 4.9% and a median relative error of 3.2%."
- [abstract] the SMRT-trained GNN-RT model can be transferred to the same type of chromatographic systems easily: "the SMRT-trained GNN-RT model can be transferred to the same type of chromatographic systems easily"
- [readme] please put your spectra files in to data directory and run Preprocess.py, Train.py and Transferlearning.py: "please put your spectra files in to **data** directory and run [Preprocess.py](Preprocess.py), [Train.py](Train.py) and [Transferlearning.py](Transferlearning.py)"
- [readme] conda install pytorch: "conda install pytorch"
