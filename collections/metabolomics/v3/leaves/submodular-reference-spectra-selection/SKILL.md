---
name: submodular-reference-spectra-selection
description: Use when a learned spectral encoder (e.g. GLEAMS) needs a small, diverse
  reference-spectrum set for pairwise similarity features, and the training corpus
  (~200,000 downsampled MS/MS spectra) is too large to compare against exhaustively.
when_to_use_negative:
- The input spectra are unidentified experimental runs rather than labeled training
  PSMs from a curated knowledge base like MassIVE-KB — the selection is defined on
  identified training data.
- The downstream model does not use reference-based similarity features (e.g. plain
  `gleams cluster` hierarchical clustering on embeddings needs no reference set).
- The candidate pool is already small enough to use directly as references, making
  submodular selection unnecessary overhead.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: apricot
  role: Performs the greedy submodular facility-location selection of 500 reference
    spectra from the precomputed pairwise similarity matrix
- name: GLEAMS
  role: Downstream encoder whose similarity features consume the 500 selected reference
    spectra; training data and configuration live in its repository
  repo: https://github.com/bittremieux/GLEAMS
- name: Python 3.8
  role: Runtime environment in which the similarity matrix computation and apricot
    selection are executed (GLEAMS requires Python 3.8 on Linux with a CUDA GPU)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:b3aa2ccc55b5bdc4f108df2da17a6958ee6243d3fd8af1fa68f56a141fbadb4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - ANN-SoLo
  - GLEAMS
  - Python
  - Python 3.8
  - apricot
  - conda
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# submodular-reference-spectra-selection

## Summary

Select a compact, representative set of reference MS/MS spectra (500 in the GLEAMS workflow) from a large labeled training corpus by running apricot's submodular facility-location function on a precomputed pairwise similarity matrix, so the encoder can compute similarity features against a small set of reference spectra instead of the full ~200,000-spectrum training set.

## When to use

Use when a learned spectral encoder (e.g. GLEAMS) needs a small, diverse reference-spectrum set for pairwise similarity features, and the training corpus (~200,000 downsampled MS/MS spectra) is too large to compare against exhaustively.

## When NOT to use

- The input spectra are unidentified experimental runs rather than labeled training PSMs from a curated knowledge base like MassIVE-KB — the selection is defined on identified training data.
- The downstream model does not use reference-based similarity features (e.g. plain `gleams cluster` hierarchical clustering on embeddings needs no reference set).
- The candidate pool is already small enough to use directly as references, making submodular selection unnecessary overhead.

## Inputs

- ~200,000 downsampled training MS/MS spectra from the MassIVE-KB (v1)-derived training set (identified PSMs)
- Pairwise similarity configuration: normalized dot product with 0.05 m/z fragment peak-matching tolerance

## Outputs

- 500 selected reference spectra (submodular facility-location candidates)
- Serialized reference-spectra file consumed by the GLEAMS encoder for similarity-feature computation

## How to apply

Load the ~200,000 downsampled training MS/MS spectra derived from the MassIVE-KB (v1) human HCD spectral library (30 million PSMs at the basis of GLEAMS training). Compute the full pairwise similarity matrix between all training spectra using the normalized dot product with a 0.05 m/z fragment peak-matching tolerance, which serves as the precomputed metric for submodular optimization. Run apricot's facility-location function greedily on this similarity matrix to select 500 representative reference spectra; the facility-location objective rewards both diversity and coverage of the training spectral space. Export the 500 selected candidate spectra to a serialized file so they can be consumed by the GLEAMS encoder as the reference set for its similarity features. Key decision points are the fragment m/z tolerance for peak matching (0.05 m/z here) and the number of references (500), both of which were tuned for the GLEAMS 32-dimensional embedding task.

## Related tools

- **apricot** (Performs the greedy submodular facility-location selection of 500 reference spectra from the precomputed pairwise similarity matrix)
- **GLEAMS** (Downstream encoder whose similarity features consume the 500 selected reference spectra; training data and configuration live in its repository) — https://github.com/bittremieux/GLEAMS
- **Python 3.8** (Runtime environment in which the similarity matrix computation and apricot selection are executed (GLEAMS requires Python 3.8 on Linux with a CUDA GPU))

## Examples

```
from apricot import FacilityLocationSelection; refs = FacilityLocationSelection(500, metric='precomputed').fit_transform(similarity_matrix)
```

## Evaluation signals

- The selection output contains exactly 500 reference spectra.
- The pairwise similarity matrix was computed with the normalized dot product using a 0.05 m/z fragment peak-matching tolerance.
- The selected set is diverse and representative: greedy facility-location gains decrease over iterations and the chosen spectra span the peptide/precursor diversity of the MassIVE-KB training set.
- The serialized reference-spectra file loads cleanly and can be used by the GLEAMS encoder to compute similarity features for held-out spectra.
- Embeddings/clusterings trained or evaluated with this reference set reproduce the GLEAMS clustering behaviour (e.g. `gleams cluster --distance_threshold 0.3` yields sensible clusters).

## Limitations

- Computing the full pairwise similarity matrix scales quadratically with the number of training spectra, which is why the corpus was downsampled to ~200,000 spectra before selection.
- The 0.05 m/z fragment tolerance and 500-reference budget were chosen for the GLEAMS/MassIVE-KB human HCD setting; other instruments or organisms may require retuning.
- Reference quality depends on the MassIVE-KB PSM identifications (1% FDR, MSGF+); errors in the training labels propagate into the reference set.

## Evidence

- [methods] Pairwise similarity matrix with 0.05 m/z tolerance drives submodular selection: [evidence span withheld — anchor status: not_found]
- [readme] Training corpus underlying the candidate pool: "GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]"
- [readme] Configuration surface for tuning selection/training parameters: "Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the `gleams/config.py` file."
