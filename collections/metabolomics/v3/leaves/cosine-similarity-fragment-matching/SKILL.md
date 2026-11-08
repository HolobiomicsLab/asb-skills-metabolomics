---
name: cosine-similarity-fragment-matching
description: Use when MS-DIAL-exported per-sample feature lists (precursor MS1 m/z,
  RT, MS/MS spectra) must be aligned/merged across samples into one non-redundant
  feature table for downstream classification.
when_to_use_negative:
- Downstream analysis is MS2toImg-CNN image encoding, which intentionally bypasses
  feature alignment and relies on the overall fragmentation landscape rather than
  precise feature matching
- Input is already an aligned/unified feature table — re-merging with the 0.8 average-similarity
  threshold would be redundant
- Samples lack MS/MS spectra, since one of the three averaged similarity components
  (MS1, RT, MS/MS) cannot be computed
tools:
- name: MS-DIAL
  role: Preprocesses raw data and extracts the precursor (MS1) and fragment (MS2/MS/MS)
    m/z pairs and intensities that form the per-sample feature lists merged by this
    skill
- name: pandas 2.2.2
  role: Implements the similarity scoring and merging pipeline in Python 3.12.3, producing
    the cosine- and entropy-based feature tables
  repo: https://github.com/jsehhs/MS2toImg-CNN
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:a7bbb91ea44e3a34e8ba5e3145e26fa02a2b02fc77176039feaa19c05347dbb1
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - MS-DIAL
  - Python 3.12
  - pandas
  - pandas 2.2.2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cosine-similarity-fragment-matching

## Summary

Merges per-sample LC-MS/MS features into a unified feature table by scoring MS1 m/z, retention time, and MS/MS spectral similarity (cosine or spectral entropy), consolidating features whose average similarity exceeds 0.8. This was the feature-alignment step for the 545-accession wild soybean dataset before Random Forest and multivariate analysis.

## When to use

Use when MS-DIAL-exported per-sample feature lists (precursor MS1 m/z, RT, MS/MS spectra) must be aligned/merged across samples into one non-redundant feature table for downstream classification.

## When NOT to use

- Downstream analysis is MS2toImg-CNN image encoding, which intentionally bypasses feature alignment and relies on the overall fragmentation landscape rather than precise feature matching
- Input is already an aligned/unified feature table — re-merging with the 0.8 average-similarity threshold would be redundant
- Samples lack MS/MS spectra, since one of the three averaged similarity components (MS1, RT, MS/MS) cannot be computed

## Inputs

- MS-DIAL-exported per-sample feature lists containing precursor MS1 m/z, retention time, and MS/MS spectra with intensities
- Wild soybean (Glycine soja Sieb. & Zucc.) extract LC-MS/MS dataset (545 accessions)

## Outputs

- Unified merged feature table based on cosine similarity for MS/MS matching
- Second unified merged feature table based on spectral entropy for MS/MS matching (for downstream Random Forest / multivariate comparison)

## How to apply

Load the MS-DIAL-exported per-sample feature lists extracted from the 545 wild soybean accession LC-MS/MS dataset. Compute MS1 and RT similarity using the exponential similarity formula, setting sigma for RT to the alignment tolerance (1 minute) and sigma for MS1 to the empirical standard deviation of observed MS1 values. Compute MS/MS spectral similarity two ways — cosine similarity and spectral entropy — treating each MS/MS spectrum as a probability distribution to capture fragment ion alignment and divergence. Merge features when the average of the three similarity scores exceeds 0.8, consolidating them into a single row; otherwise add a new distinct feature entry. Implement in Python 3.12.3 with pandas 2.2.2, producing two feature tables (cosine-based and entropy-based) so downstream classification impact can be compared. Note the article's MS2toImg-CNN path deliberately bypasses this alignment entirely, using raw fragmentation images instead.

## Related tools

- **MS-DIAL** (Preprocesses raw data and extracts the precursor (MS1) and fragment (MS2/MS/MS) m/z pairs and intensities that form the per-sample feature lists merged by this skill)
- **pandas 2.2.2** (Implements the similarity scoring and merging pipeline in Python 3.12.3, producing the cosine- and entropy-based feature tables) — https://github.com/jsehhs/MS2toImg-CNN

## Evaluation signals

- Merged table contains all detected features across the 545 accessions with no redundant rows beyond the 0.8 average-similarity threshold
- Both cosine-based and spectral-entropy-based feature table variants are generated
- RT alignment tolerance is 1 minute, and MS1 sigma equals the empirical standard deviation of observed MS1 values
- Any merged feature pair has an average of MS1, RT, and MS/MS similarity scores > 0.8; pairs below the threshold remain as distinct rows
- Feature tables feed the Random Forest and multivariate (PCA/OPLS-DA) baselines for comparison against MS2toImg-CNN accuracy

## Limitations

- The approach is sensitive to alignment errors caused by low instrument reproducibility, a problem the article's image-based framework was designed to avoid
- Merging is threshold-based (average of three similarity scores > 0.8); features just below the threshold may be kept as spurious duplicates
- Two alternative MS/MS similarity definitions (cosine vs spectral entropy) yield different tables, so downstream classification results depend on the variant chosen

## Evidence
