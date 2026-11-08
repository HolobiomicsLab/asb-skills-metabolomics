---
name: fingerprint-annotation-accuracy-evaluation
description: Use when predicted molecular fingerprints (e.g. from IDSL_MINT models
  on .msp spectra) must be benchmarked against a reference fingerprint database to
  compute overall and per-lipid-class top-hit annotation accuracy.
when_to_use_negative:
- Spectra already match an existing spectral library (e.g. NIST 20 or LipidBlast)
  — the paper excluded compounds covered by those libraries and evaluates only spectra
  without existing annotations
- The goal is qualitative compound identification rather than benchmarking predicted-fingerprint
  ranking accuracy
- Chemical classes have fewer than 50 training spectra, so the per-class accuracy
  breakdown is not well covered
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: IDSL_MINT
  role: Produces the predicted molecular fingerprints from MS/MS spectra that are
    benchmarked in this evaluation
  repo: https://github.com/idslme/IDSL_MINT
- name: IDSL.CSA
  role: Extracted the unique MS/MS spectra in ESI+/ESI− (.msp format) for study ST002044
    that form the evaluation set
- name: LipidMaps
  role: Reference fingerprint database against which predicted fingerprints are searched
    to define top-hit true positives
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:b1640259db9e67da02a05aaf42fd75e31460649cb5cde67c14f8b6634bb7f5fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL.CSA
  - IDSL_MINT
  - LipidMaps
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Fingerprint annotation accuracy evaluation

## Summary

Evaluates the annotation accuracy of MS/MS-derived predicted molecular fingerprints by searching each predicted fingerprint vector against a reference fingerprint database (e.g. LipidMaps) and scoring top-hit true positives, overall and per chemical class.

## When to use

Use when predicted molecular fingerprints (e.g. from IDSL_MINT models on .msp spectra) must be benchmarked against a reference fingerprint database to compute overall and per-lipid-class top-hit annotation accuracy.

## When NOT to use

- Spectra already match an existing spectral library (e.g. NIST 20 or LipidBlast) — the paper excluded compounds covered by those libraries and evaluates only spectra without existing annotations
- The goal is qualitative compound identification rather than benchmarking predicted-fingerprint ranking accuracy
- Chemical classes have fewer than 50 training spectra, so the per-class accuracy breakdown is not well covered

## Inputs

- Predicted molecular fingerprint vectors for MS/MS spectra (IDSL_MINT output, e.g. ms2fp_st002044_prediction.zip with ESI+ and ESI− spectra)
- Reference fingerprint database with true fingerprints and compound/lipid-class labels (e.g. ms2fp_lipidmaps.zip from LipidMaps, .msp-based)
- Ion-mode labels (ESI+/ESI−) and lipid-class annotations for the spectra

## Outputs

- Overall top-hit annotation accuracy per ion mode (e.g. 0.354 ESI+ / 0.351 ESI− for ST002044)
- Per-lipid-class top-hit accuracy table (well-covered classes with ≥50 training spectra, e.g. glycerophospholipids ≥0.50, fatty acyls ~0.20)
- accuracy_report.csv aggregating overall and per-class results

## How to apply

Load the predicted fingerprints for the test spectra (e.g. IDSL_MINT predictions for the 3,386 ESI+ and 1,901 ESI− unique MS/MS spectra of ST002044, from Zenodo file#5 ms2fp_st002044_prediction.zip) and the reference fingerprint database (file#6 ms2fp_lipidmaps.zip). For each predicted fingerprint, search it against the reference database and mark the annotation as a true positive when the correct compound is the top hit candidate — this is the paper's explicit true-positive definition. Compute overall accuracy per ion mode (ESI+ and ESI−) as the fraction of true-positive top hits over evaluated spectra; the case study yielded 0.354 (ESI+) and 0.351 (ESI−). Then stratify accuracy by chemical class, restricting the breakdown to well-covered classes with at least 50 spectra in the training data (8 lipid classes total, 6 well-covered). Write overall and per-class accuracies to a report file such as accuracy_report.csv. Rationale: predicted fingerprints cannot be assessed by exact SMILES match, so ranking the true compound first in fingerprint space is the evaluation proxy.

## Related tools

- **IDSL_MINT** (Produces the predicted molecular fingerprints from MS/MS spectra that are benchmarked in this evaluation) — https://github.com/idslme/IDSL_MINT
- **IDSL.CSA** (Extracted the unique MS/MS spectra in ESI+/ESI− (.msp format) for study ST002044 that form the evaluation set)
- **LipidMaps** (Reference fingerprint database against which predicted fingerprints are searched to define top-hit true positives)

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- Recomputed overall accuracy must equal the reported values: 0.354 for ESI+ and 0.351 for ESI− on ST002044
- Per-class breakdown must reproduce the reported pattern: glycerophospholipids ≥ 0.50 and fatty acyls ~0.20
- Class breakdown restricted to the 6 well-covered lipid classes (of 8 total) each having at least 50 training spectra
- True positives defined identically to the paper: correct compound as top hit when predicted fingerprint is searched against LipidMaps
- Denominators must match the extracted spectra counts: 3,386 ESI+ and 1,901 ESI− unique MS/MS spectra

## Limitations

- Case-study training set covered only 6.96% (ESI+) and 4.58% (ESI−) of all LipidMaps compounds, so accuracy is relative to a small reference subset
- Classes with fewer than 50 training spectra are excluded from the per-class breakdown, limiting coverage of rare classes
- Accuracy depends on the top-hit ranking definition; lower-ranked correct candidates are not counted as true positives

## Evidence

- [results] Top-hit evaluation and LipidMaps search definition: "where true positive being the top hit candidate when the predicted fingerprint was searched against the LipidMaps database."
- [results] Overall and per-class accuracy evaluation for ST002044: "Next, we evaluated the overall accuracy and the accuracy by lipid classes for the test study ST002044."
- [results] Class coverage filter for the breakdown: "Training data has 8 lipid classes and 6 classes having at least 50 spectra (Table 1)."
- [readme] Fingerprint evaluation inputs come from .msp-compatible reference data: "A key distinguishing feature of **IDSL_MINT** is its compatibility with any reference MS/MS data in ***.msp*** format"
