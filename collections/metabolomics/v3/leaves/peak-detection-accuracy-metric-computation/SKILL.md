---
name: peak-detection-accuracy-metric-computation
description: Use when a custom-trained PeakBot CNN model must be quantitatively evaluated
  — verify peak/background classification accuracy exceeds 0.99 and that train-vs-validation
  loss/metric gaps stay below 5%.
when_to_use_negative:
- Peak detection quality is being assessed by feature-count or peak-shape criteria
  rather than the model's peak-vs-background classification accuracy
- The chromatograms under evaluation overlap with the training data — validation must
  use chromatograms and reference features fully disjoint from training data
- Fewer than 100 reference features are available, which the authors state is the
  minimum needed to achieve high-quality peak-picking results
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PeakBot
  role: Provides the trained CNN model and evaluation workflow whose peakType_ACCPeakNoPeak
    metric is computed on validation datasets (V, iT, iV, eV) generated automatically
    by train.py
  repo: https://github.com/christophuv/PeakBot
- name: TensorFlow
  role: Runs the CNN model inference and reports the peak-type accuracy and loss metrics
    on training and validation sets
  repo: https://www.tensorflow.org/
- name: PeakBot_Example
  role: Supplies train.py and detect.py scripts showing how training/validation datasets
    are generated and how the trained model is evaluated before use on new chromatograms
  repo: https://github.com/christophuv/PeakBot_Example
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:1325fcdc3e935f3c9f22be8286694a097f6f29e853eccb7115242f0d410f4e7f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/christophuv/PeakBot
  repo_url: https://github.com/christophuv/PeakBot
  tools:
  - Anaconda
  - CUDA toolkit
  - PeakBot
  - PeakBot_Example
  - Python
  - TensorFlow
  - cuDNN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# peak-detection-accuracy-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes and validates the peakType_ACCPeakNoPeak classification accuracy of a PeakBot CNN model when discriminating true chromatographic peaks from background signals (including walls), on independent validation sets disjoint from training data.

## When to use

Use when a custom-trained PeakBot CNN model must be quantitatively evaluated — verify peak/background classification accuracy exceeds 0.99 and that train-vs-validation loss/metric gaps stay below 5%.

## When NOT to use

- Peak detection quality is being assessed by feature-count or peak-shape criteria rather than the model's peak-vs-background classification accuracy
- The chromatograms under evaluation overlap with the training data — validation must use chromatograms and reference features fully disjoint from training data
- Fewer than 100 reference features are available, which the authors state is the minimum needed to achieve high-quality peak-picking results

## Inputs

- Trained PeakBot CNN model (TensorFlow 2.5.0 checkpoint)
- Independent validation LC-HRMS profile-mode chromatograms (iV) disjoint from training chromatograms
- Reference feature and background/wall lists used to build validation datasets
- Training-set accuracy and loss/metric values (T, V, iT) for gap comparison

## Outputs

- peakType_ACCPeakNoPeak accuracy value on the independent validation set (iV)
- Training vs validation loss/metric comparison showing the gap percentage
- Overfitting assessment (gap < 5%, accuracy > 0.99)

## How to apply

Run the trained PeakBot CNN model on the independent validation set (iV), built from LC-HRMS chromatograms and reference features fully disjoint from the training set, and record the peakType_ACCPeakNoPeak metric (fraction of local maxima correctly classified as chromatographic peak versus background). Compare against training-set (T) accuracy and validation sets (V, iT, iV): the model is judged not overfitted when accuracy exceeds 0.99 and loss/metric values differ by less than 5% between training and validation. Reference features must number at least 100 (isotopologs of the same compound can count toward this) and should span the different peak shapes (fronting, tailing) present in the data. During training, PeakBot automatically generates a training dataset (T) and four validation datasets (V, iT, iV, eV) from different chromatograms and reference/background lists, so evaluation signals can be compared across all of them.

## Related tools

- **PeakBot** (Provides the trained CNN model and evaluation workflow whose peakType_ACCPeakNoPeak metric is computed on validation datasets (V, iT, iV, eV) generated automatically by train.py) — https://github.com/christophuv/PeakBot
- **TensorFlow** (Runs the CNN model inference and reports the peak-type accuracy and loss metrics on training and validation sets) — https://www.tensorflow.org/
- **PeakBot_Example** (Supplies train.py and detect.py scripts showing how training/validation datasets are generated and how the trained model is evaluated before use on new chromatograms) — https://github.com/christophuv/PeakBot_Example

## Examples

```
python train.py  # trains PeakBot model, generating T, V, iT, iV, eV datasets; then python detect.py to evaluate peakType_ACCPeakNoPeak on iV
```

## Evaluation signals

- peakType_ACCPeakNoPeak on the independent validation set (iV) must exceed 0.99
- Loss/metric values between training (T) and validation sets (V, iT, iV) must differ by less than 5 percentage points, indicating no overfitting
- Validation chromatograms and reference features must be verifiably disjoint from those used to build training instances
- The reference feature list contained at least 100 features (isotopologs allowed), and example detections exported as images pass user verification

## Limitations

- Accuracy above 99% is only achieved when at least 100 reference features are provided for training; fewer features degrade peak-picking quality
- Care must be taken that no co-eluting compounds (isomers) are present in the reference chromatograms, or the reference feature values determined by gradient-descend will be unreliable
- The metric reflects peak-vs-background discrimination (including walls); it does not by itself guarantee correct bounding-box or peak-center predictions, which should be checked via exported example detection images

## Evidence

- [readme] Validation datasets generated automatically: [evidence span withheld — anchor status: external_doc]
- [readme] Peak shapes must be covered: [evidence span withheld — anchor status: external_doc]
