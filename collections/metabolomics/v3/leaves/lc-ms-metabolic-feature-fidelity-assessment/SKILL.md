---
name: lc-ms-metabolic-feature-fidelity-assessment
description: Use when a feature table contains suspect LC-MS features whose EIC shapes
  may be false positives, and you need automated good/poor peak-quality classification
  before downstream statistical analysis.
when_to_use_negative:
- Features are already validated with manually inspected good/poor EIC labels — the
  CNN adds no information
- Data are not LC-MS-based metabolomics measurements from which EIC plots can be extracted
- You need quantification rather than EIC shape fidelity assessment
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: EVA
  role: C#-based Windows application embedding the trained CNN to evaluate chromatographic
    peak quality for feature fidelity
  repo: github.com/HuanLab-backup/EVA
- name: pyEVA
  role: Python version of EVA for multi-platform automated classification of EIC plots
    as true or false
  repo: https://github.com/HuanLab/pyEVA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c01309
    title: 'EVA: Evaluation of Metabolic Feature Fidelity Using a Deep Learning Model
      Trained With Over 25000 Extracted Ion Chromatograms'
  source_content_hash: sha256:556ea277c05847e32987382f8688e79a219f7283c3d85d1e839f30a31f5e2b7a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - EVA
  - pyEVA
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# lc-ms-metabolic-feature-fidelity-assessment

## Summary

Assess metabolic feature fidelity in LC-MS-based metabolomics by classifying extracted ion chromatogram (EIC) peak quality with EVA, a convolutional neural network trained on over 25000 manually annotated EIC plots.

## When to use

Use when a feature table contains suspect LC-MS features whose EIC shapes may be false positives, and you need automated good/poor peak-quality classification before downstream statistical analysis.

## When NOT to use

- Features are already validated with manually inspected good/poor EIC labels — the CNN adds no information
- Data are not LC-MS-based metabolomics measurements from which EIC plots can be extracted
- You need quantification rather than EIC shape fidelity assessment

## Inputs

- Feature table (.csv) with m/z and rt columns for each metabolic feature
- LC-MS raw data files (.mzXML or .mzML) from which EIC plots are extracted

## Outputs

- Per-feature EIC plot images in the EICplots folder under classifier
- FinalTable.csv with analysis results
- PredictionOutcomes.csv containing Boolean good/poor peak-quality labels for each input feature

## How to apply

Export EIC plots from the LC-MS raw data (.mzXML or .mzML) for each feature (m/z, rt) in the feature table. Run the EVA CNN classifier — either the C# Windows application available on the Microsoft Store by searching "EVA Metabolomics" or the Python version pyEVA (github.com/HuanLab/pyEVA) — to score each plot as good or poor EIC quality. The model was trained on 25095 manually annotated EIC plots from 22 LC-MS-based metabolomics projects and achieves over 90% classification accuracy. Discard features whose EIC shapes are classified as poor, since these are of low feature fidelity and likely false positives. In pyEVA, place the feature table .csv (first two columns m/z and rt) and the .mzXML/.mzML files into the input folder, run `python main.py`, and collect Boolean good/poor predictions in PredictionOutcomes.csv.

## Related tools

- **EVA** (C#-based Windows application embedding the trained CNN to evaluate chromatographic peak quality for feature fidelity) — github.com/HuanLab-backup/EVA
- **pyEVA** (Python version of EVA for multi-platform automated classification of EIC plots as true or false) — https://github.com/HuanLab/pyEVA

## Examples

```
python main.py
```

## Evaluation signals

- Overall classification accuracy across the evaluated EIC plot set exceeds 90%, matching the reported comprehensive evaluation
- Every feature in the input feature table receives a Boolean good/poor prediction in PredictionOutcomes.csv
- EIC plots are generated for all input m/z/rt pairs and deposited in the EICplots folder under classifier
- FinalTable.csv is produced with analysis results consistent with the PredictionOutcomes.csv labels

## Limitations

- Classification accuracy is over 90% but not perfect — some false EIC classifications will occur
- The C# EVA application is Windows-only; Python users must use pyEVA and manage a conda environment (Python 3.7) plus R packages xcms, ggplot2, scales, and dplyr
- Older versions of PyCharm or VS Code may behave differently from the tested setup

## Evidence

- [readme] pyEVA input format and run command.: [evidence span withheld — anchor status: not_found]
- [readme] EIC plots output location.: [evidence span withheld — anchor status: external_doc]
