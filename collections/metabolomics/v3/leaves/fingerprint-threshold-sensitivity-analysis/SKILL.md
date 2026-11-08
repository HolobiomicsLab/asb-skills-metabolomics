---
name: fingerprint-threshold-sensitivity-analysis
description: Use when a pipeline binarizes predicted fingerprint probabilities into
  a fixed-length bit vector via a threshold parameter, and you need to justify the
  chosen threshold or demonstrate insensitivity by retraining/evaluating across epsilon
  values.
when_to_use_negative:
- When the fingerprint representation is already binary and no probability vector
  (hence no binarization threshold) exists
- When evaluating a single fixed configuration rather than characterizing sensitivity
  of metrics to the threshold parameter
- When the downstream model conditions directly on probability vectors rather than
  thresholded bits, since the epsilon sweep would have no effect
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MIST
  role: Pretrained spectral inference model that produces the fingerprint probability
    vectors which the epsilon threshold binarizes into 4096-bit fingerprints
  repo: https://github.com/coleygroup/MIST
- name: MS-BART
  role: BART-Base model that is fine-tuned, chemically aligned, and evaluated per
    epsilon variant to measure threshold sensitivity
  repo: https://github.com/OpenDFM/MS-BART
- name: RDKit
  role: Computes Morgan fingerprints (radius 2, 2048 bits) for Tanimoto-similarity
    evaluation of generated structures
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:3b6764da4d496205dba1227fc8b7036dcbe045b65c44ed32927dd2db119cff21
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/MIST
  tools:
  - BART-Base
  - MIST
  - MS-BART
  - RDKit
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# fingerprint-threshold-sensitivity-analysis

## Summary

Sweeps the fingerprint binarization threshold (epsilon) used to convert MIST probability vectors into binary 4096-bit Morgan fingerprints, then measures how Top-1 Tanimoto similarity and related metrics vary across epsilon values to test whether the downstream structure-elucidation model is sensitive to this filter step.

## When to use

Use when a pipeline binarizes predicted fingerprint probabilities into a fixed-length bit vector via a threshold parameter, and you need to justify the chosen threshold or demonstrate insensitivity by retraining/evaluating across epsilon values.

## When NOT to use

- When the fingerprint representation is already binary and no probability vector (hence no binarization threshold) exists
- When evaluating a single fixed configuration rather than characterizing sensitivity of metrics to the threshold parameter
- When the downstream model conditions directly on probability vectors rather than thresholded bits, since the epsilon sweep would have no effect

## Inputs

- MIST-predicted molecular fingerprint probability vectors (per spectrum, conditioned on chemical formula)
- MassSpecGym [M+H]+ training/validation/test splits with metadata
- Pretrained MS-BART model weights (BART-Base backbone)
- Candidate epsilon values {0.10, 0.11, 0.15, 0.20}

## Outputs

- Per-epsilon fingerprint-SELFIES input tables for train/val/test folds
- Per-epsilon fine-tuned + aligned model checkpoints
- epsilon_sensitivity_results.csv with Top-1/Top-10 exact-match InChIKey accuracy, minimum MCES, and maximum Tanimoto similarity per threshold, ordered by epsilon

## How to apply

Convert MIST probability vectors into binary 4096-bit fingerprints for each candidate epsilon in {0.10, 0.11, 0.15, 0.20} (e.g. via preprocess/prepare_test_data.py --threshold), generating fingerprint-SELFIES input tables per variant and splitting train/val/test folds. For each variant, fine-tune the pretrained MS-Bart model (BART-Base backbone, input/output token length 256) on the MassSpecGym [M+H]+ training split, then run chemical-feedback alignment (Tanimoto-similarity rank loss, frozen encoder, updated decoder). Evaluate each aligned model on the full test fold with beam-search multinomial sampling, computing Top-1/Top-10 exact-match InChIKey accuracy, minimum MCES, and maximum Tanimoto similarity (Morgan fingerprints, radius 2, 2048 bits, RDKit). Assemble per-epsilon metrics into an ordered comparison table (epsilon_sensitivity_results.csv) and compare against the paper's reported pattern: Top-1 Tanimoto approximately 0.23 across all tested epsilon values, with epsilon=0.11 giving Top-1 accuracy 1.07% and Top-1 MCES 16.47 — i.e. epsilon=0.11 for MassSpecGym and epsilon=0.2 for NPLIB1 were chosen but MS-Bart is not sensitive to the threshold.

## Related tools

- **MIST** (Pretrained spectral inference model that produces the fingerprint probability vectors which the epsilon threshold binarizes into 4096-bit fingerprints) — https://github.com/coleygroup/MIST
- **MS-BART** (BART-Base model that is fine-tuned, chemically aligned, and evaluated per epsilon variant to measure threshold sensitivity) — https://github.com/OpenDFM/MS-BART
- **RDKit** (Computes Morgan fingerprints (radius 2, 2048 bits) for Tanimoto-similarity evaluation of generated structures)

## Examples

```
python preprocess/prepare_test_data.py --threshold 0.11
```

## Evaluation signals

- Top-1 Tanimoto similarity stays approximately constant (~0.23) across all tested epsilon values, reproducing Table 4
- The reference configuration (epsilon=0.11, MassSpecGym) yields Top-1 exact-match InChIKey accuracy of 1.07% and Top-1 MCES of 16.47
- Fingerprint tables contain exactly 4096 binary bits per spectrum for every epsilon variant
- Each epsilon variant has an identically structured train/val/test fold split, so metric differences are attributable to thresholding only
- epsilon_sensitivity_results.csv rows are ordered by threshold value and cover all four tested epsilon values {0.10, 0.11, 0.15, 0.20}

## Limitations

- The chosen epsilon differs per benchmark (0.11 for MassSpecGym vs 0.2 for NPLIB1), so a single threshold does not transfer across datasets
- Downstream performance depends on MIST fingerprint prediction accuracy, which caps what threshold tuning can achieve
- Evaluation on MassSpecGym used all adducts retained at test time although fine-tuning filtered to [M+H]+, so sensitivity conclusions apply to this evaluation protocol

## Evidence

- [methods] Fingerprint representation binarized: [evidence span withheld — anchor status: not_found]
- [readme] MIST fingerprint prediction from spectrum: [evidence span withheld — anchor status: external_doc]
