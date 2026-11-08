---
name: lc-ms-metabolomics-training-data-preparation
description: Use when building a deep-learning peak-quality classifier for LC-MS feature
  lists and you need labeled EIC plot images spanning diverse sample types, LC and
  MS conditions, with each plot manually inspected to assign good or poor quality
  labels.
when_to_use_negative:
- You only need to score an existing feature table against an already-trained EVA/pyEVA
  model — in that case run the classifier rather than rebuilding a training corpus.
- Your EIC plots are exclusively from a single sample type and LC/MS condition; the
  skill's value comes from curating plots across diverse sample types, LC and MS conditions.
- Ground-truth peak quality can be obtained from an objective chromatographic metric
  rather than human judgment — manual inspection of every plot is the labor-intensive
  step this skill mandates.
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: pyEVA
  role: Python environment in which the CNN is trained on the labeled EIC plots; also
    renders EIC plots from feature tables and .mzXML/.mzML files for annotation
  repo: https://github.com/HuanLab/pyEVA
- name: EVA
  role: C#-based program into which the trained CNN model is embedded to score new
    metabolic features; distribution target for the trained artifact
  repo: github.com/HuanLab-backup/EVA
- name: C#
  role: Language of the EVA Windows application that consumes the trained CNN model
    for feature fidelity evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c01309
    title: 'EVA: Evaluation of Metabolic Feature Fidelity Using a Deep Learning Model
      Trained With Over 25000 Extracted Ion Chromatograms'
  source_content_hash: sha256:bc88e5da2e2243f67efc6fa12562afc6f7b8d443297d6826315465d5073de278
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/pyEVA
  tools:
  - C#
  - EVA
  - pyEVA
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# lc-ms-metabolomics-training-data-preparation

## Summary

Prepares a manually labeled extracted-ion chromatogram (EIC) image dataset from LC-MS metabolomics projects to train a CNN classifier (as in EVA) that discriminates good versus poor chromatographic peak shapes, with the goal of flagging false-positive metabolic features.

## When to use

Use when building a deep-learning peak-quality classifier for LC-MS feature lists and you need labeled EIC plot images spanning diverse sample types, LC and MS conditions, with each plot manually inspected to assign good or poor quality labels.

## When NOT to use

- You only need to score an existing feature table against an already-trained EVA/pyEVA model — in that case run the classifier rather than rebuilding a training corpus.
- Your EIC plots are exclusively from a single sample type and LC/MS condition; the skill's value comes from curating plots across diverse sample types, LC and MS conditions.
- Ground-truth peak quality can be obtained from an objective chromatographic metric rather than human judgment — manual inspection of every plot is the labor-intensive step this skill mandates.

## Inputs

- EIC plots rendered from feature tables (m/z and rt columns) paired with .mzXML or .mzML raw files, aggregated across multiple LC-MS metabolomics projects (e.g. 25095 plots from 22 projects)
- Manual good/poor EIC quality labels for each plot, assigned by human inspection

## Outputs

- Manually annotated EIC image dataset with binary good/poor quality labels, suitable for supervised CNN training
- Trained CNN classifier artifact (e.g. cnn_model.h5) achieving >90% classification accuracy, ready for embedding into the EVA/pyEVA pipeline

## How to apply

Collect EIC plots generated from feature tables and associated .mzXML or .mzML files across multiple LC-MS-based metabolomics projects (the EVA paper used 25095 plots from 22 projects covering various sample types, LC and MS conditions). For each plot, manually inspect the chromatographic peak shape and assign a binary label of good or poor EIC quality — manual inspection of every plot is required for accurate model training, so do not rely on automatic heuristics to create ground truth. Train the CNN image classifier (in the pyEVA/EVA Python environment) on these labeled images for the binary good/poor task, then evaluate on held-out plots; the trained model reached over 90% classification accuracy and was subsequently embedded into a C#-based EVA program for scoring new feature tables. The labeled corpus also underpins the smoothing-variant EIC rendering used by pyEVA, where users pick a smoothing level (0, 1, or 2) before plots are output for classification.

## Related tools

- **pyEVA** (Python environment in which the CNN is trained on the labeled EIC plots; also renders EIC plots from feature tables and .mzXML/.mzML files for annotation) — https://github.com/HuanLab/pyEVA
- **EVA** (C#-based program into which the trained CNN model is embedded to score new metabolic features; distribution target for the trained artifact) — github.com/HuanLab-backup/EVA
- **C#** (Language of the EVA Windows application that consumes the trained CNN model for feature fidelity evaluation)

## Examples

```
python main.py  # in pyEVA environment; renders EIC plots for manual good/poor labeling and CNN training
```

## Evaluation signals

- Every EIC plot in the training corpus carries a binary good/poor label from manual inspection — count of unlabeled plots must be zero.
- Dataset scale and diversity match the article's reported scope: ~25095 EIC plots drawn from 22 LC-MS-based metabolomics projects spanning various sample types, LC and MS conditions.
- Trained CNN achieves over 90% classification accuracy on held-out labeled EIC plots, the performance criterion reported for EVA.
- Exported model artifact loads in the EVA (C#) or pyEVA (Python) pipeline and produces Boolean good/poor predictions consistent with manual labels on a held-out set.

## Limitations

- Manual inspection of every EIC plot is required for accurate model training, making corpus construction labor-intensive.
- Training data are specific to EIC peak-shape quality; plots must come from LC-MS metabolomics feature tables with matching .mzXML or .mzML raw files.
- EVA is distributed as a Windows application (Microsoft Store, search 'EVA Metabolomics'); cross-platform use requires the Python rewrite pyEVA.

## Evidence

- [readme] pyEVA rendering inputs for annotation: [evidence span withheld — anchor status: external_doc]
- [readme] Smoothing parameter choice before plot rendering: [evidence span withheld — anchor status: external_doc]
