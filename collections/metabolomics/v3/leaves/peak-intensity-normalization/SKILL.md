---
name: peak-intensity-normalization
description: Use when MS/MS spectra (m/z + intensity peak lists from MSP/MGF metabolomics
  library files) must be intensity-normalized and de-weighted before computing spectral-entropy
  similarity between spectrum pairs.
when_to_use_negative:
- Peak intensities are already normalized and de-weighted per the spectral-entropy
  formulation — re-normalizing would distort the entropy similarity scores
- The goal is cross-sample feature-intensity normalization for MS1 quantification
  (e.g. TIC or quantile normalization across runs) rather than intra-spectrum weighting
  for MS/MS similarity scoring
- The task is raw spectral-library search at billion-entry scale, where Flash entropy
  search's precomputation (not per-pair normalized scoring) is the intended mechanism
tools:
- name: Python
  role: Parses MSP/MGF peak lists, implements intensity normalization, the de-weighting
    transformation, Shannon entropy computation, and assembles the similarity matrix
    CSV
- name: EntropySearch
  role: Reference implementation of spectral library search scored by entropy similarity,
    whose scoring formulation this normalization step feeds
  repo: https://github.com/YuanyueLi/EntropySearch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-023-02012-9
    title: Flash entropy search to query all mass spectral libraries in real time
  source_content_hash: sha256:bc601dddc184dc83bdbfc08bff5adf40418b2c58b465be3f798069c8d66dd644
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/EntropySearch
  tools:
  - EntropySearch
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# peak-intensity-normalization

## Summary

Normalizes MS/MS peak-list intensities and applies the spectral-entropy de-weighting transformation (Li et al., Nat Methods 2021) so that high-abundance peaks do not dominate Shannon-entropy-based similarity scores used by EntropySearch for small-molecule library search.

## When to use

Use when MS/MS spectra (m/z + intensity peak lists from MSP/MGF metabolomics library files) must be intensity-normalized and de-weighted before computing spectral-entropy similarity between spectrum pairs.

## When NOT to use

- Peak intensities are already normalized and de-weighted per the spectral-entropy formulation — re-normalizing would distort the entropy similarity scores
- The goal is cross-sample feature-intensity normalization for MS1 quantification (e.g. TIC or quantile normalization across runs) rather than intra-spectrum weighting for MS/MS similarity scoring
- The task is raw spectral-library search at billion-entry scale, where Flash entropy search's precomputation (not per-pair normalized scoring) is the intended mechanism

## Inputs

- MS/MS spectra as peak lists (m/z and intensity values) with precursor metadata, parsed from a public metabolomics mass spectral library file in MSP or MGF text format

## Outputs

- Normalized, de-weighted intensity vectors per spectrum (probability distributions over peaks)
- Pairwise entropy similarity scores in [0, 1] for every spectrum pair
- Symmetric similarity matrix written to entropy_similarity_matrix.csv

## How to apply

Parse peak lists (m/z and intensity pairs with precursor metadata) from MSP/MGF text-format library files using Python. Normalize each spectrum's peak intensities, then apply the intensity de-weighting transformation of the spectral-entropy formulation (Li et al., Nat Methods 2021) so high-abundance peaks do not dominate the entropy calculation. Treat the resulting weights as a probability distribution (weights sum to 1) before computing Shannon entropies. For every spectrum pair, compute the Shannon entropy of each individual peak list and of their merged (union) peak list, and derive the entropy similarity score from the merged and unmerged entropies. Assemble all pairwise scores into a symmetric similarity matrix written to entropy_similarity_matrix.csv.

## Related tools

- **Python** (Parses MSP/MGF peak lists, implements intensity normalization, the de-weighting transformation, Shannon entropy computation, and assembles the similarity matrix CSV)
- **EntropySearch** (Reference implementation of spectral library search scored by entropy similarity, whose scoring formulation this normalization step feeds) — https://github.com/YuanyueLi/EntropySearch

## Examples

```
import numpy as np; w = ints / ints.max(); p = w / w.sum(); S = -np.sum(p[p>0] * np.log(p[p>0]))
```

## Evaluation signals

- Every pairwise similarity score lies within [0, 1]
- The diagonal self-similarity of the matrix equals 1
- The similarity matrix is symmetric
- Normalized de-weighted intensities form a valid probability distribution (non-negative, summing to 1) before Shannon entropy is computed
- After de-weighting, the highest-abundance peak no longer dominates the entropy contribution relative to its raw intensity fraction

## Limitations

- The de-weighting transformation is specific to the spectral-entropy formulation of Li et al. (Nat Methods 2021); applying arbitrary normalization (e.g. raw TIC scaling) instead will not reproduce entropy similarity scores
- Full entropy similarity comparison of a single MS/MS spectrum against billion-entry libraries takes more than 8 hours without the Flash entropy search acceleration; this skill targets pairwise score construction, not exhaustive large-library search
- Entropy-based peak lists must have positive intensities; zero-weight peaks must be excluded from the Shannon entropy summation

## Evidence

- [other] Normalize peak intensities and apply the intensity de-weighting transformation of the spectral-entropy formulation (Li et al., Nat Methods 2021) so that high-abundance peaks do not dominate the entropy calculation.: [evidence span withheld — anchor status: not_found]
- [other] Compute Shannon entropies of individual and merged peak lists: [evidence span withheld — anchor status: not_found]
- [readme] Entropy similarity as the library-search mechanism: "Search spectral library with entropy similarity. Download the software from here: [https://github.com/YuanyueLi/EntropySearch/releases]"
