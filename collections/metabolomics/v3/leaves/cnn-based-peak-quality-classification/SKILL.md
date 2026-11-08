---
name: cnn-based-peak-quality-classification
description: Use when QCing feature tables from LC-MS-based metabolomics runs where
  automated feature detection produces EICs that may include false positives, and
  manual peak-by-peak inspection is impractical.
when_to_use_negative:
- Input is not from LC-MS-based metabolomics feature detection — the model was trained
  only on EIC plots from 22 LC-MS metabolomics projects
- You need GC-MS or non-chromatographic peak quality assessment; EVA evaluates chromatographic
  peak quality for feature fidelity in LC-MS specifically
- Only a raw LC-MS file with no feature table is available — pyEVA requires a feature
  table (.csv with m/z and rt) plus matched raw data to render EICs
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3382
tools:
- name: pyEVA
  role: Python implementation that extracts EIC plots from feature tables/mzXML files
    and runs the embedded pretrained CNN to classify EIC quality; run via python main.py
  repo: https://github.com/HuanLab/pyEVA
- name: EVA
  role: Original C#-based Windows program embedding the trained CNN, distributed on
    the Microsoft Store as 'EVA Metabolomics'; pyEVA is its Python rewrite for Mac,
    PC, and Linux
  repo: https://github.com/HuanLab/pyEVA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c01309
    title: 'EVA: Evaluation of Metabolic Feature Fidelity Using a Deep Learning Model
      Trained With Over 25000 Extracted Ion Chromatograms'
  source_content_hash: sha256:c9b01192b1fe1ca73dee2bdde1c742a7d62cfbfa1e15c577f9444f8dcb2d50e9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/pyEVA
  tools:
  - EVA
  - pyEVA
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# cnn-based-peak-quality-classification

## Summary

Use a convolutional neural network trained on 25095 manually annotated extracted ion chromatogram (EIC) plots to automatically classify LC-MS metabolic features as good or poor EIC quality, flagging likely false positives for feature fidelity checking.

## When to use

Use when QCing feature tables from LC-MS-based metabolomics runs where automated feature detection produces EICs that may include false positives, and manual peak-by-peak inspection is impractical.

## When NOT to use

- Input is not from LC-MS-based metabolomics feature detection — the model was trained only on EIC plots from 22 LC-MS metabolomics projects
- You need GC-MS or non-chromatographic peak quality assessment; EVA evaluates chromatographic peak quality for feature fidelity in LC-MS specifically
- Only a raw LC-MS file with no feature table is available — pyEVA requires a feature table (.csv with m/z and rt) plus matched raw data to render EICs

## Inputs

- Feature table as .csv with columns m/z, rt, then sample data columns
- Raw LC-MS data as .mzXML or .mzML files
- Optional: smoothing level choice 0, 1, or 2

## Outputs

- EIC plots rendered per metabolic feature (EICplots folder under classifier)
- FinalTable.csv with analysis results
- PredictionOutcomes.csv with per-feature Boolean good/poor EIC classifications

## How to apply

Collect EIC plots for each metabolic feature in the feature table using pyEVA: place a .csv feature table (first two columns m/z and rt, followed by sample data columns) plus the matched .mzXML or .mzML files in the input folder, choose a smoothing level (0, 1, or 2), and run the EIC extraction so plots are written to the EICplots folder under classifier. The pretrained CNN — trained on 25095 manually labeled good/poor EIC plots from 22 LC-MS metabolomics projects spanning diverse sample types, LC and MS conditions — is embedded in pyEVA (the Python reimplementation of the C# EVA program) and scores each rendered EIC for chromatographic peak quality. Features whose EIC shapes the model rates as poor are likely low-fidelity, false-positive detections and can be filtered from downstream analyses. Collect the Boolean per-feature predictions and the results table from the output folder to drive the fidelity check.

## Related tools

- **pyEVA** (Python implementation that extracts EIC plots from feature tables/mzXML files and runs the embedded pretrained CNN to classify EIC quality; run via python main.py) — https://github.com/HuanLab/pyEVA
- **EVA** (Original C#-based Windows program embedding the trained CNN, distributed on the Microsoft Store as 'EVA Metabolomics'; pyEVA is its Python rewrite for Mac, PC, and Linux) — https://github.com/HuanLab/pyEVA

## Examples

```
python main.py  # feature table (.csv: m/z, rt, ...) and .mzXML/.mzML in input/; outputs EICplots/, FinalTable.csv, PredictionOutcomes.csv
```

## Evaluation signals

- The trained CNN achieves over 90% classification accuracy — extended inputs should reproduce this benchmark when compared against manually assigned good/poor labels
- PredictionOutcomes.csv must contain a Boolean value for every row of the input feature table (one prediction per metabolic feature)
- EICplots folder must contain one rendered EIC per feature, matching the m/z and rt entries of the input table
- Every EIC must receive a binary good/poor label; features labeled poor are expected to show poor EIC shapes characteristic of false positives
- Output FinalTable.csv should pair each feature's m/z and rt with its classification result

## Limitations

- Model accuracy was benchmarked at over 90% on training-distribution LC-MS metabolomics data; performance on new LC-MS vendor formats should be validated against manual labels rather than assumed
- Requires specific environment setup: Python 3.7.12 conda environment, R packages XCMS/ggplot2/scales/dplyr, and up-to-date PyCharm or VS Code versions — older versions may behave differently
- EIC rendering smoothing level (0, 1, 2) is a user choice that may affect classification inputs
- Per the README, no changelog is published, so version-to-version behavior of pyEVA is not documented

## Evidence

- [readme] Feature table input format: [evidence span withheld — anchor status: external_doc]
- [readme] Run command and outputs: [evidence span withheld — anchor status: external_doc]
