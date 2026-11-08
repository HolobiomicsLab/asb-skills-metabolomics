---
name: spectral-similarity-scoring
description: Use when LC-HRMS/MS fragment peak lists (m/z + intensity pairs) must
  be compared across parent and TP spectra, e.g. to support TP componentization, library
  matching, or clustering by MS/MS similarity.
when_to_use_negative:
- Input is already a feature table or neutral-mass alignment result rather than raw
  MS/MS fragment peak lists
- The goal is structural (not spectral) similarity assessment — the article describes
  a complementary 'Structure similarity' filter that removes TPs with high MS similarity
  but low structural similarity
- Only MS1-level feature detection/grouping is needed; spectral matching requires
  MS/MS peak lists
tools:
- name: patRoon 2.0
  role: Implements the spectral-match similarity calculation (pre-treatment, weighting,
    neutral-loss shifting) with the comparison loop in C++ for efficient pairwise
    scoring
  repo: https://rickhelmus.github.io/patRoon
- name: R
  role: Platform in which patRoon's spectral similarity workflow is executed and configured
- name: patRoonData
  role: Provides example LC-HRMS data (standard mixture and blank solvent, first ten
    minutes, m/z 75-450) for exercising and validating the similarity scoring
  repo: https://github.com/rickhelmus/patRoonData
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:76f5afb4d30845ce18b290aac8fc868254976aa1323f63fb47183af8d055ed39
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - patRoon 2.0
  - patRoonData
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spectral-similarity-scoring

## Summary

Scores MS/MS spectral similarity between parent and transformation-product fragment spectra using configurable peak filtering, intensity/m/z weighting, and neutral-loss spectrum shifting, as implemented in patRoon 2.0 with a C++ core for efficient comparison of thousands of spectra.

## When to use

Use when LC-HRMS/MS fragment peak lists (m/z + intensity pairs) must be compared across parent and TP spectra, e.g. to support TP componentization, library matching, or clustering by MS/MS similarity.

## When NOT to use

- Input is already a feature table or neutral-mass alignment result rather than raw MS/MS fragment peak lists
- The goal is structural (not spectral) similarity assessment — the article describes a complementary 'Structure similarity' filter that removes TPs with high MS similarity but low structural similarity
- Only MS1-level feature detection/grouping is needed; spectral matching requires MS/MS peak lists

## Inputs

- MS/MS peak lists (m/z and intensity pairs per spectrum) for parent and transformation-product features
- Configuration options: peak count/intensity pre-treatment thresholds, intensity and m/z weighting, m/z tolerance, spectrum-shifting (neutral-loss) setting

## Outputs

- Symmetric spectral similarity matrix with a row/column per input spectrum
- Pairwise similarity scores for parent–TP spectrum pairs used downstream in TP componentization and MS similarity filtering

## How to apply

Load MS/MS peak lists (m/z and intensity pairs per spectrum), e.g. from patRoonData example LC-HRMS data, and pre-treat each spectrum with configurable peak count and intensity thresholds, normalizing fragment intensities. For each spectrum pair, match fragments within a configurable m/z tolerance while weighting matches by intensity and m/z. Enable the shifted-spectrum comparison so TP spectra are shifted to highlight equal neutral losses, allowing fragments related by a constant mass shift to contribute to the score. Compute similarity scores for every spectrum pair and assemble them into a symmetric similarity matrix; run the comparison loop in C++ (as patRoon does, e.g. via Rcpp) because the number of spectra is typically in the thousands.

## Related tools

- **patRoon 2.0** (Implements the spectral-match similarity calculation (pre-treatment, weighting, neutral-loss shifting) with the comparison loop in C++ for efficient pairwise scoring) — https://rickhelmus.github.io/patRoon
- **R** (Platform in which patRoon's spectral similarity workflow is executed and configured)
- **patRoonData** (Provides example LC-HRMS data (standard mixture and blank solvent, first ten minutes, m/z 75-450) for exercising and validating the similarity scoring) — https://github.com/rickhelmus/patRoonData

## Evaluation signals

- Output similarity matrix is square and symmetric with one row/column per input spectrum
- All similarity values lie in the [0, 1] range
- Matrix diagonal entries equal the self-similarity (maximum score) for each spectrum
- Fragments related by a constant mass shift contribute to the score when spectrum shifting for equal neutral losses is enabled
- Comparison completes efficiently on large inputs (patRoon targets thousands of spectra via the C++ implementation)

## Limitations

- Isomers can be difficult or impossible to distinguish with MS, so high spectral similarity alone cannot confirm a TP identity (patRoon adds a Remove isomers filter and a structure similarity filter for this reason)
- Similarity results depend on configurable pre-treatment thresholds and weighting choices; results are not comparable unless settings are fixed
- Pure R comparison of large numbers of spectra is impractical; the efficient path relies on the C++ implementation

## Evidence

- [abstract] C++ implementation for scale: "This was largely implemented in C++ to allow efficient comparison of large numbers of spectra (typically thousands)."
- [other] configurable pre-treatment and shifting: [evidence span withheld — anchor status: not_found]
- [readme] MS/MS similarity in componentization: [evidence span withheld — anchor status: external_doc]
