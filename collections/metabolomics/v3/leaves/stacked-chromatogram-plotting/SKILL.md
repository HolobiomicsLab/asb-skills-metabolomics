---
name: stacked-chromatogram-plotting
description: Use when non-targeted LC-MS peak lists (e.g. from MZmine 2 centWave extraction)
  must be transformed into image representations so a CNN can score peaks as true
  vs false.
when_to_use_negative:
- The input is already a feature/parameter table (e.g. 59 peak group parameters) destined
  for the multiple logistic regression or Random Forest filter models rather than
  the image-based DNN.
- Raw chromatographic peak shapes are unavailable (only summary peak quality metrics),
  since the raster images require per-sample intensity-vs-scan curves.
- The goal is peak detection itself — features must already have been extracted (e.g.
  MZmine 2 centWave) before stacked plots can be generated.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Generates the non-filtered peak lists whose features are converted into stacked
    peak plots
- name: R
  role: Custom R scripts produce the stacked peak plots for each extracted feature
- name: EDML_deep_learning2.py
  role: Consumes the raster-image training/calibration/test sets to train and evaluate
    the CNN
  repo: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:17cab9af8cdbf295061eab3072e4b7595baecba7a305675d9aafd53904f6217f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
  tools:
  - EDML_deep_learning2.py
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# stacked-chromatogram-plotting

## Summary

Renders each extracted LC-MS peak group as a stacked chromatogram plot over its 63 most intense samples and converts the stack into a 64x64 raster image (pixel = intensity, row = sample, column = time/scans) suitable as input to the image-based deep neural network peak classifier.

## When to use

Use when non-targeted LC-MS peak lists (e.g. from MZmine 2 centWave extraction) must be transformed into image representations so a CNN can score peaks as true vs false.

## When NOT to use

- The input is already a feature/parameter table (e.g. 59 peak group parameters) destined for the multiple logistic regression or Random Forest filter models rather than the image-based DNN.
- Raw chromatographic peak shapes are unavailable (only summary peak quality metrics), since the raster images require per-sample intensity-vs-scan curves.
- The goal is peak detection itself — features must already have been extracted (e.g. MZmine 2 centWave) before stacked plots can be generated.

## Inputs

- Non-filtered LC-MS peak lists extracted with MZmine 2 (mzXML-derived peak data from Cohort 1/Cohort 2 human plasma)
- Per-sample peak intensity and retention-time/scan information for each extracted feature
- Author-supplied custom R scripts and EDML_deep_learning2.py from JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks

## Outputs

- One stacked peak plot per extracted feature covering the 63 samples where the peak is most intense
- 64x64 raster images per peak group (pixel = intensity, row = sample, column = time/scans) formatted for CNN input
- Probability scores per peak group from the downstream trained DNN (consumed for ROC/AUC computation)

## How to apply

Start from non-filtered MZmine 2 peak lists generated with 'More Restrictive' and/or 'Less Restrictive' extraction parameters. For each extracted feature, select the 63 samples in which the candidate peak was most intense and produce a stacked peak plot from that peak information using custom R scripts within the MZmine 2 workflow. Convert the stacked peak-shape information into a 64x64 raster image in which each pixel value indicates intensity, each row represents a single sample, and each column represents a time/scans dimension. These images then feed the training/calibration/test pipeline (EDML_deep_learning2.py: 1304 training, 740 calibration, 726 test manually reviewed Cohort 1 peaks), so consistent image construction across all splits is essential for the model's probability scores and the reported AUCs (0.992 train, 0.988 test) to be reproducible.

## Related tools

- **MZmine 2** (Generates the non-filtered peak lists whose features are converted into stacked peak plots)
- **R** (Custom R scripts produce the stacked peak plots for each extracted feature)
- **EDML_deep_learning2.py** (Consumes the raster-image training/calibration/test sets to train and evaluate the CNN) — https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks

## Examples

```
python EDML_deep_learning2.py
```

## Evaluation signals

- Each raster image is exactly 64x64 with rows corresponding to individual samples and columns to the time/scans dimension; pixel values encode intensity.
- Each stacked plot is built from peak information in exactly the 63 samples where the candidate peak was most intense.
- Images are generated consistently across the 1304 training, 740 calibration, and 726 test Cohort 1 peak examples.
- Downstream reproduction check: the trained model reaches AUC ~0.992 (training) and ~0.988 (test), with the optimum threshold 0.4725 giving TPR ~0.945 and FPR ~0.04.

## Limitations

- Only the 63 most intense samples per peak are used, discarding peak information from other samples in the cohort.
- The raster images are only as good as the upstream MZmine 2 peak extraction; no independent quality filtering is applied before plotting.
- Script EDML_deep_learning2.py is provided without a version number, and the deep learning framework/library used to build the model is not named in the article.

## Evidence
