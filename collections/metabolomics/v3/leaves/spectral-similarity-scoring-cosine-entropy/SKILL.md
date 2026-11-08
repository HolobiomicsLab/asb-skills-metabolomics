---
name: spectral-similarity-scoring-cosine-entropy
description: Use when you have MS-DIAL-aligned peak tables from untargeted LC-MS/MS
  runs and need to decide whether detected features across samples are equivalent
  (merge) or distinct (new entry) before building a feature table.
when_to_use_negative:
- The input is already a merged, aligned feature table — this skill's merge decision
  (average of MS1/RT/MS/MS scores > 0.8) applies only to unmerged aligned peak data
- The goal is end-to-end bioactivity prediction from raw data without feature alignment
  — the article's MS2toImg approach deliberately bypasses this alignment-and-merge
  pipeline
- Structural identification or annotation of individual features is required — this
  skill only scores feature equivalence, not compound identity
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS-DIAL
  role: Preprocesses raw data and extracts precursor (MS1) and fragment (MS2) m/z
    pairs with intensities that feed the similarity scoring and merging steps
- name: pandas
  role: Merges equivalent features (average similarity > 0.8) into the two unified
    feature tables (cosine similarity and spectral entropy variants)
- name: scikit-learn
  role: Provides the downstream Random Forest baseline (v1.3.0, default hyperparameters,
    5-fold CV) trained on the merged feature tables
- name: numpy
  role: Supports the exponential-decay score computations using empirical MS1 standard
    deviation and RT tolerance as sigma
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:2adb86fb4642f54715cb097db45c993cdf68b62562ccfcd317a155ab9841a477
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MS-DIAL
  - numpy
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Spectral similarity scoring with cosine and spectral entropy

## Summary

Computes pairwise feature-similarity scores — exponential-decay scores for MS1 m/z and retention time, plus MS/MS spectral similarity as both cosine similarity and spectral entropy — to merge aligned LC-MS/MS features into unified feature tables for downstream classical ML baselines.

## When to use

Use when you have MS-DIAL-aligned peak tables from untargeted LC-MS/MS runs and need to decide whether detected features across samples are equivalent (merge) or distinct (new entry) before building a feature table.

## When NOT to use

- The input is already a merged, aligned feature table — this skill's merge decision (average of MS1/RT/MS/MS scores > 0.8) applies only to unmerged aligned peak data
- The goal is end-to-end bioactivity prediction from raw data without feature alignment — the article's MS2toImg approach deliberately bypasses this alignment-and-merge pipeline
- Structural identification or annotation of individual features is required — this skill only scores feature equivalence, not compound identity

## Inputs

- MS-DIAL-aligned peak table (profile MS1, centroid MS/MS) with peak intensities, filtered to values > 10^-4
- TIC-normalized peak intensities aligned with 1-minute retention-time tolerance
- Empirical standard deviation of observed MS1 m/z values (decay sigma for MS1 scoring)
- RT search tolerance (decay sigma for retention-time scoring)
- Paired MS/MS peak lists (m/z, intensity) per feature per sample

## Outputs

- Unified feature table built using MS/MS cosine similarity
- Unified feature table built using MS/MS spectral entropy
- Per-pair similarity scores (MS1 decay, RT decay, MS/MS cosine, MS/MS spectral entropy) used for the merge decision

## How to apply

Start from MS-DIAL-aligned peak data (profile MS1, centroid MS/MS) filtered to peak intensities > 10^-4 and TIC-normalized, with alignment at a 1-minute retention-time tolerance. For each candidate feature pair, compute an exponential-decay score for MS1 m/z using sigma equal to the empirical standard deviation of observed MS1 values, an exponential-decay score for retention time using sigma equal to the RT search tolerance, and MS/MS spectral similarity computed both as cosine similarity and as spectral entropy. Merge each pair into a unified feature table with pandas when the average of the three similarity scores exceeds 0.8; otherwise add the feature as a new entry. This yields two parallel tables — one keyed on cosine similarity, one on spectral entropy — which serve as inputs to baseline classifiers (e.g., Random Forest with scikit-learn 1.3.0 default hyperparameters, 5-fold cross-validation). Correctness is judged by whether the downstream baseline reproduces reported metrics, e.g., an AUC of approximately 0.817 for the DPPH high/low bioactivity task.

## Related tools

- **MS-DIAL** (Preprocesses raw data and extracts precursor (MS1) and fragment (MS2) m/z pairs with intensities that feed the similarity scoring and merging steps)
- **pandas** (Merges equivalent features (average similarity > 0.8) into the two unified feature tables (cosine similarity and spectral entropy variants))
- **scikit-learn** (Provides the downstream Random Forest baseline (v1.3.0, default hyperparameters, 5-fold CV) trained on the merged feature tables)
- **numpy** (Supports the exponential-decay score computations using empirical MS1 standard deviation and RT tolerance as sigma)

## Examples

```
score = np.mean([s_ms1, s_rt, s_ms2]); feature_table = pd.merge(feature_table, feature) if score > 0.8 else pd.concat([feature_table, feature], axis=1)
```

## Evaluation signals

- The two merged feature tables (cosine-similarity-based and spectral-entropy-based) are produced from the same filtered, TIC-normalized aligned peak data
- Every merged pair has an average of MS1, RT, and MS/MS similarity scores strictly above 0.8; every non-merged pair appears as a new table entry
- MS1 decay sigma equals the empirical standard deviation of observed MS1 values; RT decay sigma equals the search tolerance
- Random Forest trained on either table (scikit-learn 1.3.0, default hyperparameters, 5-fold CV) reports accuracy, precision, recall, F1, and ROC AUC ≈ 0.817 on the DPPH task, matching Figure 4C / Figure 5 within reasonable tolerance
- DPPH labels follow the article's grouping: normalized values < 0.4 = low activity, > 0.6 = high activity over 545 wild soybean accessions

## Limitations

- The merge threshold (0.8 average of three scores) and the negative-mode adduct settings ([M-H]⁻, [M-H₂O-H]⁻, [M+Na-2H]⁻, [M-K-2H]⁻, [M-FA-H]⁻) are specific to this study and may not transfer to other instruments or ionization modes
- Feature alignment quality depends on instrumental reproducibility; the article notes alignment errors from low instrument reproducibility as a general weakness of feature-table pipelines
- This skill supports conventional feature-table baselines (Random Forest, PCA, OPLS-DA), which the source paper reports being outperformed by the image-based CNN approach

## Evidence
