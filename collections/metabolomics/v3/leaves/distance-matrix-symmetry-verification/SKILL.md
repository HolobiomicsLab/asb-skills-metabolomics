---
name: distance-matrix-symmetry-verification
description: Use when assembling an n×n global distance matrix from pairwise dataset
  comparisons (e.g. 10 comparisons for 5 serum datasets from PXD034932) and you must
  confirm the metric is symmetric and the diagonal is zero.
when_to_use_negative:
- The underlying metric is asymmetric or the tool reports direction-dependent distances
  — mirroring values would mask real asymmetry rather than verify it
- The matrix is being used for non-symmetric analyses (e.g. directed graphs) where
  the diagonal-zero, symmetric-matrix assumption does not hold
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: compareMS2
  role: Computes the pairwise dataset distances with the symmetric eq 1 spectral similarity
    metric whose outputs are verified for symmetry
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:f3854dedb18214ed7f7a77a179865610d209b97b99bdc59f6f17ede5a4ac9cf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Distance matrix symmetry verification

## Summary

Verify that a pairwise distance matrix computed with compareMS2 2.0's fully symmetric metric satisfies exact symmetry, D(A,B) = D(B,A) for all pairs, before saving or downstream phylogenetic use.

## When to use

Use when assembling an n×n global distance matrix from pairwise dataset comparisons (e.g. 10 comparisons for 5 serum datasets from PXD034932) and you must confirm the metric is symmetric and the diagonal is zero.

## When NOT to use

- The underlying metric is asymmetric or the tool reports direction-dependent distances — mirroring values would mask real asymmetry rather than verify it
- The matrix is being used for non-symmetric analyses (e.g. directed graphs) where the diagonal-zero, symmetric-matrix assumption does not hold

## Inputs

- 10 unique pairwise distance values from compareMS2 2.0 runs over the five serum datasets (ProteomeXchange PXD034932, 42,629–47,626 tandem MS2 spectra each)
- List of the five dataset identifiers defining matrix row/column order

## Outputs

- Symmetric 5x5 distance matrix table saved as distance_matrix.tsv
- Verification record that all 10 unique pairwise distances are present and D(A,B) = D(B,A) exactly

## How to apply

After running compareMS2 2.0 on all unique pairwise combinations (n(n-1)/2 = 10 comparisons for the five new serum datasets of PXD034932, each containing 42,629–47,626 tandem mass spectra), collect the pairwise distances into a 5x5 matrix table. The default compareMS2 2.0 distance metric is symmetric, so if D(A, B) has already been calculated there is no need to calculate D(B, A), and D(A, A) is always zero, so self-comparisons are unnecessary. Populate only the lower (or upper) triangle, then mirror the values and check that D(A,B) = D(B,A) exactly for all 10 unique pairs. On the reference Intel Xeon W-2135 at 3.70 GHz the 10 default-parameter comparisons took about 40 min, so a far longer runtime may signal duplicated work. Save the validated, symmetric matrix as distance_matrix.tsv.

## Related tools

- **compareMS2** (Computes the pairwise dataset distances with the symmetric eq 1 spectral similarity metric whose outputs are verified for symmetry) — https://github.com/524D/compareMS2.git

## Evaluation signals

- Matrix is exactly symmetric: D(A,B) = D(B,A) for all 10 unique pairs of the five datasets
- Diagonal entries are all zero (D(A,A) = 0 by definition of the metric)
- All n(n-1)/2 = 10 unique pairwise comparisons are present, with no missing or duplicated pairs
- Compute cost is consistent with the reference: ~40 min for the 10 comparisons at default parameters on an Intel Xeon W-2135 at 3.70 GHz
- Output file distance_matrix.tsv parses and matches the lower-triangular distances reported by compareMS2

## Limitations

- Only the default compareMS2 2.0 distance metric is guaranteed symmetric; earlier versions or custom settings may not be
- Dataset pairs with no similar spectra tend toward distance +infinity as set size grows, so extreme distances can occur and should be checked for numerical overflow
- Compute time scales quadratically: 100 LC-MS/MS datasets require 4950 pairwise comparisons taking several hours, so full matrices become expensive

## Evidence

- [readme] The default distance metric D is symmetric, i.e. the distance from dataset A to dataset B is identical to the distance from dataset B to dataset A.: [evidence span withheld — anchor status: external_doc]
- [readme] If the distance D(A, B) has already been calculated, there is no need to calculate D(B, A).: [evidence span withheld — anchor status: external_doc]
- [results] 10 pairwise comparisons in compareMS2 took 40 min with default parameters: "The 10 pairwise comparisons in compareMS2 took 40 min with default parameters"
- [results] five serum datasets contain between 42,629 and 47,626 tandem mass spectra each: "each containing between 42,629 and 47,626 tandem mass spectra"
- [results] comparing 100 LC-MS/MS datasets require 4950 pairwise comparisons, taking several hours: "comparing 100 LC-MS/MS datasets require 4950 pairwise comparisons, taking several hours"
