---
name: pairwise-distance-matrix-construction
description: Use when you have processed MS/MS spectra per LC-MS/MS feature and need
  a square pairwise spectral-similarity matrix as input for k-medoids clustering or
  feature-set testing in msFeaST.
when_to_use_negative:
- You already have a similarity/distance matrix — this skill constructs one from raw
  per-feature spectra and would duplicate work or overwrite the existing matrix.
- Your goal is k-medoids clustering or globaltest statistical testing on the already-computed
  matrix — those are downstream steps; the matrix must exist first.
- Your input is a quantification/metadata table without MS/MS spectra — the skill
  requires per-feature MS/MS spectra as input.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: msFeaST
  role: 'Executes the spectral similarity computation stage: it applies the user-selected
    similarity score to the processed MS/MS spectra and emits the distance matrix
    used for k-medoid clustering and t-SNE embedding.'
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: Downstream consumer of the similarity-derived feature groupings (feature group-wise
    statistical testing), not the matrix construction itself.
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:0a9d21db5fdcaa63dc0746aa59a43f3a16ff5da4cea58730a48d09842a95822a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pairwise-distance-matrix-construction

## Summary

Computes all pairwise spectral similarity scores between feature MS/MS spectra in an LC-MS/MS untargeted metabolomics dataset and assembles them into a square, symmetric similarity/distance matrix. This matrix is the canonical intermediate that msFeaST feeds into k-medoids feature grouping and downstream t-SNE embedding.

## When to use

Use when you have processed MS/MS spectra per LC-MS/MS feature and need a square pairwise spectral-similarity matrix as input for k-medoids clustering or feature-set testing in msFeaST.

## When NOT to use

- You already have a similarity/distance matrix — this skill constructs one from raw per-feature spectra and would duplicate work or overwrite the existing matrix.
- Your goal is k-medoids clustering or globaltest statistical testing on the already-computed matrix — those are downstream steps; the matrix must exist first.
- Your input is a quantification/metadata table without MS/MS spectra — the skill requires per-feature MS/MS spectra as input.

## Inputs

- Processed MS/MS spectra of LC-MS/MS features (one spectrum per feature, e.g. from the Kathib et al. (2024) example dataset bundled in the msFeaST GitHub repository)
- User-selected similarity score type (the msFeaST spectral similarity module's configurable score)

## Outputs

- spectral_similarity_matrix.csv — a square, symmetric similarity/distance matrix with one row and column per feature and self-similarity on the diagonal
- Matrix entries bounded within the valid range of the chosen similarity score

## How to apply

Load the processed MS/MS spectra of your feature table (e.g. the Kathib et al. example data bundled with the msFeaST repository) and apply the spectral preprocessing/peak handling configured by the msFeaST spectral similarity module (methods section 2.1) so fragment peaks are comparable across features. Compute all pairwise similarity scores between feature MS/MS spectra using the msFeaST spectral similarity computation stage, with the similarity score selected by the user as part of the pipeline configuration. Assemble the scores into a square, symmetric similarity/distance matrix with one row and column per feature and self-similarity on the diagonal. Write the matrix to spectral_similarity_matrix.csv as the canonical output consumed by msFeaST's k-medoids clustering step (and, in the broader workflow, t-SNE embedding).

## Related tools

- **msFeaST** (Executes the spectral similarity computation stage: it applies the user-selected similarity score to the processed MS/MS spectra and emits the distance matrix used for k-medoid clustering and t-SNE embedding.) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Downstream consumer of the similarity-derived feature groupings (feature group-wise statistical testing), not the matrix construction itself.) — https://github.com/kevinmildau/msFeaST

## Examples

```
conda activate msfeast_environment; jupyter-notebook  # then run msfeast_pipeline_mushroom_type_comparison.ipynb to produce spectral_similarity_matrix.csv
```

## Evaluation signals

- The emitted matrix is square (same number of rows and columns).
- The matrix is symmetric (entry (i,j) equals entry (j,i)).
- The matrix has one row/column per input feature — dimension matches the feature count.
- The diagonal holds self-similarity (1.0) for all features.
- All pairwise entries lie within the chosen similarity score's valid bounded range.

## Limitations

- The similarity score type must be selected by the user as part of the msFeaST pipeline configuration; the matrix construction is only valid for the score actually selected.
- The msFeaST pre-processing and pipeline workflow has been tested on macOS and is expected to work on Linux; Windows support is still being worked on.
- Only the bundled dashboard plus ready-made data can be inspected without installing dependencies — running the full pipeline (and thus this matrix step) requires the conda/python/R environment described in the msFeaST setup.

## Evidence

- [abstract] Spectral similarity-based feature grouping via k-medoids clustering: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [readme] msFeaST pipeline consumes the similarity matrix for clustering: [evidence span withheld — anchor status: external_doc]
- [abstract] Matrix is the distance matrix used downstream: "Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package"
