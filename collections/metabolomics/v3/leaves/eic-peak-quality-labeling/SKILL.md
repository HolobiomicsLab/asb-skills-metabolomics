---
name: eic-peak-quality-labeling
description: Use when building or retraining a CNN peak-quality model for LC-MS metabolic
  feature fidelity, where a corpus of EIC plots must be manually annotated as good
  or poor quality before training and validation.
when_to_use_negative:
- Feature tables where EIC plots have already been scored by a trained EVA/pyEVA model
  — no new manual labeling is needed for downstream prediction.
- Analysis pipelines that only need automated prediction of existing plots (e.g.,
  running pyEVA's FinalTable.csv/PredictionOutcomes.csv workflow) rather than training-data
  curation.
- Datasets lacking corresponding .mzXML or .mzML raw files, so EIC plots cannot be
  rendered or the source of poor EIC shapes cannot be traced.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: EVA
  role: The trained CNN model embedded in a C#-based Windows application that the
    manually labeled EIC plots are used to train and validate.
  repo: github.com/HuanLab-backup/EVA
- name: pyEVA
  role: Python reimplementation used to render EIC plots from feature tables and .mzXML/.mzML
    files into the classifier folder, then train and evaluate the CNN on the manually
    labeled corpus.
  repo: https://github.com/HuanLab/pyEVA
- name: XCMS
  role: R package installed in R Studio to preprocess raw LC-MS data and generate
    the EIC plot images that are subsequently manually labeled.
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c01309
    title: 'EVA: Evaluation of Metabolic Feature Fidelity Using a Deep Learning Model
      Trained With Over 25000 Extracted Ion Chromatograms'
  source_content_hash: sha256:804c12defd364154ef754b81cbf204988eb09fc125e5bfefa5211bb5fc67c830
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - EVA
  - XCMS
  - pyEVA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# eic-peak-quality-labeling

## Summary

Manually inspect extracted ion chromatogram (EIC) plots generated from LC-MS-based metabolomics feature tables and assign each plot a binary good/poor quality label, producing a curated training corpus for CNN-based peak quality classifiers such as EVA/pyEVA.

## When to use

Use when building or retraining a CNN peak-quality model for LC-MS metabolic feature fidelity, where a corpus of EIC plots must be manually annotated as good or poor quality before training and validation.

## When NOT to use

- Feature tables where EIC plots have already been scored by a trained EVA/pyEVA model — no new manual labeling is needed for downstream prediction.
- Analysis pipelines that only need automated prediction of existing plots (e.g., running pyEVA's FinalTable.csv/PredictionOutcomes.csv workflow) rather than training-data curation.
- Datasets lacking corresponding .mzXML or .mzML raw files, so EIC plots cannot be rendered or the source of poor EIC shapes cannot be traced.

## Inputs

- EIC plot images rendered from LC-MS feature tables paired with .mzXML or .mzML raw files
- Manually assigned good/poor EIC quality labels for each plot
- Corpus metadata: sample type, LC and MS condition per project (22 projects, 25095 plots)

## Outputs

- Manually labeled corpus of 25095 good/poor EIC plots for CNN training and validation
- Class-balanced training and validation subsets of labeled EIC plots
- Serialized CNN model artifact (cnn_model.h5) ready for embedding in EVA or pyEVA

## How to apply

Collect EIC plot images rendered from feature tables and corresponding .mzXML or .mzML raw files (e.g., via XCMS in R, as used by pyEVA). Manually inspect every EIC plot — the paper inspected all 25095 plots drawn from 22 LC-MS-based metabolomics projects spanning various sample types, LC and MS conditions — and assign a binary good/poor quality label based on whether the chromatographic peak shape indicates a real metabolic feature or a likely false positive. Preserve the good/poor class balance when partitioning plots into training and validation subsets, since the labeled corpus directly determines whether the downstream CNN reaches the reported >90% classification accuracy. Store the labeled plots in an image format compatible with the classifier (the trained EVA model is exported as a serialized Keras .h5 artifact for embedding in C# or Python pipelines).

## Related tools

- **EVA** (The trained CNN model embedded in a C#-based Windows application that the manually labeled EIC plots are used to train and validate.) — github.com/HuanLab-backup/EVA
- **pyEVA** (Python reimplementation used to render EIC plots from feature tables and .mzXML/.mzML files into the classifier folder, then train and evaluate the CNN on the manually labeled corpus.) — https://github.com/HuanLab/pyEVA
- **XCMS** (R package installed in R Studio to preprocess raw LC-MS data and generate the EIC plot images that are subsequently manually labeled.)

## Examples

```
python main.py  # after placing feature table (.csv with m/z, rt columns) and .mzXML/.mzML files in input/; choose smoothing level (0, 1, 2) to render EIC plots in EICplots/ under classifier/
```

## Evaluation signals

- The manually labeled corpus must reach the paper's reported scale of 25095 EIC plots from 22 LC-MS-based metabolomics projects, each carrying a good or poor EIC quality label.
- Validation of the downstream CNN on held-out labeled plots must exceed 90% classification accuracy, matching the reported comprehensive evaluation of EVA.
- Every EIC plot in the corpus must be manually inspected (the paper explicitly states all plots were inspected) — no plots should carry auto-generated or default labels.
- The resulting model artifact (cnn_model.h5) must be loadable by pyEVA and produce per-plot good/poor predictions consistent with manual labels in PredictionOutcomes.csv.
- Coverage of sample types, LC and MS conditions should span the 22 source projects so the labels generalize across varied chromatographic and spectrometric conditions.

## Limitations

- Manual labeling is time-intensive: all 25095 EIC plots must be individually inspected to assign accurate good/poor labels for training.
- The label quality directly caps downstream model performance; the paper reports >90% accuracy only because the labels were manually curated.
- The CNN binary good/poor scheme does not capture intermediate peak-quality gradations, so ambiguous EIC shapes may be forced into one of two classes.
- Windows-oriented EVA application is C#-based; cross-platform users must rely on the Python pyEVA rewrite.

## Evidence

- [readme] pyEVA renders EIC plots from feature tables and .mzXML files: [evidence span withheld — anchor status: external_doc]
- [readme] pyEVA outputs PredictionOutcomes.csv with Boolean good/poor predictions: [evidence span withheld — anchor status: external_doc]
