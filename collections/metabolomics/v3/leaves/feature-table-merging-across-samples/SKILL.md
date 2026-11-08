---
name: feature-table-merging-across-samples
description: Use when per-sample MS-DIAL feature lists (precursor MS1 m/z, RT, MS/MS
  spectra) must be consolidated into one aligned feature table across many LC-MS/MS
  samples before classification or multivariate analysis.
when_to_use_negative:
- Input is already an aligned/unified feature table — merging would only re-deduplicate
  rows
- The downstream model is the MS2toImg CNN, which deliberately bypasses explicit feature
  alignment and consumes raw LC-MS/MS data as 2D grayscale molecular fingerprint images
- MS/MS spectra are unavailable for most features, since one third of the 0.8 average-similarity
  criterion cannot be computed reliably
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MS-DIAL
  role: Preprocesses raw LC-MS/MS data and extracts per-sample precursor (MS1) and
    fragment (MS2) m/z pairs with intensities that feed the merging step
- name: Python 3.12.3
  role: Implementation language for the merging pipeline computing MS1/RT/MS/MS similarity
    scores
- name: pandas 2.2.2
  role: Dataframe handling for consolidating merged feature rows and exporting the
    two feature-table variants
- name: MS2toImg-CNN (official implementation)
  role: Downstream framework in the same study whose CNN input bypasses this alignment
    step; its repository documents the parent manuscript
  repo: https://github.com/jsehhs/MS2toImg-CNN
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:f4ab966ba644b5f81dd80ba882ce6ce1b15e499719d98a0199f7067219c92519
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - MS-DIAL
  - MS2toImg-CNN (official implementation)
  - Python 3.12
  - Python 3.12.3
  - pandas
  - pandas 2.2.2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# feature-table-merging-across-samples

## Summary

Merges equivalent MS-DIAL-detected features across many LC-MS/MS samples (e.g. 545 wild soybean accessions) into a unified feature table by averaging MS1 m/z, retention time, and MS/MS spectral similarity scores and merging rows whose average exceeds 0.8. Produces paired cosine- and spectral-entropy-based tables for downstream Random Forest / multivariate comparison.

## When to use

Use when per-sample MS-DIAL feature lists (precursor MS1 m/z, RT, MS/MS spectra) must be consolidated into one aligned feature table across many LC-MS/MS samples before classification or multivariate analysis.

## When NOT to use

- Input is already an aligned/unified feature table — merging would only re-deduplicate rows
- The downstream model is the MS2toImg CNN, which deliberately bypasses explicit feature alignment and consumes raw LC-MS/MS data as 2D grayscale molecular fingerprint images
- MS/MS spectra are unavailable for most features, since one third of the 0.8 average-similarity criterion cannot be computed reliably

## Inputs

- MS-DIAL-exported per-sample feature lists from the 545 wild soybean (Glycine soja Sieb. & Zucc.) LC-MS/MS accessions, each containing precursor MS1 m/z, retention time, and MS/MS spectra
- MS-DIAL alignment settings: RT tolerance 1 minute; adduct set [M-H]-, [M-H2O-H]-, [M+Na-2H]-, [M-K-2H]-, [M-FA-H]-; peaks retained above 10^-4 intensity; TIC normalization

## Outputs

- Unified merged feature table (cosine-similarity variant) containing all detected features across the 545 accessions
- Unified merged feature table (spectral-entropy variant) for evaluating downstream Random Forest / multivariate classification impact

## How to apply

Load MS-DIAL-exported per-sample feature lists (precursor MS1 m/z, retention time, and MS/MS spectra) extracted from the 545 wild soybean extract LC-MS/MS dataset. Compute MS1 and RT similarity scores with the exponential similarity formula, setting sigma for RT to the 1-minute search tolerance and sigma for MS1 to the empirical standard deviation of observed MS1 values. Compute MS/MS spectral similarity treating the spectrum as a probability distribution, using both cosine similarity and spectral entropy to capture fragment ion alignment. Merge features when the average of the three similarity scores exceeds 0.8 (treating them as equivalent and consolidating into a single row); otherwise add a new distinct feature entry, with RT alignment tolerance fixed at 1 minute. Implement in Python 3.12.3 with pandas 2.2.2, emitting two feature tables — one cosine-based, one entropy-based — to evaluate the downstream classification impact of the similarity choice.

## Related tools

- **MS-DIAL** (Preprocesses raw LC-MS/MS data and extracts per-sample precursor (MS1) and fragment (MS2) m/z pairs with intensities that feed the merging step)
- **Python 3.12.3** (Implementation language for the merging pipeline computing MS1/RT/MS/MS similarity scores)
- **pandas 2.2.2** (Dataframe handling for consolidating merged feature rows and exporting the two feature-table variants)
- **MS2toImg-CNN (official implementation)** (Downstream framework in the same study whose CNN input bypasses this alignment step; its repository documents the parent manuscript) — https://github.com/jsehhs/MS2toImg-CNN

## Evaluation signals

- Merged table contains all detected features across the 545 accessions with no redundant rows beyond the 0.8 average-similarity threshold
- Two table variants (cosine-based and spectral-entropy-based) are both generated for downstream comparison
- Merged rows respect the 1-minute RT alignment tolerance
- Features failing the 0.8 average of MS1, RT, and MS/MS similarity appear as distinct entries rather than being force-merged
- Downstream Random Forest / multivariate analyses run without duplicate-feature conflicts from the merged tables

## Limitations

- The 0.8 average-similarity threshold and 1-minute RT tolerance are empirical choices; alignment is sensitive to instrumental reproducibility across samples and platforms
- Results depend on the similarity metric choice — the pipeline must produce both cosine and spectral entropy variants because the classification impact differs
- No standalone released implementation or license for the merging pipeline exists; the parent repo is under peer-review restriction (NOASSERTION) forbidding redistribution or use until formal publication

## Evidence
