---
name: tanimoto-top-hit-search-evaluation
description: 'Use when benchmarking MS/MS-to-fingerprint models on an unannotated
  study: you need to convert predicted fingerprints into annotation accuracy against
  a fingerprint reference database, overall and per chemical class.'
when_to_use_negative:
- When the evaluation target is library spectral matching rather than fingerprint
  search — the article uses spectral similarity matching against NIST 20 and LipidBlast
  (spectral entropy ≥ 0.75) as a separate annotation-coverage step, not this top-hit
  fingerprint evaluation
- When chemical classes of interest have fewer than 50 spectra in the training data
  — the article restricts the per-class breakdown to the 6 well-covered classes out
  of 8
- When a compound is already covered in LipidBlast or NIST 20 libraries, since the
  case-study training set deliberately excluded those spectra to avoid circular evaluation
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: IDSL_MINT
  role: Predicts the molecular fingerprints from MS/MS spectra whose top-hit search
    accuracy is being evaluated; its evaluation procedure defines the true-positive
    criterion
  repo: https://github.com/idslme/IDSL_MINT
- name: IDSL.CSA
  role: Extracted the unique MS/MS spectra (3,386 ESI+ / 1,901 ESI−) of study ST002044
    that serve as the evaluation set
- name: RDKit
  role: Converts InChI/SMILES entries into canonical SMILES and computes the molecular
    fingerprint descriptors (e.g., ECFPs or MACCS Keys) forming the searched database
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:e8506a1dd6b4260127ab700ca263e3a8d7030c3e08ae3d3592a6d7338d63d2ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL.CSA
  - IDSL_MINT
  - RDKit
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Tanimoto top-hit search evaluation

## Summary

Evaluates fingerprint-prediction annotation accuracy by searching predicted molecular fingerprints against a reference fingerprint database (e.g., LipidMaps) and scoring a true positive when the correct compound is the top hit, reporting overall and per-class accuracy.

## When to use

Use when benchmarking MS/MS-to-fingerprint models on an unannotated study: you need to convert predicted fingerprints into annotation accuracy against a fingerprint reference database, overall and per chemical class.

## When NOT to use

- When the evaluation target is library spectral matching rather than fingerprint search — the article uses spectral similarity matching against NIST 20 and LipidBlast (spectral entropy ≥ 0.75) as a separate annotation-coverage step, not this top-hit fingerprint evaluation
- When chemical classes of interest have fewer than 50 spectra in the training data — the article restricts the per-class breakdown to the 6 well-covered classes out of 8
- When a compound is already covered in LipidBlast or NIST 20 libraries, since the case-study training set deliberately excluded those spectra to avoid circular evaluation

## Inputs

- Predicted molecular fingerprints from IDSL_MINT for unique MS/MS spectra (e.g., file#5 ms2fp_st002044_prediction.zip: 3,386 ESI+ and 1,901 ESI− spectra of ST002044)
- Reference fingerprint database from LipidMaps (file#6 ms2fp_lipidmaps.zip)
- Lipid-class assignments for reference compounds and per-class training-spectrum counts (≥50 spectra coverage filter)

## Outputs

- Overall top-hit annotation accuracy per ion mode (reported: 0.354 ESI+, 0.351 ESI−)
- Per-lipid-class top-hit accuracy breakdown (glycerophospholipids ≥ 0.50, fatty acyls ~0.20)
- accuracy_report.csv with overall (ESI+ and ESI−) and per-class accuracies

## How to apply

Take the IDSL_MINT-predicted molecular fingerprints for the study's unique MS/MS spectra (e.g., 3,386 ESI+ and 1,901 ESI− spectra of ST002044 from file#5 ms2fp_st002044_prediction.zip) and load the reference fingerprint database built from LipidMaps (file#6 ms2fp_lipidmaps.zip). For each predicted fingerprint, search it against the reference database exactly as in IDSL_MINT's evaluation procedure and mark the annotation as a true positive when the correct compound is the top hit candidate. Compute overall accuracy separately per ion mode (ESI+ and ESI−) as the fraction of true-positive top hits over the evaluated spectra. Then stratify accuracy by lipid class, restricting the breakdown to well-covered classes with at least 50 spectra in the training data (the training data had 8 lipid classes, 6 with ≥50 spectra), and write overall and per-class values to accuracy_report.csv. Validate by checking recomputed overall accuracy against reported values (0.354 ESI+ / 0.351 ESI−) and the per-class pattern (glycerophospholipids ≥ 0.50, fatty acyls ~0.20).

## Related tools

- **IDSL_MINT** (Predicts the molecular fingerprints from MS/MS spectra whose top-hit search accuracy is being evaluated; its evaluation procedure defines the true-positive criterion) — https://github.com/idslme/IDSL_MINT
- **IDSL.CSA** (Extracted the unique MS/MS spectra (3,386 ESI+ / 1,901 ESI−) of study ST002044 that serve as the evaluation set)
- **RDKit** (Converts InChI/SMILES entries into canonical SMILES and computes the molecular fingerprint descriptors (e.g., ECFPs or MACCS Keys) forming the searched database) — https://www.rdkit.org/docs/Install.html

## Examples

```
MINT_workflow --yaml /path/to/ms2fp_yaml/file  # then search predicted fingerprints against ms2fp_lipidmaps.zip top hits and write accuracy_report.csv
```

## Evaluation signals

- Recomputed overall top-hit accuracy must match reported values: 0.354 for ESI+ and 0.351 for ESI−
- Per-class breakdown must reproduce the reported pattern: glycerophospholipids ≥ 0.50 and fatty acyls ~0.20
- True positives are counted only when the correct compound is the top hit candidate of the LipidMaps fingerprint search — not within top-k
- Per-class accuracy must be reported only for classes with at least 50 spectra in the training data (6 of 8 lipid classes)
- Spectrum counts entering the evaluation must equal the extracted unique spectra counts (3,386 ESI+ and 1,901 ESI− from ST002044)

## Limitations

- The case-study training set covered only 6.96% (ESI+) and 4.58% (ESI−) of all LipidMaps compounds, limiting generalization to classes outside the training data
- Per-class accuracy is undefined for poorly covered classes — only classes with ≥50 training spectra are reported
- Top-hit criterion is strict: a correct compound ranked below first place counts as a false negative
- Reference evaluation depends on the completeness of the fingerprint database (LipidMaps); annotations outside it cannot be recovered

## Evidence

- [results] True-positive top-hit criterion: "where true positive being the top hit candidate when the predicted fingerprint was searched against the LipidMaps database."
- [results] Per-class coverage filter: "Training data has 8 lipid classes and 6 classes having at least 50 spectra (Table 1)."
- [results] Case-study spectrum extraction: "The IDSL.CSA R package [2] was used to extract 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− for this study"
- [results] Evaluation of overall and per-class accuracy: "Next, we evaluated the overall accuracy and the accuracy by lipid classes for the test study ST002044."
- [results] Training-set coverage of LipidMaps: "Our case study training set included 6.96% and 4.58% of all the compounds in the LipidMaps database in positive and negative modes, respectively"
