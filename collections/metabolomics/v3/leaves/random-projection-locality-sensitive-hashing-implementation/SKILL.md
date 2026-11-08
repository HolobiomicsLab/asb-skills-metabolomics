---
name: random-projection-locality-sensitive-hashing-implementation
description: Use when you need to tune or characterize the collision probability P(m,n)([x_i,x_j])
  of an LSH scheme as a function of pair similarity, e.g. to pick (m,n) settings like
  (32,16), (64,32) or (32,32) for signal/noise classification.
when_to_use_negative:
- 'The goal is explicit candidate scoring rather than probabilistic retrieval: the
  article''s pipeline selects candidates via LSH and then computes cosine similarity
  explicitly for each candidate isotope pattern, so LSH alone does not replace explicit
  similarity scoring'
- The signal of interest is self-similar chemical noise (repeated contaminants), which
  the LSH approach would misclassify as true signal
- Signals with low relative intensity to noise peaks are the target, since the similarity
  measure on which the LSH is based drops and such signals can be lost
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3520
tools:
- name: Python
  role: Host language for implementing the hash family and the Monte-Carlo sweep over
    (m,n)
- name: TensorFlow 2
  role: GPU-accelerated backend for evaluating the hash family over the similarity
    grid
- name: pyproteizard-algorithm (proteolizard-algorithm)
  role: Reference implementation of the LSH scheme for ion-mobility MS raw data
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: pyproteolizard-data (proteolizard-data)
  role: Provides binned window vectors from timsTOF frames via OpenTIMS/pybind11 data
    access, which are the inputs to hashing
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:0052777e833db17f15844d7bbb18541ed40ce9e2923aa9da7fba89aeb5630b95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-algorithm
  tools:
  - Python
  - TensorFlow 2
  - pyproteizard-algorithm (proteolizard-algorithm)
  - pyproteolizard-data (proteolizard-data)
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# random-projection-locality-sensitive-hashing-implementation

## Summary

Implements a random-projection locality-sensitive hash h(x) = sign(<x,r>) for random unit vectors r and composes it into the paper's augmented LSH scheme with n AND-connectives and m OR-connectives, so that the (m,n) pair sets the effective similarity threshold for window-pair collisions.

## When to use

Use when you need to tune or characterize the collision probability P(m,n)([x_i,x_j]) of an LSH scheme as a function of pair similarity, e.g. to pick (m,n) settings like (32,16), (64,32) or (32,32) for signal/noise classification.

## When NOT to use

- The goal is explicit candidate scoring rather than probabilistic retrieval: the article's pipeline selects candidates via LSH and then computes cosine similarity explicitly for each candidate isotope pattern, so LSH alone does not replace explicit similarity scoring
- The signal of interest is self-similar chemical noise (repeated contaminants), which the LSH approach would misclassify as true signal
- Signals with low relative intensity to noise peaks are the target, since the similarity measure on which the LSH is based drops and such signals can be lost

## Inputs

- Pairs of window vectors (e.g. binned mass-axis windows of 10 Da length, offset by half a window length) represented as numeric arrays for cosine-similarity computation
- Grid of amplification parameters (m OR-connectives, n AND-connectives), e.g. (32,16), (64,32), (32,32)
- Monte-Carlo sample budget over random unit vectors r for the hash family

## Outputs

- Collision-probability surface P_{m,n}(s) mapping cosine similarity s to probability of retrieving a pair for each (m,n) setting
- Heatmap/surface plot (matplotlib) of P(m,n) vs similarity, comparable to Additional file 1 of 10.1186/s12859-022-04833-5
- Effective similarity threshold implied by a chosen (m,n), used to set false-positive and false-negative rates

## How to apply

