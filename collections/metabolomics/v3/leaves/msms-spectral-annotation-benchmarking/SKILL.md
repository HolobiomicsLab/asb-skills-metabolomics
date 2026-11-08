---
name: msms-spectral-annotation-benchmarking
description: Use when you have model-predicted fingerprints for unlabeled MS/MS spectra
  (e.g., .msp-derived ESI+/ESI− predictions) plus a reference fingerprint database
  (LipidMaps) and need to quantify top-hit annotation accuracy, overall and stratified
  by lipid class.
when_to_use_negative:
- The goal is routine annotation coverage via existing mass spectral libraries — the
  paper annotates library matches separately by matching spectra against NIST 20 and
  LipidBlast (638/3386 ESI+ and 299/1901 ESI−), which is a different step from fingerprint-model
  benchmarking
- The chemical classes of interest have fewer than 50 training spectra — per-class
  accuracy estimates are restricted to well-covered classes (6 of 8 lipid classes)
- Input predictions are already structure annotations (e.g., SMILES top hits with
  Tanimoto ≥ 0.6) rather than predicted fingerprint vectors to be searched against
  a reference database
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL_MINT
  role: Deep learning framework whose ESI+ and ESI− models produce the predicted fingerprints
    that are benchmarked against LipidMaps
  repo: https://github.com/idslme/IDSL_MINT
- name: IDSL.CSA
  role: R package used to extract the 3,386 and 1,901 unique MS/MS spectra in ESI+
    and ESI− from study ST002044 as the evaluation set
- name: LipidMaps database
  role: Reference fingerprint database searched by predicted fingerprints; top-hit
    correctness defines true positives
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:6f33c7b0ba52432d7a630b0ee76d7e61cc3579ba684b5437e2fda544478eb8df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL.CSA
  - IDSL_MINT
  - LipidMaps database
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# msms-spectral-annotation-benchmarking

## Summary

Benchmark top-hit annotation accuracy of MS/MS-to-fingerprint deep learning models (IDSL_MINT) by searching predicted fingerprints against a reference structure database (LipidMaps), reporting overall and per-chemical-class accuracy per ion mode.

## When to use

Use when you have model-predicted fingerprints for unlabeled MS/MS spectra (e.g., .msp-derived ESI+/ESI− predictions) plus a reference fingerprint database (LipidMaps) and need to quantify top-hit annotation accuracy, overall and stratified by lipid class.

## When NOT to use

- The goal is routine annotation coverage via existing mass spectral libraries — the paper annotates library matches separately by matching spectra against NIST 20 and LipidBlast (638/3386 ESI+ and 299/1901 ESI−), which is a different step from fingerprint-model benchmarking
- The chemical classes of interest have fewer than 50 training spectra — per-class accuracy estimates are restricted to well-covered classes (6 of 8 lipid classes)
- Input predictions are already structure annotations (e.g., SMILES top hits with Tanimoto ≥ 0.6) rather than predicted fingerprint vectors to be searched against a reference database

## Inputs

- Predicted molecular fingerprints for unique MS/MS spectra in ESI+ and ESI− (Zenodo file#5 ms2fp_st002044_prediction.zip)
- Reference LipidMaps fingerprint database (Zenodo file#6 ms2fp_lipidmaps.zip)
- Per-spectrum lipid class labels / training-data class counts to identify classes with ≥50 spectra
- Study ST002044 MS/MS spectra extracted via IDSL.CSA in NIST .msp format

## Outputs

- accuracy_report.csv with overall top-hit accuracy for ESI+ and ESI−
- Per-lipid-class accuracy breakdown restricted to classes with ≥50 training spectra (including glycerophospholipids and fatty acyls)

## How to apply

Download and unzip the Zenodo deposit files: file#5 ms2fp_st002044_prediction.zip (predicted fingerprints for 3,386 ESI+ and 1,901 ESI− unique MS/MS spectra of study ST002044) and file#6 ms2fp_lipidmaps.zip (reference LipidMaps fingerprints as the true-positive database). For each predicted fingerprint, search it against the LipidMaps database as in IDSL_MINT's evaluation procedure and mark the annotation a true positive when the correct compound is the top hit candidate. Compute overall accuracy separately for positive and negative ion modes as the fraction of true-positive top hits over the evaluated spectra. Stratify top-hit accuracy by lipid class, restricting the breakdown to well-covered classes with at least 50 spectra in the training data (training data has 8 lipid classes, 6 with ≥50 spectra). Write overall (ESI+ and ESI−) and per-lipid-class accuracies to accuracy_report.csv.

## Related tools

- **IDSL_MINT** (Deep learning framework whose ESI+ and ESI− models produce the predicted fingerprints that are benchmarked against LipidMaps) — https://github.com/idslme/IDSL_MINT
- **IDSL.CSA** (R package used to extract the 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− from study ST002044 as the evaluation set)
- **LipidMaps database** (Reference fingerprint database searched by predicted fingerprints; top-hit correctness defines true positives)

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- Recomputed overall top-hit accuracies must match reported values: 0.354 for ESI+ and 0.351 for ESI−
- Per-class breakdown must reproduce the reported pattern: glycerophospholipids ≥ 0.50 and fatty acyls ≈ 0.20
- Class stratification restricted to classes with at least 50 spectra in training data (6 of 8 lipid classes reported)
- Denominators must equal the extracted spectrum counts: 3,386 unique ESI+ and 1,901 unique ESI− MS/MS spectra
- True positives defined identically: correct compound is the top hit candidate when the predicted fingerprint is searched against the LipidMaps database

## Limitations

- Accuracy is top-hit-based only; annotation rate improvements elsewhere in the paper used a Tanimoto similarity ≥ 0.6 high-confidence threshold, a different criterion
- Per-class accuracies are unreliable for classes with <50 training spectra; 2 of the 8 lipid classes are excluded
- Benchmarking depends on the training set composition — the case study training set included only 6.96% (ESI+) and 4.58% (ESI−) of all LipidMaps compounds, so accuracy is not comparable to a full-library benchmark
- Fingerprint prediction performance is modality- and mode-specific: ESI+ and ESI− models are trained and evaluated separately

## Evidence

- [results] True-positive definition: "where true positive being the top hit candidate when the predicted fingerprint was searched against the LipidMaps database."
- [results] Evaluation of overall and per-class accuracy: "Next, we evaluated the overall accuracy and the accuracy by lipid classes for the test study ST002044."
- [results] Class coverage cutoff: "Training data has 8 lipid classes and 6 classes having at least 50 spectra (Table 1)."
- [results] Evaluation set sizes: "The IDSL.CSA R package [2] was used to extract 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− for this study"
- [readme] Fingerprint prediction capability being benchmarked: "IDSL_MINT includes a method to translate MS/MS spectra into molecular fingerprint descriptors."
