---
name: lipid-class-stratified-benchmarking
description: Use when you have predicted molecular fingerprints for a set of extracted
  MS/MS spectra and need to quantify annotation accuracy overall and per chemical
  class (e.g., lipid classes) against a reference database.
when_to_use_negative:
- The spectra are already annotated by library matching (e.g., against NIST 20 and
  LipidBlast) and you only need library-coverage statistics, not fingerprint-prediction
  accuracy.
- The lipid classes of interest have fewer than 50 spectra in the training data, so
  per-class accuracy estimates are not well-covered and the stratified breakdown is
  unreliable.
- The evaluation target is de novo structure elucidation or SMILES translation rather
  than top-hit fingerprint retrieval against a fixed reference database.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: IDSL_MINT
  role: Deep learning framework whose predicted molecular fingerprints are searched
    against the LipidMaps database to compute top-hit accuracies
  repo: https://github.com/idslme/IDSL_MINT
- name: IDSL.CSA
  role: R package used to extract the 3,386 ESI+ and 1,901 ESI− unique MS/MS spectra
    from study ST002044 that form the evaluation set
- name: PyTorch
  role: Python framework in which the IDSL_MINT transformer models producing the evaluated
    fingerprints are implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:628a3b20d17d3bb459540e53f9aaf78f2f722ef167b21852336177063c7ba1ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL.CSA
  - IDSL_MINT
  - PyTorch
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lipid-class-stratified-benchmarking

## Summary

Benchmark a fingerprint-based MS/MS annotation method by computing overall top-hit accuracy per ion mode and stratifying it by lipid class, as done for IDSL_MINT predictions on study ST002044 searched against the LipidMaps database.

## When to use

Use when you have predicted molecular fingerprints for a set of extracted MS/MS spectra and need to quantify annotation accuracy overall and per chemical class (e.g., lipid classes) against a reference database.

## When NOT to use

- The spectra are already annotated by library matching (e.g., against NIST 20 and LipidBlast) and you only need library-coverage statistics, not fingerprint-prediction accuracy.
- The lipid classes of interest have fewer than 50 spectra in the training data, so per-class accuracy estimates are not well-covered and the stratified breakdown is unreliable.
- The evaluation target is de novo structure elucidation or SMILES translation rather than top-hit fingerprint retrieval against a fixed reference database.

## Inputs

- Predicted molecular fingerprints for unique MS/MS spectra of the test study (ms2fp_st002044_prediction.zip, Zenodo file#5)
- Reference LipidMaps fingerprint database (ms2fp_lipidmaps.zip, Zenodo file#6)
- Metabolomics WorkBench study ST002044 MS/MS spectra in NIST .msp format
- Training-set class coverage table (spectra counts per lipid class, e.g., Table 1)

## Outputs

- Overall top-hit annotation accuracy per ion mode (ESI+ and ESI−)
- Per-lipid-class top-hit accuracy breakdown for classes with ≥ 50 training spectra
- accuracy_report.csv

## How to apply

Extract unique MS/MS spectra from the test study (e.g., 3,386 ESI+ and 1,901 ESI− spectra for ST002044 via IDSL.CSA) and obtain predicted fingerprint descriptors for each spectrum (file#5 ms2fp_st002044_prediction.zip from the Zenodo deposit). Load the reference fingerprint database built from the LipidMaps database (file#6 ms2fp_lipidmaps.zip) and, for each predicted fingerprint, search it against the reference database, marking an annotation as a true positive when the correct compound is the top hit candidate. Compute overall accuracy separately per ion mode as the fraction of true-positive top hits over evaluated spectra (reported values: 0.354 for ESI+ and 0.351 for ESI−). Then stratify top-hit accuracy by lipid class, restricting the breakdown to well-covered classes with at least 50 spectra in the training data (6 of 8 lipid classes), and report classes such as glycerophospholipids (expected ≥ 0.50) and fatty acyls (expected ~0.20). Write the overall and per-class accuracies to accuracy_report.csv.

## Related tools

- **IDSL_MINT** (Deep learning framework whose predicted molecular fingerprints are searched against the LipidMaps database to compute top-hit accuracies) — https://github.com/idslme/IDSL_MINT
- **IDSL.CSA** (R package used to extract the 3,386 ESI+ and 1,901 ESI− unique MS/MS spectra from study ST002044 that form the evaluation set)
- **PyTorch** (Python framework in which the IDSL_MINT transformer models producing the evaluated fingerprints are implemented)

## Examples

```
MINT_workflow --yaml /path/to/ms2fp_st002044_prediction.yaml
```

## Evaluation signals

- Recomputed overall accuracies match the reported values: 0.354 for ESI+ and 0.351 for ESI−.
- Per-class breakdown reproduces the reported pattern: glycerophospholipids ≥ 0.50 and fatty acyls ~0.20.
- Stratified analysis is restricted to the 6 well-covered lipid classes with at least 50 spectra in the training data.
- True positives are defined consistently as top hit candidates from searching predicted fingerprints against the LipidMaps database.
- The evaluated spectra counts equal the extracted unique spectra (3,386 ESI+ and 1,901 ESI− for ST002044).

## Limitations

- The case-study training set covered only 6.96% (ESI+) and 4.58% (ESI−) of all compounds in the LipidMaps database, so accuracy is conditional on reference coverage.
- Only 6 of 8 lipid classes had at least 50 training spectra, so per-class accuracy for rare classes cannot be reliably reported.
- Accuracy is defined strictly as the correct compound being the top hit; lower-ranked correct hits are not credited.

## Evidence

- [results] Overall top-hit accuracy for case study with the true positive being the top hit candidate when the predicted fingerprint was searched against the LipidMaps database.: "where true positive being the top hit candidate when the predicted fingerprint was searched against the LipidMaps database."
- [results] Class coverage filter for stratification: "Training data has 8 lipid classes and 6 classes having at least 50 spectra (Table 1)."
- [results] Extraction of the evaluation spectra for ST002044: "The IDSL.CSA R package [2] was used to extract 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− for this study"
- [abstract] Reference database for the fingerprint search: "IDSL_MINT was benchmarked using the LipidMaps database and improved the annotation rate of a test study"
- [readme] Fingerprint search capability grounding in README: "IDSL_MINT has been meticulously engineered to predict molecular fingerprint descriptors and structures from MS/MS spectra"
