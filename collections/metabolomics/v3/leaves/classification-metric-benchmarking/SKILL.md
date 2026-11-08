---
name: classification-metric-benchmarking
description: Use when comparing two peak-classification approaches on identical LC/MS
  data and needing to quantify whether a low-label active-learning model matches or
  beats a transfer-learning model trained on more labels.
when_to_use_negative:
- Input is a single model's predictions with no competing baseline to compare against
  — use standard single-model validation instead
- The datasets or validation peak sets differ between the two classifiers, so accuracy/recall/precision
  are not computed on the same labeled peaks
- The goal is peak detection/feature finding rather than comparing curation classifiers
  on an existing peak list
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PeakDetective
  role: Active-learning peak classifier whose accuracy/recall/precision on 70 selected
    labels is one side of the benchmark
  repo: https://github.com/pattilab/PeakDetective
- name: NeatMS
  role: Transfer-learning baseline retrained with 70, ~300, and ~400 manually labeled
    peaks on the same data
- name: XCMS
  role: Generates the initial peak list from the mzML files that both classifiers
    curate
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:bb2b8ef8fa98bac410dcf69be8ca1d013a3524bf344a783603f0aaf117d14493
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - NeatMS
  - PeakDetective
  - Python
  - TensorFlow
  - XCMS
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Classification-metric benchmarking

## Summary

Benchmark a peak-curation classifier (PeakDetective trained via active learning on 70 peaks) against a transfer-learning baseline (NeatMS retrained with 70/300/400 manually labeled peaks) by tabulating accuracy, recall (TPR), and precision (1-FDR) on the same Dataset 1 validation peaks.

## When to use

Use when comparing two peak-classification approaches on identical LC/MS data and needing to quantify whether a low-label active-learning model matches or beats a transfer-learning model trained on more labels.

## When NOT to use

- Input is a single model's predictions with no competing baseline to compare against — use standard single-model validation instead
- The datasets or validation peak sets differ between the two classifiers, so accuracy/recall/precision are not computed on the same labeled peaks
- The goal is peak detection/feature finding rather than comparing curation classifiers on an existing peak list

## Inputs

- Dataset 1 raw mzML files
- XCMS-generated peak list (peaks_formatted.csv with mz, rt columns)
- Manually labeled true/artifact peak annotations for NeatMS retraining (70, ~300, ~400 peaks)
- PeakDetective validation peak labels (numVal = 10) and active-learning labels (~70 peaks)

## Outputs

- benchmark_comparison.csv tabulating accuracy, recall (TPR), and precision (1-FDR) for PeakDetective (70 labels) vs NeatMS (70/300/400 labels)
- TPR/FDR curves per cutoff from the validation peaks
- Saved PeakDetective model weights (integ.save())

## How to apply

Train PeakDetective on Dataset 1 mzML files and the XCMS peak list (ms1ppm = 25.5, resolution = 60 points/min, window = 1.0 min, align = True) with trainClassifierActive (numVal = 10, 10 labels/round, 200-1000 epochs, classifierBatchSize = 4) until ~70 user-labeled peaks, then set cutoff = 0.8 from the TPR/FDR curves and record accuracy, recall, and precision on validation peaks. Independently retrain the NeatMS deep learning model three times on the same data using 70, ~300, and ~400 manually labeled peaks and record its accuracy, recall, and precision for each labeling budget. Tabulate PeakDetective (70 active labels) versus NeatMS (70/300/400 labels) in a benchmark table (e.g. benchmark_comparison.csv). Acceptance is that PeakDetective with 70 actively selected peaks achieves higher accuracy, recall, and precision than NeatMS at every labeling budget, reproducing the reported pattern where NeatMS reached only 63.9% accuracy at 70 peaks and 69.3% at ~400 peaks. Save the PeakDetective model weights with integ.save() so the benchmark is auditable.

## Related tools

- **PeakDetective** (Active-learning peak classifier whose accuracy/recall/precision on 70 selected labels is one side of the benchmark) — https://github.com/pattilab/PeakDetective
- **NeatMS** (Transfer-learning baseline retrained with 70, ~300, and ~400 manually labeled peaks on the same data)
- **XCMS** (Generates the initial peak list from the mzML files that both classifiers curate)

## Examples

```
integ.trainClassifierActive(X,[],[],200,1000,4,1,numVal=10,numManualPerRound=10,inJupyter=True); pd.DataFrame(rows).to_csv('benchmark_comparison.csv')
```

## Evaluation signals

- NeatMS accuracy should be 63.9% at 70 labeled peaks and 69.3% at ~400 peaks on Dataset 1 (Table 1/Table S2 pattern)
- PeakDetective trained on 70 actively selected peaks must exceed NeatMS accuracy, recall, and precision at all three labeling budgets
- Reported accuracy on training and validation data must agree with the benchmark table values
- Bimodal score histogram with sharply dropped val_mean_absolute_error confirms the classifier was trained before metrics were read
- Cutoff = 0.8 chosen from the plotted TPR/FDR curves, and saved model weights allow audit of the reported metrics

## Limitations

- Metrics are estimated from only 10 user-labeled validation peaks (numVal = 10), so TPR/FDR estimates carry sampling uncertainty
- The comparison depends on active-learning peak selection; with different selected peaks the 70-label advantage may not reproduce
- NeatMS results require retraining three times at different labeling budgets, so the benchmark is sensitive to transfer-learning training variance

## Evidence

- [results] On Dataset 1, NeatMS accuracy improved with more labeled peaks (63.9% at 70 peaks, 69.3% at ~400 peaks) but even with ~400 peaks remained slightly below PeakDetective trained on just 70 active-learning-selected peaks.: [evidence span withheld — anchor status: not_found]
- [abstract] Through active learning, the classifier is trained with less than 100 user-labeled peaks in a matter of minutes: "Through active learning, the classifier is trained with less than 100 user-labeled peaks in a matter of minutes"
- [readme] After 5-10 iterations, a bimodal distribution of scores will be produced and the val_mean_absolute_error will drop dramatically.: "After 5-10 iterations, a bimodal distribution of scores will be produced and the val_mean_absolute_error will drop dramatically"
- [readme] the FDR and TPR estimates based on the validation data will be reported for each cutoff: "the FDR and TPR estimates based on the validation data will be reported for each cutoff"
