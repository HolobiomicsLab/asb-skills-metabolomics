---
name: msms-similarity-scoring
description: Use when you have per-sample MS-DIAL feature lists (precursor MS1 m/z,
  RT, MS/MS spectra) from many samples (e.g. 545 wild soybean accessions) and need
  to align and merge equivalent features into one table.
when_to_use_negative:
- Your downstream model consumes raw fragmentation patterns directly (e.g. the MS2toImg
  image encoding), which is explicitly less sensitive to alignment errors and does
  not rely on precise feature matching
- You already have an aligned, non-redundant feature table and only need preprocessing
  steps like TIC normalization or adduct filtering
- Features lack MS/MS spectra or retention time information, making the three-score
  average impossible to compute
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-DIAL
  role: Preprocesses raw LC-MS/MS data and exports the per-sample feature lists (MS1
    m/z, RT, MS/MS pairs with intensities) that serve as the input to similarity-based
    merging
- name: Python 3.12
  role: Runtime for implementing the exponential similarity formulas and merging pipeline
- name: pandas
  role: Used to build and consolidate the merged feature tables (pandas 2.2.2)
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:73ddb690d4b6dd5405f4c9b916639efe4e72999f903a0dee171fbe9a9e67a4eb
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MS-DIAL
  - Python 3.12
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# msms-similarity-scoring

## Summary

Merge equivalent per-sample LC-MS/MS features into a unified feature table by computing similarity scores for MS1 m/z, retention time, and MS/MS spectra, and consolidating features when the average of the three similarity scores exceeds 0.8.

## When to use

Use when you have per-sample MS-DIAL feature lists (precursor MS1 m/z, RT, MS/MS spectra) from many samples (e.g. 545 wild soybean accessions) and need to align and merge equivalent features into one table.

## When NOT to use

- Your downstream model consumes raw fragmentation patterns directly (e.g. the MS2toImg image encoding), which is explicitly less sensitive to alignment errors and does not rely on precise feature matching
- You already have an aligned, non-redundant feature table and only need preprocessing steps like TIC normalization or adduct filtering
- Features lack MS/MS spectra or retention time information, making the three-score average impossible to compute

## Inputs

- MS-DIAL-exported per-sample feature lists (precursor MS1 m/z, retention time, MS/MS spectra with intensities) from 545 wild soybean LC-MS/MS accessions
- Empirical standard deviation of observed MS1 m/z values (used as sigma in the exponential MS1 similarity formula)

## Outputs

- Cosine-similarity-based merged feature table (one row per distinct feature across all accessions)
- Spectral-entropy-based merged feature table for downstream comparison

## How to apply

Load the MS-DIAL-exported per-sample feature lists containing precursor MS1 m/z, retention time, and MS/MS spectra. Compute MS1 and RT similarity scores with an exponential similarity formula, setting sigma for RT to the 1-minute search tolerance and sigma for MS1 to the empirical standard deviation of observed MS1 values; set RT alignment tolerance to 1 minute. Compute MS/MS spectral similarity using both cosine similarity and spectral entropy, treating the spectrum as a probability distribution to capture fragment ion alignment. Merge two features into a single row when the average of the three similarity scores (MS1, RT, MS/MS) exceeds 0.8; otherwise add the feature as a new distinct entry. Implement the pipeline in Python with pandas, producing two variant tables—one cosine-based and one spectral-entropy-based—so downstream classifiers (Random Forest, CNN) can be evaluated on each.

## Related tools

- **MS-DIAL** (Preprocesses raw LC-MS/MS data and exports the per-sample feature lists (MS1 m/z, RT, MS/MS pairs with intensities) that serve as the input to similarity-based merging)
- **Python 3.12** (Runtime for implementing the exponential similarity formulas and merging pipeline)
- **pandas** (Used to build and consolidate the merged feature tables (pandas 2.2.2))

## Examples

```
import pandas as pd; feats = {s: pd.read_csv(f'{s}_msdial_features.csv') for s in samples}; merged = merge_features(feats, rt_tol=1.0, avg_sim_threshold=0.8, ms2_metric='cosine')
```

## Evaluation signals

- Merged table contains all detected features across the 545 accessions with no redundant rows beyond the 0.8 average-similarity threshold
- Both cosine- and entropy-based feature table variants are produced and differ only where the two MS/MS similarity measures disagree
- Merged features never pair entries whose retention times differ by more than the 1-minute alignment tolerance
- Sigma for the MS1 similarity term equals the empirical standard deviation of observed MS1 values in the dataset
- Downstream Random Forest / multivariate analysis runs successfully on both table variants without duplicate-feature artifacts

## Limitations

- The 0.8 average-similarity threshold and 1-minute RT tolerance are dataset-specific choices; features with low instrument reproducibility may be misaligned, motivating the image-based alternative that avoids alignment entirely
- Requires MS/MS spectra per feature; features lacking fragmentation data cannot be scored on one of the three similarity components
- Two parallel tables (cosine vs spectral entropy) must be maintained, since the choice of spectral similarity metric can affect downstream classification results

## Evidence

- [methods] MS-DIAL merged equivalent features across samples by computing similarity scores for MS1 m/z, retention time, and MS/MS, treating features as equivalent and merging them when the average of these three similarity scores exceeded 0.8: [evidence span withheld — anchor status: not_found]
- [methods] Compute MS/MS spectral similarity using both cosine similarity and spectral entropy approaches, treating the MS/MS spectrum as a probability distribution: [evidence span withheld — anchor status: not_found]
- [methods] Implement the merging pipeline in Python 3.12.3 using pandas 2.2.2, producing two feature tables—one based on cosine similarity and one on spectral entropy: [evidence span withheld — anchor status: not_found]
