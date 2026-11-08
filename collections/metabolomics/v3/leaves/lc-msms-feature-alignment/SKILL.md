---
name: lc-msms-feature-alignment
description: Use when aligning per-sample precursor (MS1) and fragment (MS2) m/z/intensity
  feature lists exported from MS-DIAL across many LC-MS/MS samples into one feature
  table for downstream multivariate or machine-learning analysis.
when_to_use_negative:
- Input is already a single merged feature table or aligned matrix — re-alignment
  would be redundant
- The task is direct bioactivity prediction from raw spectra without alignment — use
  the image-based CNN pipeline instead, which bypasses alignment
- Sample set is small or has distinct features with no expected cross-sample duplicates
  — the 0.8 similarity threshold would incorrectly merge distinct features
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS-DIAL
  role: Preprocesses raw LC-MS/MS data and exports per-sample feature lists (MS1/MS2
    m/z pairs and intensities) that serve as the alignment input
- name: pandas 2.2.2
  role: Implements the feature-merging pipeline and stores the two merged feature
    tables
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:aecde0565e1723715e566e872b5d894282e8f9dbba6d7910f6ea47b9d9120619
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MS-DIAL
  - Python 3.12
  - pandas
  - pandas 2.2.2
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lc-msms-feature-alignment

## Summary

Merges per-sample LC-MS/MS feature lists from 545 wild soybean accessions into a unified feature table by scoring MS1 m/z, retention time, and MS/MS spectral similarity, treating features as equivalent when the average of the three scores exceeds 0.8 with a 1-minute RT tolerance.

## When to use

Use when aligning per-sample precursor (MS1) and fragment (MS2) m/z/intensity feature lists exported from MS-DIAL across many LC-MS/MS samples into one feature table for downstream multivariate or machine-learning analysis.

## When NOT to use

- Input is already a single merged feature table or aligned matrix — re-alignment would be redundant
- The task is direct bioactivity prediction from raw spectra without alignment — use the image-based CNN pipeline instead, which bypasses alignment
- Sample set is small or has distinct features with no expected cross-sample duplicates — the 0.8 similarity threshold would incorrectly merge distinct features

## Inputs

- Per-sample MS-DIAL feature lists from 545 wild soybean (Glycine soja) extract LC-MS/MS runs, each containing precursor MS1 m/z, retention time, and MS/MS spectra with intensities
- Adduct-filtered, TIC-normalized peak intensity values > 10^-4

## Outputs

- Merged unified feature table (one row per aligned feature across all 545 accessions) based on cosine MS/MS similarity
- Second merged feature table variant based on spectral-entropy MS/MS similarity

## How to apply

Load MS-DIAL-exported per-sample feature lists containing precursor MS1 m/z, retention time, and MS/MS spectra for all 545 wild soybean accessions. Compute an MS1 similarity score using an exponential formula with sigma set to the empirical standard deviation of observed MS1 values, an RT similarity score with sigma equal to the 1-minute alignment tolerance, and an MS/MS spectral similarity score using both cosine similarity and spectral entropy (treating spectra as probability distributions). For each cross-sample feature pair, average the three similarity scores; when the average exceeds 0.8, merge the features into a single row, otherwise append as a distinct feature. Implement in Python 3.12 with pandas 2.2.2, producing two variants of the merged table (cosine-based and entropy-based) to evaluate downstream Random Forest / multivariate classification impact.

## Related tools

- **MS-DIAL** (Preprocesses raw LC-MS/MS data and exports per-sample feature lists (MS1/MS2 m/z pairs and intensities) that serve as the alignment input)
- **pandas 2.2.2** (Implements the feature-merging pipeline and stores the two merged feature tables)

## Examples

```
python merge_features.py --mzml raw/*.mzML --rt-tol 1 --sim-threshold 0.8 --method cosine --out features_cosine.csv
```

## Evaluation signals

- Merged table contains all detected features across the 545 accessions with no redundant rows beyond the 0.8 average-similarity threshold
- Two variants are generated: one using cosine similarity, one using spectral entropy
- RT differences between merged features do not exceed 1 minute (the alignment tolerance)
- Rows correspond to unique features, each with a consistent precursor m/z and RT across contributing samples

## Limitations

- Alignment is sensitive to instrument reproducibility; low RT or m/z reproducibility inflates false merges or splits
- The 0.8 threshold on the averaged similarity score is empirically set for this 545-accession wild soybean dataset and may not generalize to other matrices or platforms
- The paper notes that its own image-based CNN approach is 'inherently less sensitive to alignment errors caused by low instrument reproducibility', implying this alignment step is a weaker link when reproducibility is poor

## Evidence
