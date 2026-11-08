---
name: cosine-similarity-hash-collision-probability-analysis
description: Use when you need to predict or tune the false-positive/false-negative
  trade-off of a random-projection LSH scheme — e.g., choosing (m,n) settings like
  (32,16), (64,32) or (32,32) before running signal classification on windowed MS
  data.
when_to_use_negative:
- The goal is explicit candidate scoring rather than probabilistic threshold tuning
  — the article's reference search instead selects candidates via hash keys and computes
  cosine similarity explicitly
- The data are self-similar chemical noise (repeated contaminants), which the LSH
  approach would classify as 'true' signal regardless of (m,n)
- Signals have very low relative intensity to noise peaks, since the similarity measure
  on which the LSH is based drops and windows can be lost
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3315
tools:
- name: Python
  role: Implementation language for the Monte-Carlo simulation of the hash family
    and plotting of the P(m,n) surface
- name: TensorFlow 2
  role: GPU-accelerated backend used for the LSH implementation whose (m,n) behavior
    is being reconstructed
- name: proteolizard-algorithm
  role: Reference implementation of the LSH scheme whose amplified collision probabilities
    the simulation reproduces
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: proteolizard-data
  role: Provides binned/windowed timsTOF frame vectors whose pairwise cosine similarities
    are the input to P_{m,n}(s)
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:9f683675b0d6e1965cc2c9eb6c47ef9e8eaf1e889b9cea1024fa00883204f7b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-algorithm
  tools:
  - Python
  - TensorFlow 2
  - proteolizard-algorithm
  - proteolizard-data
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cosine-similarity-hash-collision-probability-analysis

## Summary

Reconstructs and characterizes the augmented LSH scheme with n AND-connectives and m OR-connectives by Monte-Carlo simulation of the collision-probability surface P_{m,n}(s) as a function of pair similarity, showing how (m,n) sets the effective similarity threshold for signal classification.

## When to use

Use when you need to predict or tune the false-positive/false-negative trade-off of a random-projection LSH scheme — e.g., choosing (m,n) settings like (32,16), (64,32) or (32,32) before running signal classification on windowed MS data.

## When NOT to use

- The goal is explicit candidate scoring rather than probabilistic threshold tuning — the article's reference search instead selects candidates via hash keys and computes cosine similarity explicitly
- The data are self-similar chemical noise (repeated contaminants), which the LSH approach would classify as 'true' signal regardless of (m,n)
- Signals have very low relative intensity to noise peaks, since the similarity measure on which the LSH is based drops and windows can be lost

## Inputs

- Window vectors derived from binned mass axes (e.g., 10 Da windows offset by half a window length) represented by their cosine similarity s(x_i,x_j)
- Grid of (m,n) amplification parameters (e.g., (32,16), (64,32), (32,32))

## Outputs

- Collision-probability surface P_{m,n}(s) over the (m,n) grid, rendered as a heatmap/surface plot
- Effective similarity-threshold characterization per (m,n) setting for tuning false-positive and false-negative rates

## How to apply

Implement the random-projection hash h(x) = sign(<x,r>) for random unit vectors r, whose single-hash collision probability is a monotone function of the cosine similarity between two vectors (here, windowed mass-axis vectors from timsTOF frames). Compose the augmented LSH scheme with m OR-connectives over blocks of n AND-connectives, so two windows collide only when all n hashes of at least one of the m blocks agree, matching the paper's amplification parameters. Sweep a 2D grid over m and n and numerically compute the collision probability P(m,n) as a function of cosine similarity by Monte-Carlo simulation of the hash family. Render the P(m,n) values as a heatmap/surface plot (e.g., matplotlib), then select (m,n) pairs that place the effective similarity threshold where desired: low thresholds yield high true-positive and false-positive rates, stricter settings improve discrimination, and very high thresholds trade a lower true-positive rate for a shrunk false-positive rate. Validate against the reference plot in Additional file 1 of the publication (10.1186/s12859-022-04833-5).

## Related tools

- **Python** (Implementation language for the Monte-Carlo simulation of the hash family and plotting of the P(m,n) surface)
- **TensorFlow 2** (GPU-accelerated backend used for the LSH implementation whose (m,n) behavior is being reconstructed)
- **proteolizard-algorithm** (Reference implementation of the LSH scheme whose amplified collision probabilities the simulation reproduces) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **proteolizard-data** (Provides binned/windowed timsTOF frame vectors whose pairwise cosine similarities are the input to P_{m,n}(s)) — https://github.com/theGreatHerrLebert/proteolizard-data

## Examples

```
import numpy as np; s=np.linspace(0,1,101); h=(np.random.randn(1000,len(s))>0).astype(float); p_single=s/2+0.5; P=(1-(1-p_single**16)**32)  # P for (m,n)=(32,32)
```

## Evaluation signals

- All computed P_{m,n} values lie in [0,1]
- P is monotone in both m and n: increasing m (more OR blocks) raises collision probability at fixed similarity; increasing n (longer AND blocks) lowers it
- P_{m,n}(s) is monotonically increasing in similarity s for every (m,n) pair
- Simulated surface visually matches the reference plot in Additional file 1 of the publication
- Chosen (m,n) reproduces reported qualitative behavior: (32,16) gives very high true-positive and false-positive rates, (64,32) improves discrimination, (32,32) lowers both rates

## Limitations

- The collision-probability surface is derived from the hash family's similarity measure; signals with low relative intensity to noise peaks could be lost as the similarity measure drops
- Simulation on synthetic representations cannot fully reflect the behavior of real timsTOF data, where no gold-standard peak-level ground truth was available
- The (m,n) threshold governs window-level collisions only; per-peak classification and reference-based candidate confirmation (explicit cosine similarity) are separate downstream steps

## Evidence

- [results] By using an augmented LSH with n AND connectives and m OR connectives, the algorithm allows for tuning of false-positive and false-negative rates.: "By using an augmented LSH with n AND connectives and m OR connectives, the algorithm allows for tuning of false-positive and false-negative rates."
- [results] When setting a low threshold, e.g., $$(m,n) = (32,16)$$ ... a large fraction of windows are considered signal, resulting in very high true-positive and false-positive rates.: [evidence span withheld — anchor status: not_found]
- [results] A stricter discrimination with $$(m,n) = (64,32)$$ improves the performance significantly.: [evidence span withheld — anchor status: not_found]
- [results] Using a very high threshold, e.g., $$(m,n) = (32,32)$$ ... some windows are lost, resulting in a lower true-positive rate. As the false-positive rate shrinks as well: [evidence span withheld — anchor status: not_found]
- [discussion] The calculation of a window’s hash value requires the mass axis in a binned form with equally spaced bins.: "The calculation of a window’s hash value requires the mass axis in a binned form with equally spaced bins."
