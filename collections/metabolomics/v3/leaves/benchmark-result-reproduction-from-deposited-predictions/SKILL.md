---
name: benchmark-result-reproduction-from-deposited-predictions
description: Use when a paper's headline accuracy numbers are tied to deposited prediction
  files (e.g. Zenodo file#5) and you must independently verify the reported metrics
  rather than retrain the model or re-run inference.
when_to_use_negative:
- You need to reproduce model training or inference itself — use the IDSL_MINT yaml-driven
  training workflow with file#4 ms2fp_yaml.zip instead of evaluating deposited predictions
- The benchmark metric is spectral library matching (NIST 20 / LipidBlast hits via
  IDSL.CSA) rather than fingerprint-search accuracy — that coverage analysis uses
  file#1 csa_spreadsheet.zip
- Your spectra or predicted fingerprints are not from the deposited ST002044 case
  study, since the validation anchors (0.354/0.351) are specific to that study
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: IDSL_MINT
  role: Framework whose deposited predicted fingerprints are re-scored; its evaluation
    procedure defines the fingerprint search against LipidMaps
  repo: https://github.com/idslme/IDSL_MINT
- name: LipidMaps database
  role: Reference fingerprint database searched by predicted fingerprints; top-hit
    defines true positives
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:89f8bd5d02893be4b69ae60e675cecf23c1037ed4f3f93b75a2d0d446d8d448a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL_MINT
  - LipidMaps database
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Benchmark-result reproduction from deposited predictions

## Summary

Reproduce a published benchmark table by taking authors' deposited model predictions (Zenodo archives) and re-running the evaluation procedure — here, searching IDSL_MINT-predicted fingerprints for ST002044 against the LipidMaps fingerprint database to recover the reported overall top-hit accuracies (0.354 ESI+ / 0.351 ESI−) and per-lipid-class breakdown.

## When to use

Use when a paper's headline accuracy numbers are tied to deposited prediction files (e.g. Zenodo file#5) and you must independently verify the reported metrics rather than retrain the model or re-run inference.

## When NOT to use

- You need to reproduce model training or inference itself — use the IDSL_MINT yaml-driven training workflow with file#4 ms2fp_yaml.zip instead of evaluating deposited predictions
- The benchmark metric is spectral library matching (NIST 20 / LipidBlast hits via IDSL.CSA) rather than fingerprint-search accuracy — that coverage analysis uses file#1 csa_spreadsheet.zip
- Your spectra or predicted fingerprints are not from the deposited ST002044 case study, since the validation anchors (0.354/0.351) are specific to that study

## Inputs

- Zenodo file#5 ms2fp_st002044_prediction.zip (IDSL_MINT-predicted fingerprints for 3,386 ESI+ and 1,901 ESI− ST002044 spectra)
- Zenodo file#6 ms2fp_lipidmaps.zip (reference LipidMaps fingerprint database)
- Per-spectrum lipid-class labels from the ST002044 study (Metabolomics WorkBench) for stratification

## Outputs

- accuracy_report.csv with overall ESI+/ESI− top-hit accuracies
- Per-lipid-class top-hit accuracy table (classes with ≥ 50 training spectra)

## How to apply

Download and unzip the deposited prediction archive (file#5 ms2fp_st002044_prediction.zip) containing predicted fingerprints for the 3,386 ESI+ and 1,901 ESI− unique MS/MS spectra of study ST002044, and load the reference fingerprint database from file#6 ms2fp_lipidmaps.zip. For each predicted fingerprint, search against the LipidMaps database exactly as in the paper's evaluation and mark a true positive when the correct compound is the top hit candidate. Compute overall accuracy per ion mode as the fraction of true-positive top hits over evaluated spectra, then stratify the top-hit accuracy by lipid class, restricting to classes with at least 50 training spectra (6 well-covered classes out of 8). Write overall and per-class accuracies to accuracy_report.csv and compare against reported values: overall 0.354 (ESI+) and 0.351 (ESI−), glycerophospholipids ≥ 0.50, fatty acyls ~0.20. Retraining is unnecessary because inference outputs are deposited; only the search-and-scoring step needs re-execution.

## Related tools

- **IDSL_MINT** (Framework whose deposited predicted fingerprints are re-scored; its evaluation procedure defines the fingerprint search against LipidMaps) — https://github.com/idslme/IDSL_MINT
- **LipidMaps database** (Reference fingerprint database searched by predicted fingerprints; top-hit defines true positives)

## Examples

```
unzip ms2fp_st002044_prediction.zip && python search_fingerprints.py --predictions ms2fp_st002044_prediction --database ms2fp_lipidmaps --modes ESI+,ESI- --out accuracy_report.csv
```

## Evaluation signals

- Recomputed ESI+ overall accuracy equals 0.354
- Recomputed ESI− overall accuracy equals 0.351
- Glycerophospholipid class accuracy ≥ 0.50 and fatty acyls ~0.20 in the per-class breakdown
- Number of evaluated spectra matches 3,386 (ESI+) and 1,901 (ESI−) unique MS/MS spectra
- Per-class breakdown restricted to the 6 lipid classes with at least 50 training spectra

## Limitations

- Accuracy is defined only as top-hit candidate when the predicted fingerprint is searched against LipidMaps; different search definitions will not reproduce the reported numbers
- The training set covered only 6.96% (ESI+) and 4.58% (ESI−) of all LipidMaps compounds, so per-class accuracies are only reported for well-covered classes
- Case-study numbers depend on excluding compounds already covered by LipidBlast and NIST 20 libraries during training-data compilation

## Evidence

- [results] Overall accuracy definition and top-hit criterion: "where true positive being the top hit candidate when the predicted fingerprint was searched against the LipidMaps database."
- [results] Deposited prediction and reference archives: "provided at Zenodo entry [27](file#6 ms2fp_lipidmaps.zip and file#7 ms2fp_idsl.csa.zip)."
- [results] Spectra counts for the case study: "The IDSL.CSA R package [2] was used to extract 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− for this study"
- [results] Class coverage constraint for stratification: "Training data has 8 lipid classes and 6 classes having at least 50 spectra (Table 1)."
- [results] Evaluation scope statement: "Next, we evaluated the overall accuracy and the accuracy by lipid classes for the test study ST002044."
