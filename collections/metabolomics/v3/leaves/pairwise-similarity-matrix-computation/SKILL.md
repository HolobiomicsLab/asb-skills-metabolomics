---
name: pairwise-similarity-matrix-computation
description: Use when you need to score every pair of MS/MS peak lists (m/z and intensity
  values with precursor metadata) against each other with entropy similarity instead
  of dot-product similarity, e.g. to build a library-search similarity matrix.
when_to_use_negative:
- The analysis goal is real-time querying of billion-entry spectral libraries rather
  than explicit pairwise scoring — flash entropy search is the article's accelerated
  alternative, since full pairwise entropy/dot-product comparisons of a single spectrum
  take more than 8 hours against such libraries.
- Input spectra are already a precomputed similarity matrix or a feature table rather
  than raw peak lists needing entropy-based scoring.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Python
  role: Parses MSP/MGF peak lists, applies the intensity de-weighting transformation,
    and computes Shannon entropies of merged and unmerged peak lists to derive pairwise
    entropy similarity scores
- name: EntropySearch
  role: Library-search tool whose scoring mechanism is entropy similarity based on
    the spectral-entropy formulation of Li et al. (Nat Methods 2021)
  repo: https://github.com/YuanyueLi/EntropySearch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-023-02012-9
    title: Flash entropy search to query all mass spectral libraries in real time
  source_content_hash: sha256:ad10b831ef4b6e6888cbfdd409e929370892dcd348a1083d343280e6bcd87b84
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/EntropySearch
  tools:
  - EntropySearch
  - Python
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# pairwise-similarity-matrix-computation

## Summary

Compute a symmetric matrix of pairwise spectral-entropy similarity scores between MS/MS spectra, replacing dot-product similarity per Li et al.'s spectral-entropy formulation (Nat Methods 2021), as used by the EntropySearch library-search tool.

## When to use

Use when you need to score every pair of MS/MS peak lists (m/z and intensity values with precursor metadata) against each other with entropy similarity instead of dot-product similarity, e.g. to build a library-search similarity matrix.

## When NOT to use

- The analysis goal is real-time querying of billion-entry spectral libraries rather than explicit pairwise scoring — flash entropy search is the article's accelerated alternative, since full pairwise entropy/dot-product comparisons of a single spectrum take more than 8 hours against such libraries.
- Input spectra are already a precomputed similarity matrix or a feature table rather than raw peak lists needing entropy-based scoring.

## Inputs

- MS/MS spectra peak lists (m/z and intensity pairs with precursor metadata) from public metabolomics mass spectral library files in MSP/MGF text format

## Outputs

- Symmetric pairwise entropy similarity matrix (e.g. entropy_similarity_matrix.csv) with scores in [0, 1] and unit diagonal

## How to apply

Parse MS/MS spectra (m/z and intensity peak lists with precursor metadata) from public metabolomics mass spectral library files in MSP/MGF text format using Python. Normalize the peak intensities and apply the intensity de-weighting transformation of the spectral-entropy formulation so that high-abundance peaks do not dominate the entropy calculation. For every spectrum pair, compute the Shannon entropy of each individual peak list and of their merged (union) peak list, and derive the entropy similarity score from the merged versus unmerged entropies. Assemble all pairwise scores into a symmetric similarity matrix (e.g. entropy_similarity_matrix.csv); the same entropy-similarity scoring is the mechanism EntropySearch uses to query a spectral library. Validate that every score lies in [0, 1], the diagonal self-similarity equals 1, and the matrix is symmetric.

## Related tools

- **Python** (Parses MSP/MGF peak lists, applies the intensity de-weighting transformation, and computes Shannon entropies of merged and unmerged peak lists to derive pairwise entropy similarity scores)
- **EntropySearch** (Library-search tool whose scoring mechanism is entropy similarity based on the spectral-entropy formulation of Li et al. (Nat Methods 2021)) — https://github.com/YuanyueLi/EntropySearch

## Examples

```
python backend/main.py
```

## Evaluation signals

- Every pairwise similarity score lies within [0, 1]
- Diagonal self-similarity of the matrix equals 1
- The similarity matrix is symmetric (score(i,j) == score(j,i))
- Scores computed with the intensity de-weighting step do not let high-abundance peaks dominate the entropy calculation (compare with/without de-weighting on spectra with one dominant peak)
- Entropy similarity results rank consistently with dot-product similarity on known compound-identification benchmarks (entropy similarity outperforms MS/MS dot product similarity per Li et al. 2021)

## Limitations

- Full entropy similarity comparison of a single MS/MS spectrum takes more than 8 hours against billion-entry public repositories, so exhaustive pairwise scoring does not scale without the flash entropy search acceleration (10,000x speedup, <2 s for 1 billion spectra).
- The preprint (rs-2693233) is superseded by the peer-reviewed Nature Methods article (10.1038/s41592-023-02012-9); cite the published version.
- The software is built only for Windows 10 (Windows-2019), Ubuntu 20.04 (Focal Fossa), and MacOS 11 (Big Sur); other platforms require building from source with build_linux.sh or build_win.bat.

## Evidence

- [intro] entropy similarity based on the spectral-entropy formulation of Li et al. (Nat Methods 2021): [evidence span withheld — anchor status: not_found]
- [readme] Search spectral library with entropy similarity: "Search spectral library with entropy similarity. Download the software from here: [https://github.com/YuanyueLi/EntropySearch/releases]"
- [abstract] Full dot product or entropy similarity comparisons take more than 8 hours: "Full dot product or entropy similarity comparisons of a single MS/MS spectrum take more than 8 hours."
- [readme] Spectral entropy outperforms MS/MS dot product similarity: [evidence span withheld — anchor status: not_found]