Implement the single-hash random-projection scheme h(x) = sign(<x,r>) for random unit vectors r, whose collision probability is a monotone function of the cosine similarity between two window vectors (e.g. binned mass-axis windows from timsTOF frames). Compose the augmented scheme with m OR-connectives over blocks of n AND-connectives, so two windows collide only when all n hashes of at least one of the m blocks agree; the paper used (m,n) = (32,16), (64,32) and (32,32). Sweep a 2D grid over m and n and compute the collision probability surface P(m,n) as a function of cosine similarity by Monte-Carlo simulation of the hash family (Python/TensorFlow 2 supports GPU acceleration of the hash evaluations). Render P(m,n) as a heatmap/surface plot with matplotlib and compare against the reference plot in Additional file 1. Choose m and n so that retrieved pairs have a high probability of exceeding the target similarity: (32,16) acts as a low threshold with high true- and false-positive rates, (64,32) improves discrimination, and (32,32) acts as a strong filter that shrinks the false-positive rate.

## Related tools

- **Python** (Host language for implementing the hash family and the Monte-Carlo sweep over (m,n))
- **TensorFlow 2** (GPU-accelerated backend for evaluating the hash family over the similarity grid)
- **pyproteizard-algorithm (proteolizard-algorithm)** (Reference implementation of the LSH scheme for ion-mobility MS raw data) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **pyproteolizard-data (proteolizard-data)** (Provides binned window vectors from timsTOF frames via OpenTIMS/pybind11 data access, which are the inputs to hashing) — https://github.com/theGreatHerrLebert/proteolizard-data

## Examples

```
import numpy as np; R = np.random.randn(m*n, D); R /= np.linalg.norm(R, axis=1, keepdims=True); H = np.sign(W @ R.T); P = ((H[:n].all(axis=0)).reshape(m, -1).any(axis=0)).mean()
```

## Evaluation signals

- All computed collision probabilities P_{m,n}(s) lie in [0,1] for every (m,n) grid point
- P(m,n) is monotone in the expected directions: increasing n lowers collision probability at fixed similarity; increasing m raises it, matching (32,16) < (64,32)/(32,32) strictness ordering
- The simulated surface reproduces the reference plot in Additional file 1 of the publication (10.1186/s12859-022-04833-5)
- Single-hash collision probability is a monotone increasing function of cosine similarity between the two window vectors
- For a chosen (m,n), the implied similarity threshold matches the classification behavior reported in the paper (e.g. (32,16) very high true- and false-positive rates; (32,32) lower true-positive and false-positive rates)

## Limitations

- Chemical noise is self-similar and would be classified as 'true' signal by this LSH approach, so the (m,n) threshold cannot separate it
- Signals with low relative intensity to noise peaks could be lost, as the similarity measure on which the LSH is based drops
- Synthetic data used for ROC evaluation cannot fully reflect the behavior of real data, and no gold-standard ground-truth peak-level annotation of real data was available

## Evidence

- [results] augmented LSH with n AND connectives and m OR connectives, the algorithm allows for tuning of false-positive and false-negative rates: "By using an augmented LSH with n AND connectives and m OR connectives, the algorithm allows for tuning of false-positive and false-negative rates."
- [results] low threshold (32,16) yields very high true-positive and false-positive rates: [evidence span withheld — anchor status: not_found]
- [results] stricter (64,32) improves classification performance: [evidence span withheld — anchor status: not_found]
- [results] high threshold (32,32) as strong filter: "this setting may be employed as a strong filter to reduce large data sets."
- [results] GPU acceleration via TensorFlow 2: "our implementation supports acceleration by graphics processing units (GPUs) through the TensorFlow 2 software library [28]."
- [discussion] candidate selection followed by explicit cosine similarity: "When an unidentified window is now hashed, its key set can be used to select a set of candidate isotope patterns, for which cosine similarity is calculated explicitly."
- [readme] mzBucket repository and publication link: "# mzBucket
Locality-sensitive hashing for mass spectrometry data.

## Publication
See [here](https://doi.org/10.1186/s12859-022-04833-5) for the corresponding publication."
