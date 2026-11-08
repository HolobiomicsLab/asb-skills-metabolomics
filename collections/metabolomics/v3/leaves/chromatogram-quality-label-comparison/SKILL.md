---
name: chromatogram-quality-label-comparison
description: Use when you have a feature table plus .mzXML/.mzML raw files, a CNN
  model (EVA/pyEVA) producing predicted EIC quality classes, and manually annotated
  good/poor labels against which classification accuracy must be measured.
when_to_use_negative:
- You have no manually annotated (manually inspected) good/poor EIC labels, so there
  is no ground truth to compare predicted labels against — use EVA purely for prediction
  instead.
- Your chromatograms are not LC-MS-based metabolomics EIC plots (e.g. GC-MS, LC-MS/MS
  spectra, or non-chromatographic data), which the EVA CNN was not trained to classify.
- Your goal is peak detection or feature deconvolution rather than evaluating feature
  fidelity/peak quality labels of already-extracted metabolic features.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3391
tools:
- name: pyEVA
  role: Python implementation of the EVA CNN used to run inference on EIC plots and
    output per-plot good/poor predictions for comparison against manual labels
  repo: https://github.com/HuanLab/pyEVA
- name: EVA
  role: C#-based Windows Application (also on Microsoft Store as 'EVA Metabolomics')
    that embeds the trained CNN model to process metabolic features and classify EIC
    quality
  repo: github.com/HuanLab-backup/EVA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c01309
    title: 'EVA: Evaluation of Metabolic Feature Fidelity Using a Deep Learning Model
      Trained With Over 25000 Extracted Ion Chromatograms'
  source_content_hash: sha256:c3517d967f62e3346a01747bc689c2403fa36643123da6323797851d741ebe79
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/pyEVA
  tools:
  - EVA
  - pyEVA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# chromatogram-quality-label-comparison

## Summary

Evaluate a CNN-based EIC peak-quality classifier (EVA/pyEVA) by running inference on manually inspected extracted ion chromatogram plots and comparing predicted good/poor labels against ground-truth annotations, yielding an overall classification accuracy metric.

## When to use

Use when you have a feature table plus .mzXML/.mzML raw files, a CNN model (EVA/pyEVA) producing predicted EIC quality classes, and manually annotated good/poor labels against which classification accuracy must be measured.

## When NOT to use

- You have no manually annotated (manually inspected) good/poor EIC labels, so there is no ground truth to compare predicted labels against — use EVA purely for prediction instead.
- Your chromatograms are not LC-MS-based metabolomics EIC plots (e.g. GC-MS, LC-MS/MS spectra, or non-chromatographic data), which the EVA CNN was not trained to classify.
- Your goal is peak detection or feature deconvolution rather than evaluating feature fidelity/peak quality labels of already-extracted metabolic features.

## Inputs

- Feature table (.csv) with first two columns m/z and rt followed by sample data columns
- Matching raw mass-spectrometry files (.mzXML or .mzML)
- Pretrained EVA CNN model (via pyEVA Python package or C#-based EVA Windows Application)
- Manually inspected EIC plots with assigned good/poor quality labels (ground truth)

## Outputs

- Per-plot predicted good/poor peak-quality class labels
- PredictionOutcomes.csv with Boolean predicted-true/false values for the input data
- FinalTable.csv containing the results of the analysis
- Aggregate overall classification accuracy across all evaluated EIC plots

## How to apply

Load the set of manually inspected EIC plots (e.g. the 25095 plots from 22 LC-MS-based metabolomics projects used in the EVA paper) together with their manually assigned good/poor labels. Instantiate the pretrained CNN via pyEVA (github.com/HuanLab/pyEVA) or the C#-based EVA Windows Application, and run inference on each EIC plot to obtain predicted good/poor peak-quality classes. First generate EICs from a feature table (first two columns m/z and rt) and the matching .mzXML/.mzML files by running python main.py in pyEVA, choosing a smoothing level (0, 1, or 2) so EICs land in the EICplots folder under classifier. Compare every predicted label against the manually inspected ground-truth label, then compute the overall classification accuracy across all plots. In the EVA paper the evaluation on the 25095 manually inspected plots showed over 90% classification accuracy, so an accuracy above 90% is the benchmark for a successful comparison.

## Related tools

- **pyEVA** (Python implementation of the EVA CNN used to run inference on EIC plots and output per-plot good/poor predictions for comparison against manual labels) — https://github.com/HuanLab/pyEVA
- **EVA** (C#-based Windows Application (also on Microsoft Store as 'EVA Metabolomics') that embeds the trained CNN model to process metabolic features and classify EIC quality) — github.com/HuanLab-backup/EVA

## Examples

```
python main.py
```

## Evaluation signals

- Overall classification accuracy on the manually inspected EIC plot set exceeds 90% (the reported comprehensive evaluation result)
- PredictionOutcomes.csv exists and contains one Boolean predicted-true/false value per input feature row
- FinalTable.csv is produced in the Output folder containing the analysis results
- EIC plot images are generated in the EICplots folder under classifier before inference runs
- Per-plot predictions match the count of manually inspected EIC plots compared (no missing/extra labels)

## Limitations

- Classification accuracy was established on EIC plots from 22 LC-MS-based metabolomics projects of various sample types, LC and MS conditions; performance outside this distribution is not guaranteed by the reported evaluation.
- The C#-based EVA Windows Application runs on Windows via the Microsoft Store; cross-platform use requires the pyEVA Python version.
- pyEVA setup depends on Anaconda, PyCharm or VS Code, R with xcms/ggplot2/scales/dplyr packages, and Python 3.7.9 — version drift in the IDE or packages may cause differences (the README notes 'differences may occur with older versions').
- The EVA paper's stated accuracy is a comprehensive evaluation aggregate; per-project or per-condition accuracy breakdowns are not provided in the surfaced context.

## Evidence

- [abstract] Reported >90% classification accuracy benchmark for EVA evaluation: "Our comprehensive evaluation of EVA indicates that it achieves over 90% classification accuracy."
- [abstract] Training corpus of 25095 EIC plots from 22 LC-MS-based metabolomics projects used for evaluation: "Our CNN model was trained using 25095 EIC plots collected from 22 LC-MS-based metabolomics projects of various sample types, LC and MS conditions."
- [readme] pyEVA output files produced for label comparison: [evidence span withheld — anchor status: external_doc]
- [readme] pyEVA invocation and smoothing-level choice that generates EICs prior to inference: [evidence span withheld — anchor status: external_doc]
- [abstract] Manual inspection of EIC plots establishes the good/poor ground-truth labels: "Notably, we manually inspected all the EIC plots to assign good or poor EIC quality for accurate model training."
- [abstract] Purpose of the tool — recognizing poor EIC shapes / false-positive features: "we take advantage of deep learning technology to develop a convolutional neural network (CNN)-based program that can automatically recognize metabolic features with poor EIC shapes, which are of low"
- [intro] pyEVA availability and platform scope: "EVA now has a python version that can be easily installed and used on Mac, PC, and Linux systems. Please check out pyEVA at https://github.com/HuanLab/pyEVA"
