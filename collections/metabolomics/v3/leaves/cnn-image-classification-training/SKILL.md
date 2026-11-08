---
name: cnn-image-classification-training
description: Use when a collection of EIC plot images from LC-MS metabolomics runs
  must be labeled good/poor and a CNN classifier must be trained and deployed for
  peak-quality prediction.
when_to_use_negative:
- The goal is batch EIC classification of already rendered plots rather than training
  a new model — pyEVA ships the pretrained EVA model and main.py pipeline for inference
- Input data are raw spectra without a feature table providing m/z and rt columns,
  so no EIC plots can be rendered for labeling or classification
- The task is evaluating chromatographic peak quality outside LC-MS-based metabolomics
  EIC rendering, where the trained good/poor EIC labels and model are not applicable
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: pyEVA
  role: Python environment for training and running the EVA CNN classifier on EIC
    plot images (Mac, PC, Linux)
  repo: https://github.com/HuanLab/pyEVA
- name: EVA
  role: C#-based Windows application into which the trained CNN model is embedded
    for automatic peak-quality evaluation
- name: C#
  role: Implementation language of the EVA Windows application that hosts the embedded
    CNN model
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c01309
    title: 'EVA: Evaluation of Metabolic Feature Fidelity Using a Deep Learning Model
      Trained With Over 25000 Extracted Ion Chromatograms'
  source_content_hash: sha256:705f72ca8a62d519a3be32dc04409d604fcdc32ad7c6d3235cf1ee06a2c19fc8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/pyEVA
  tools:
  - C#
  - EVA
  - pyEVA
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cnn-image-classification-training

## Summary

Train and embed a CNN image classifier that assigns good versus poor quality labels to extracted ion chromatogram (EIC) plots, as described for the EVA metabolomics feature-fidelity tool.

## When to use

Use when a collection of EIC plot images from LC-MS metabolomics runs must be labeled good/poor and a CNN classifier must be trained and deployed for peak-quality prediction.

## When NOT to use

- The goal is batch EIC classification of already rendered plots rather than training a new model — pyEVA ships the pretrained EVA model and main.py pipeline for inference
- Input data are raw spectra without a feature table providing m/z and rt columns, so no EIC plots can be rendered for labeling or classification
- The task is evaluating chromatographic peak quality outside LC-MS-based metabolomics EIC rendering, where the trained good/poor EIC labels and model are not applicable

## Inputs

- EIC plot images generated from 25095 manually annotated plots across 22 LC-MS-based metabolomics projects
- Manually assigned good or poor EIC quality labels for each plot
- LC-MS feature table (m/z and rt columns) with related .mzXML or .mzML files, from which EIC plots are rendered before classification

## Outputs

- Trained CNN model artifact (cnn_model.h5) for good/poor peak-quality classification
- Per-plot good/poor prediction labels (e.g. PredictionOutcomes.csv with Boolean values)
- Classification accuracy metric over held-out labeled EIC plots

## How to apply

Assemble a training corpus of 25095 EIC plots collected from 22 LC-MS-based metabolomics projects spanning various sample types and LC/MS conditions. Manually inspect every EIC plot to assign a good or poor quality label, which the authors state is necessary for accurate model training. Train a CNN image classifier on the labeled plots (in the pyEVA Python environment) as a binary good/poor task, then embed the trained model artifact into the C#-based EVA Windows application. Evaluate on held-out plots and confirm the classifier reaches over 90% classification accuracy, matching the reported comprehensive evaluation of EVA before deployment.

## Related tools

- **pyEVA** (Python environment for training and running the EVA CNN classifier on EIC plot images (Mac, PC, Linux)) — https://github.com/HuanLab/pyEVA
- **EVA** (C#-based Windows application into which the trained CNN model is embedded for automatic peak-quality evaluation)
- **C#** (Implementation language of the EVA Windows application that hosts the embedded CNN model)

## Examples

```
conda env activate pyEVA && python main.py --input feature_table.csv --mzxml sample.mzXML --smoothing 1
```

## Evaluation signals

- The trained CNN achieves over 90% classification accuracy on held-out manually-inspected EIC plots, matching the reported comprehensive evaluation of EVA
- Every one of the 25095 training plots carries a manually assigned good or poor quality label — no unlabeled plots remain
- The trained model serializes as cnn_model.h5 and loads into the C#-based EVA program without error
- On pyEVA demo data, PredictionOutcomes.csv contains one Boolean good/poor prediction per row of the input feature table
- Training corpus spans the stated diversity — plots from 22 distinct LC-MS-based metabolomics projects with varied sample types, LC and MS conditions

## Limitations

- Requires a large manually inspected corpus — 25095 plots from 22 projects were needed for accurate model training, so manual labeling cost is high
- The C#-based EVA is a Windows application; cross-platform use requires the rewritten pyEVA version
- Classifier performance depends on EIC plot rendering parameters (e.g. chosen smoothing level 0, 1, or 2 in pyEVA) matching training-time image conventions
- The README notes pyEVA is pinned to Python 3.7.12 and that older PyCharm/VS Code versions may behave differently during environment setup

## Evidence

- [readme] pyEVA purpose: automating EIC plot fidelity checks: [evidence span withheld — anchor status: external_doc]
- [readme] pyEVA input format and run command: [evidence span withheld — anchor status: external_doc]
- [readme] pyEVA outputs after inference: [evidence span withheld — anchor status: external_doc]
- [readme] Smoothing level choice affects EIC rendering: [evidence span withheld — anchor status: external_doc]
