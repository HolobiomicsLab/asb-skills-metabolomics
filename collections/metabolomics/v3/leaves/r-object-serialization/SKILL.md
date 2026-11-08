---
name: r-object-serialization
description: Use when an R-based ML pipeline trains a model (e.g. a caret random forest
  on 80/20-split quality-score data) that must be persisted to disk and reused by
  a downstream processing function such as automRm::process_batch().
when_to_use_negative:
- The model must be retrained on new data—re-run train_model() rather than loading
  a stale serialized object
- The analysis target is peak picking or raw-data preprocessing itself (use process_batch()),
  not persisting the trained peak-reporting model
- The workflow is not in R/caret (e.g. Python scikit-learn models require a different
  serialization mechanism such as pickle/joblib)
tools:
- name: automRm
  role: Provides train_model(), which trains and serializes the ML model to disk for
    later use by process_batch()
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: caret
  role: Performs hyperparameter tuning and random-forest training; the resulting caret
    model object is the entity serialized to disk
- name: R
  role: Language/runtime in which train_model(), model serialization, and model loading
    take place (tested on R 3.6.0 and 4.0.2)
- name: openxlsx
  role: Reads the manual_peakcheck.xlsx expert labels and writes xlsx files during
    the training workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:fe450e6c78bd502a2e3314dd25fac8b9812db7734d3aeecc2f4d5b08241af062
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - automRm
  - caret
  - openxlsx
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# r-object-serialization

## Summary

Serializes trained R objects—here, the caret-trained random forest peak-reporting model from automRm's train_model()—to disk so they can be reloaded and applied later by process_batch() without retraining.

## When to use

Use when an R-based ML pipeline trains a model (e.g. a caret random forest on 80/20-split quality-score data) that must be persisted to disk and reused by a downstream processing function such as automRm::process_batch().

## When NOT to use

- The model must be retrained on new data—re-run train_model() rather than loading a stale serialized object
- The analysis target is peak picking or raw-data preprocessing itself (use process_batch()), not persisting the trained peak-reporting model
- The workflow is not in R/caret (e.g. Python scikit-learn models require a different serialization mechanism such as pickle/joblib)

## Inputs

- qslog.tsv containing peak-picking classification scores for every metabolite in every sample (from process_batch())
- manual_peakcheck.xlsx with expert scores 0/1/2 per metabolite-sample pair (read via openxlsx)
- trained random forest R model object from caret hyperparameter tuning

## Outputs

- Serialized peak-reporting ML model file on disk, loadable by process_batch()
- Training-summary metrics for the 20% validation subset (e.g. F1 score)

## How to apply

In R, run automRm::train_model() after collecting the peak-picking classification scores in qslog.tsv and the expert labels (0 = do not report, 1 = exclude from training, 2 = report) in manual_peakcheck.xlsx, read with openxlsx. train_model() combines these inputs, automatically splits the data into 80% training and 20% validation subsets, and performs hyperparameter tuning of a random forest peak-reporting model via the caret package. Once validation performance is acceptable, serialize the trained model object to disk so that process_batch() can load it directly instead of retraining. Record training-summary metrics (e.g. F1 score) for the validation subset alongside the serialized model so future runs can be quality-checked. Rationale: persisting the model object decouples the expensive training step from repeated batch preprocessing of LC-QQQ-MS MRM raw data.

## Related tools

- **automRm** (Provides train_model(), which trains and serializes the ML model to disk for later use by process_batch()) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **caret** (Performs hyperparameter tuning and random-forest training; the resulting caret model object is the entity serialized to disk)
- **R** (Language/runtime in which train_model(), model serialization, and model loading take place (tested on R 3.6.0 and 4.0.2))
- **openxlsx** (Reads the manual_peakcheck.xlsx expert labels and writes xlsx files during the training workflow)

## Examples

```
Rscript -e 'automRm::train_model()  # combines qslog.tsv + manual_peakcheck.xlsx, trains caret RF, serializes model to disk'
```

## Evaluation signals

- The serialized model file loads without error in a fresh R session and is consumed by process_batch()
- Validation-set F1 score on the 20% held-out subset exceeds 0.98 for combined models trained on combined data sets, consistent with reported peak-reporting performance
- The training data split is 80% training / 20% validation as applied by train_model()
- Training inputs match: quality scores from qslog.tsv joined with 0/1/2 labels from manual_peakcheck.xlsx for every metabolite-sample pair
- A training-summary metric file/log exists recording the F1 score of the validation subset

## Limitations

- automRm requires sufficient training on relevant raw data to reach quality similar to manual peak review; a serialized model trained on inappropriate data will not generalize
- For optimal performance the acquisition must include at least one qualifier in addition to the quantifier, constraining the data the model can be trained on
- Manual peak review is still needed for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods, even with a serialized trained model

## Evidence

- [methods] train_model() for training the ML models used in process_batch(): "train_model() for training the ML models used in process_batch()"
- [methods] caret for ML: "caret13 for machine learning (ML)"
- [methods] openxlsx for xlsx IO: "openxlsx14 for reading and writing of xlsx files"
- [discussion] sufficient training requirement: "With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review."
- [abstract] package availability: "The R package is freely available from gitlab (https://gitlab.gwdg.de/joerg.buescher/automrm)."
