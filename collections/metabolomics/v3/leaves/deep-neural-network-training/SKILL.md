---
name: deep-neural-network-training
description: Use when you need to classify LC-MS spectral peaks as true or false from
  nontargeted metabolomics data, especially when you have manually reviewed peak examples
  (e.g., ~1300 training peaks) and peak-extraction features rendered as raster images.
when_to_use_negative:
- Input is already a curated peak-quality feature table where hand-engineered peak
  group parameters (e.g., medRelFWHM) suffice — a Random Forest or multiple logistic
  regression on the 59 peak group parameters is the article's alternative, not the
  image-based DNN
- You lack manually reviewed peak examples for training/calibration/testing; the model
  requires labelled true/false peak groups
- You need a filter for something other than true vs false spectral peaks (e.g., cohort-level
  statistical testing, annotation, or ID) — the DNN's sole output is a true-peak probability
  score
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: MZmine 2
  role: Used to generate non-filtered peak lists from raw LC-MS data with both 'More
    Restrictive' and 'Less Restrictive' peak extraction parameters before rasterizing
    to images
- name: R
  role: Used for custom scripts to generate stacked peak plots and raster image representations
    from MZmine 2 peak-list output
- name: EDML_deep_learning2.py
  role: Author-provided script to train the image-based deep neural network model
    from the downloaded datasets
  repo: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:502b5004064fd743d4adad2333f567b32d0f5b315fec69a968c988f055bdfe59
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
  tools:
  - EDML_deep_learning2.py
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# deep-neural-network-training

## Summary

Train an image-based deep neural network on rasterized LC-MS peak plots to assign each peak group a probability score of being a true peak, achieving AUCs of 0.992 (training) and 0.988 (test). The approach treats stacked peak shape images as classifier input, bypassing hand-engineered peak-quality metrics.

## When to use

Use when you need to classify LC-MS spectral peaks as true or false from nontargeted metabolomics data, especially when you have manually reviewed peak examples (e.g., ~1300 training peaks) and peak-extraction features rendered as raster images.

## When NOT to use

- Input is already a curated peak-quality feature table where hand-engineered peak group parameters (e.g., medRelFWHM) suffice — a Random Forest or multiple logistic regression on the 59 peak group parameters is the article's alternative, not the image-based DNN
- You lack manually reviewed peak examples for training/calibration/testing; the model requires labelled true/false peak groups
- You need a filter for something other than true vs false spectral peaks (e.g., cohort-level statistical testing, annotation, or ID) — the DNN's sole output is a true-peak probability score

## Inputs

- Raw mzXML-derived peak data from non-targeted HILIC LC-MS runs (e.g., Cohort 1 N=78 human plasma)
- Author-provided manual peak classifications (1304 training examples, 740 calibration, 726 test)
- Stacked peak plots for each extracted feature (peak info from the 63 samples where the peak was most intense)
- Raster images (64×64, pixel = intensity, row = sample, column = scans/time)

## Outputs

- Trained deep neural network model assigning each peak group a probability score of being a true peak
- Receiver operating characteristic (ROC) curves with true positive rate and false positive rate at each score threshold
- dnn_peak_classification_metrics.csv containing AUC (0.992 training / 0.988 test) and selected score threshold (0.4725)
- Rendered ROC curve plot

## How to apply

1. Download the deposited Cohort 1 raw mzXML-derived peak data and author-provided manual peak classifications from https://doi.org/doi:10.25345/C5FD2F. 2. For each extracted feature, generate stacked peak plots using peak information from the 63 samples in which the candidate peak was most intense, and convert the stacked peak shape information into 64×64 raster images where each pixel value encodes intensity, each row a single sample, and each column a time/scans dimension. 3. Train the convolutional neural network on 1304 manually reviewed peak examples (50% true, 50% false) and calibrate on 740 peak examples using the author's script EDML_deep_learning2.py from github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks. 4. Test the calibrated model on 726 held-out peak examples, assigning each peak group a probability score of being a true peak. 5. Compute ROC curves to determine the true positive rate (sensitivity) and true negative rate (specificity) at each score threshold cutoff value. 6. Select the optimum operating point (score threshold of 0.4725) and report training/test AUC (0.992/0.988), writing metrics to dnn_peak_classification_metrics.csv and rendering the ROC curve. Decision point: the training-set size can be downsampled — a high-performing model is achievable with fewer than 1000 peak examples, though the required size depends on the specific LC method.

## Related tools

- **MZmine 2** (Used to generate non-filtered peak lists from raw LC-MS data with both 'More Restrictive' and 'Less Restrictive' peak extraction parameters before rasterizing to images)
- **R** (Used for custom scripts to generate stacked peak plots and raster image representations from MZmine 2 peak-list output)
- **EDML_deep_learning2.py** (Author-provided script to train the image-based deep neural network model from the downloaded datasets) — https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks

## Examples

```
python EDML_deep_learning2.py
```

## Evaluation signals

- Reproduced training-set AUC should be ~0.992 and held-out test-set AUC ~0.988 on Cohort 1 peaks
- At the selected score threshold of 0.4725, the true positive rate should be approximately 0.945 and the false positive rate approximately 0.04
- Learning-curve check: a high-performing model is achieved with fewer than 1000 training peak examples, though performance depends on the specific LC method
- Training set must be balanced at 50% true peaks and 50% false peaks (1304 examples), with 740 calibration and 726 test examples
- Raster images must be 64×64 with pixel intensity encoding, rows as samples, columns as scans/time

## Limitations

- The required training-set size depends on the specific LC method used, so a model trained on one HILIC method may not transfer directly to another
- The image-based DNN needs manually reviewed peak examples for training/calibration/testing; fully unsupervised peak filtering is out of scope
- The article's manual training set contains 50% true and 50% false peaks, so class imbalance is baked in by design; raw feature lists from less restrictive MZmine settings can have ~75% false positives
- No specific version details for the neural net implementation are given in the intro; the script EDML_deep_learning2.py is provided without a version
- Only Cohort 1 (N=78) was used for training/calibration/testing; Cohort 2 (N=526) was used for independent evaluation with the peak group parameter model

## Evidence

- [results] Threshold selection: [evidence span withheld — anchor status: not_found]
