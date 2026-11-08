---
name: neutral-loss-spectrum-combination
description: Use when comparing parent and TP MS/MS fragment spectra (m/z–intensity
  peak lists) to score similarity for TP componentization, and equal neutral losses
  or a constant mass shift between fragments should contribute to the score.
when_to_use_negative:
- Input is only MS1 feature data with no MS/MS fragment peak lists — spectral matching
  requires fragment spectra
- Candidate TPs are isomers that cannot be distinguished by MS (patRoon instead provides
  a 'Remove isomers' filter for this case)
- The goal is to prioritize candidate structures for a single feature — compound annotation
  (MetFrag, SIRIUS) rather than spectrum comparison is the appropriate step
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: patRoon
  role: Implements the configurable MS spectral similarity comparison (with spectrum
    shifting for equal neutral losses) largely in C++ for efficient comparison of
    large numbers of spectra; used during TP screening and componentization.
  repo: https://github.com/rickhelmus/patRoon
- name: patRoonData
  role: Provides example LC-HRMS data (standard-X standard mixture and solvent-X blanks,
    filtered to 0–10 min and m/z 75–450) used to exercise the workflow including MS
    similarity.
  repo: https://github.com/rickhelmus/patRoonData
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:1edb10455ed7b9a67ab8f08fd4dd110b88ef639d3ef7ab78bdde1155e305d9f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - R
  - patRoon
  - patRoon 2.0
  - patRoonData
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# neutral-loss-spectrum-combination

## Summary

Computes MS/MS spectral similarity between parent and transformation-product (TP) fragment spectra in patRoon, combining matched fragments with neutral losses so that fragments related by a constant mass shift also contribute to cosine and Jaccard similarity scores.

## When to use

Use when comparing parent and TP MS/MS fragment spectra (m/z–intensity peak lists) to score similarity for TP componentization, and equal neutral losses or a constant mass shift between fragments should contribute to the score.

## When NOT to use

- Input is only MS1 feature data with no MS/MS fragment peak lists — spectral matching requires fragment spectra
- Candidate TPs are isomers that cannot be distinguished by MS (patRoon instead provides a 'Remove isomers' filter for this case)
- The goal is to prioritize candidate structures for a single feature — compound annotation (MetFrag, SIRIUS) rather than spectrum comparison is the appropriate step

## Inputs

- MS/MS peak lists (m/z–intensity pairs per spectrum) extracted from LC-HRMS mzML/mzXML data
- Feature lists with parent and TP candidate assignments
- Configuration: m/z match tolerance, intensity and peak-count thresholds, intensity/m/z weights, spectrum-shift option
- Example data from the patRoonData package (standard-X and solvent-X, LC-MS, positive and negative ionization)

## Outputs

- Symmetric similarity matrix (cosine and Jaccard) with a row/column per input spectrum, values in [0, 1]
- TP components grouping each parent feature with its corresponding TP features

## How to apply

Extract and average MS/MS peak lists (m/z and intensity pairs) for each feature, e.g. from patRoon's native MS data extraction on mzML/mzXML input. Pre-treat spectra with configurable peak count and intensity thresholds and normalize fragment intensities, optionally weighting matches by intensity and m/z. For each parent–TP spectrum pair, match fragments within a configurable m/z tolerance and, in the shifted-spectrum comparison, shift TP spectra so fragments related by a constant mass shift (equal neutral losses) are combined into the score. Compute cosine and Jaccard similarities for every spectrum pair and assemble them into a symmetric similarity matrix; patRoon implements this largely in C++ via its C++ backend so thousands of spectra can be compared efficiently. Use the resulting similarities during TP componentization, where each parent feature is linked with corresponding TP features and grouped into a TP component.

## Related tools

- **patRoon** (Implements the configurable MS spectral similarity comparison (with spectrum shifting for equal neutral losses) largely in C++ for efficient comparison of large numbers of spectra; used during TP screening and componentization.) — https://github.com/rickhelmus/patRoon
- **patRoonData** (Provides example LC-HRMS data (standard-X standard mixture and solvent-X blanks, filtered to 0–10 min and m/z 75–450) used to exercise the workflow including MS similarity.) — https://github.com/rickhelmus/patRoonData

## Examples

```
remotes::install_github("rickhelmus/patRoon"); remotes::install_github("rickhelmus/patRoonData")
```

## Evaluation signals

- The output similarity matrix is square and symmetric with a row and column per input spectrum
- All similarity values lie in the [0, 1] range for both cosine and Jaccard metrics
- Spectra compared with the shift option yield higher scores for fragment pairs related by a constant mass shift than the unshifted comparison
- The comparison completes efficiently on large feature sets (patRoon's C++ implementation targets thousands of spectra)
- Parent features are grouped with their corresponding TP features into TP components

## Limitations

- Isomers can be difficult or impossible to distinguish with MS, so identical spectra may not prove identical structures (patRoon applies a 'Remove isomers' filter)
- The shifted-spectrum comparison highlights equal neutral losses, i.e. fragments related by a constant mass shift; fragment differences without a shared shift do not benefit
- Similarity alone can mislead: patRoon's 'Structure similarity' filter removes TPs with high MS similarity but low structural similarity

## Evidence

- [abstract] MS spectral similarity comparison was largely implemented in C++ for efficient comparison of large numbers of spectra: "This was largely implemented in C++ to allow efficient comparison of large numbers of spectra (typically thousands)."
- [abstract] TP componentization links parent and TP features: "During TP componentization (Figure 1F), each parent feature is linked with corresponding TP features and grouped in a TP component."
- [readme] MS data extraction provides the peak lists this skill consumes: [evidence span withheld — anchor status: external_doc]
- [readme] patRoon purpose and NTA scope: "`patRoon` aims to provide comprehensive mass spectrometry based non-target analysis (NTA) workflows for environmental analysis."
