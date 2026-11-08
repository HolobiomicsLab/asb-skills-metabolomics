---
name: shifted-spectrum-alignment
description: Use when MS/MS peak lists (m/z and intensity pairs) of a parent and a
  candidate TP feature must be compared for TP componentization, especially when fragments
  differ by a constant mass shift corresponding to equal neutral losses.
when_to_use_negative:
- Features have no MS/MS peak lists (MS1-only data), so no fragment spectra exist
  to align
- The goal is distinguishing isomers — the article's remove-isomers filter notes isomers
  can be difficult or impossible to distinguish with MS
- Comparison requires structural rather than spectral evidence — the structure-similarity
  filter specifically removes TPs with high MS similarity but low structural similarity
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: patRoon
  role: Provides the MS spectral similarity comparison (implemented largely in C++
    for efficiency) and uses the scores for TP componentization
  repo: https://rickhelmus.github.io/patRoon
- name: patRoonData
  role: Supplies the example raw LC-HRMS data (standard-X and solvent-X triplicates)
    used to demonstrate and validate the spectral matching
  repo: https://github.com/rickhelmus/patRoonData
- name: R
  role: Implementation platform hosting the Rcpp-backed C++ comparison loop and the
    similarity-matrix assembly
  repo: https://rickhelmus.github.io/patRoon
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:e6716b9ea340d3bf3c58213fb2fd8c0139a50912576fb6d1f8ddcdb122b6ce59
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - patRoon
  - patRoon 2.0
  - patRoonData
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# shifted-spectrum-alignment

## Summary

A spectral-matching method from patRoon 2.0 that compares parent and transformation-product (TP) MS/MS fragment spectra, allowing TP spectra to be shifted by a constant m/z so that fragments related by equal neutral losses contribute to cosine and Jaccard similarity scores.

## When to use

Use when MS/MS peak lists (m/z and intensity pairs) of a parent and a candidate TP feature must be compared for TP componentization, especially when fragments differ by a constant mass shift corresponding to equal neutral losses.

## When NOT to use

- Features have no MS/MS peak lists (MS1-only data), so no fragment spectra exist to align
- The goal is distinguishing isomers — the article's remove-isomers filter notes isomers can be difficult or impossible to distinguish with MS
- Comparison requires structural rather than spectral evidence — the structure-similarity filter specifically removes TPs with high MS similarity but low structural similarity

## Inputs

- MS/MS peak lists (m/z and intensity pairs) for parent and TP feature spectra, e.g. from mzML/mzXML raw data processed in patRoon
- patRoonData example LC-HRMS data (standard-X and solvent-X, positive and negative ionization)
- Parent/TP feature assignments (e.g. from TP suspect screening by mass)

## Outputs

- Symmetric similarity matrix (cosine and Jaccard scores in [0, 1]) with a row and column per input spectrum
- TP components grouping each parent feature with its MS-similar TP features

## How to apply

Load MS/MS peak lists (m/z and intensity pairs per spectrum), e.g. from the patRoonData example LC-HRMS data, and pre-process them with configurable peak count and intensity thresholds plus fragment-intensity normalization. For each parent/TP spectrum pair, match fragments within a configurable m/z tolerance, weighting matches by intensity and m/z, and additionally compare the TP spectrum under a constant m/z shift so fragments related by equal neutral losses also contribute to the score. Compute cosine and Jaccard similarity scores for every spectrum pair and assemble them into a symmetric similarity matrix. For efficiency over thousands of spectra, run the comparison loop in C++ (patRoon uses Rcpp). The resulting similarities feed patRoon's TP componentization, which links each parent feature with corresponding TP features and groups them into a TP component.

## Related tools

- **patRoon** (Provides the MS spectral similarity comparison (implemented largely in C++ for efficiency) and uses the scores for TP componentization) — https://rickhelmus.github.io/patRoon
- **patRoonData** (Supplies the example raw LC-HRMS data (standard-X and solvent-X triplicates) used to demonstrate and validate the spectral matching) — https://github.com/rickhelmus/patRoonData
- **R** (Implementation platform hosting the Rcpp-backed C++ comparison loop and the similarity-matrix assembly) — https://rickhelmus.github.io/patRoon

## Examples

```
generateComponentsTP(fGroups, transformations)  # patRoon TP componentization using MS spectral similarity
```

## Evaluation signals

- The output similarity file exists and the matrix is square and symmetric with one row/column per input spectrum
- All cosine and Jaccard similarity values lie in the [0, 1] range
- Fragments related by a constant mass shift (equal neutral losses) contribute to the score under the shifted-spectrum comparison
- Changing the configured m/z tolerance, intensity thresholds, or intensity/m/z weights visibly changes the similarity matrix
- Computation over thousands of spectra completes efficiently, consistent with the C++ implementation

## Limitations

- Similarity settings (peak count/intensity thresholds, weighting of intensity and m/z, m/z tolerance, shifting) are configurable, so results depend on chosen parameters
- High MS similarity alone is not sufficient evidence of a TP — patRoon's structure similarity filter removes TPs with high MS similarity but low structural similarity
- Isomers can be difficult or impossible to distinguish with MS, limiting what spectral similarity can prove

## Evidence

- [abstract] This was largely implemented in C++ to allow efficient comparison of large numbers of spectra (typically thousands).: [evidence span withheld — anchor status: not_found]
- [readme] Componentization grouping by MS/MS similarity: [evidence span withheld — anchor status: external_doc]
- [abstract] Parent/TP linking during TP componentization: "During TP componentization (Figure 1F), each parent feature is linked with corresponding TP features and grouped in a TP component."
- [intro] patRoon's purpose statement: "patRoon aims to provide comprehensive mass spectrometry based non-target analysis (NTA) workflows for environmental analysis."
