---
name: spectral-library-searching
description: Use when identifying peptide tandem mass spectra against a large spectral
  library where modifications may be present and a full pairwise comparison of all
  library spectra is too slow, e.g. HEK293 human cell line HCD data.
when_to_use_negative:
- When the search goal is a standard narrow-mass database search without expected
  modifications, since the cascade open search and ANN indexing add overhead that
  a conventional sequence database search (e.g. MSFragger) handles directly
- When the spectral library is small enough that exhaustive pairwise scoring is tractable,
  as approximate ANN candidate selection sacrifices recall for speed and may miss
  true best matches
- When FDR estimation requires extremely fine-grained subgroups, because FDR filtering
  is only applied to subgroups containing at least 20 SSMs and small subgroups may
  underestimate the actual FDR
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ANN-SoLo
  role: 'Implements the full open modification spectral library search pipeline: spectral
    preprocessing, ANN indexing, two-level cascade search, shifted dot product scoring,
    and FDR estimation.'
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Used to concatenate spectral libraries while removing duplicates, and serves
    as the baseline spectral library search engine for speed and identification comparisons.
- name: MSFragger
  role: Baseline sequence database search engine optimized for open modification searching
    against which ANN-SoLo identifications are compared.
- name: Pyteomics
  role: Supports parsing proteomics data and executing the search workflow steps in
    the Python environment.
- name: Faiss
  role: General-purpose approximate nearest neighbor search library whose alternative
    index structures (HNSW, LSH, product quantization) can be benchmarked against
    ANN-SoLo's random projection trees for open spectral library candidate selection.
  repo: https://github.com/facebookresearch/faiss
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:34c2b6e71998ecc8e65f6b1f9e499c0da96eaf98d097fc65cfec795888d3ce6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Faiss
  - MSFragger
  - NumPy
  - Pyteomics
  - Python
  - SpectraST
  - pandas
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spectral-library-searching

## Summary

Open modification spectral library searching using a two-level cascade with approximate nearest neighbor (ANN) indexing for candidate selection, followed by scoring with the dot product and shifted dot product and target-decoy FDR filtering, as implemented in ANN-SoLo for fast and sensitive identification of modified peptides.

## When to use

Use when identifying peptide tandem mass spectra against a large spectral library where modifications may be present and a full pairwise comparison of all library spectra is too slow, e.g. HEK293 human cell line HCD data.

## When NOT to use

- When the search goal is a standard narrow-mass database search without expected modifications, since the cascade open search and ANN indexing add overhead that a conventional sequence database search (e.g. MSFragger) handles directly
- When the spectral library is small enough that exhaustive pairwise scoring is tractable, as approximate ANN candidate selection sacrifices recall for speed and may miss true best matches
- When FDR estimation requires extremely fine-grained subgroups, because FDR filtering is only applied to subgroups containing at least 20 SSMs and small subgroups may underestimate the actual FDR

## Inputs

- Query MS/MS spectra in standard formats such as mzML or MGF (e.g. raw files from PRIDE project PXD001468)
- Peptide spectral library, e.g. MassIVE-KB version 2017/11/27 or the human HCD spectral library from NIST version 2016/09/12
- Optional decoy spectra (shuffle-and-reposition method in a 1:1 ratio)

## Outputs

- Spectrum-spectrum matches (SSMs) with FDR-filtered confident identifications at a chosen FDR threshold
- Search-time benchmark and candidate-retrieval quality metrics (e.g. fraction of true best matches recovered versus the full index)
- ann_index_benchmark.csv of comparison metrics for each indexing technique

## How to apply

Preprocess both query spectra and library spectra identically: remove precursor and low-intensity peaks below 1% of the most intense peak, restrict to the 50 most intense peaks, discard spectra with fewer than 10 peaks or a mass range under 250 Da, and rank-transform intensities. Run a two-level cascade search: level one identifies unmodified peptides with a precursor mass filter (5 ppm) using the standard dot product on unit-normalized, 1 Da-binned sparse spectral vectors; level two identifies modified peptides with a 500 Da precursor window using the shifted dot product to account for peak shifts. Use an ANN index (random projection tree ensemble, 1000 trees, 200,000 nodes inspected) to select a limited set of the most similar library spectra for each query rather than searching the full library. Add shuffle-and-reposition decoys in a 1:1 ratio and filter SSMs using the target-decoy strategy with subgroup FDR estimation, grouping by charge-normalized precursor mass difference and combining subgroups with fewer than 20 SSMs into a residual group. Compare confident identifications at the chosen FDR threshold and search time against alternative search engines (SpectraST, MSFragger).

## Related tools

- **ANN-SoLo** (Implements the full open modification spectral library search pipeline: spectral preprocessing, ANN indexing, two-level cascade search, shifted dot product scoring, and FDR estimation.) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Used to concatenate spectral libraries while removing duplicates, and serves as the baseline spectral library search engine for speed and identification comparisons.)
- **MSFragger** (Baseline sequence database search engine optimized for open modification searching against which ANN-SoLo identifications are compared.)
- **Pyteomics** (Supports parsing proteomics data and executing the search workflow steps in the Python environment.)
- **Faiss** (General-purpose approximate nearest neighbor search library whose alternative index structures (HNSW, LSH, product quantization) can be benchmarked against ANN-SoLo's random projection trees for open spectral library candidate selection.) — https://github.com/facebookresearch/faiss

## Examples

```
ann-solo --spectra-file spectra.mzML --library-file library.mgf --precursor-tolerance 5 --open-mods 500 --fdr 0.01 --out results.tsv
```

## Evaluation signals

- The number of confident SSMs at a target FDR (e.g. 1%) should match or exceed baseline results — on the HEK293 data set ANN-SoLo confidently identifies more spectra than SpectraST or MSFragger.
- Search time should be substantially faster than exhaustive search — ANN-SoLo is an order of magnitude faster than SpectraST on the HEK293 data, reducing search time from almost a day to under two hours.
- Candidate retrieval quality: the fraction of true best matches recovered versus a full exhaustive search should be reported for each ANN index configuration (e.g. 1000 trees, 200,000 nodes inspected).
- The fraction of additional identifications in the open search versus a standard search should correspond to modified peptides (25% increase on iPRG2012).
- Output metrics must be written to ann_index_benchmark.csv for downstream comparison of alternative indexing techniques.

## Limitations

- On very large spectral libraries the ANN index trades recall for speed; the fraction of true best matches recovered versus exhaustive search must be monitored.
- FDR may be underestimated when too-small subgroups (fewer than 20 SSMs) are combined or treated independently.
- Accurate FDR estimation for open modification searches remains an unresolved research question, and alternative subgrouping strategies may be needed.
- ANN-SoLo requires Python 3.6 to 3.9; the implementation is not compatible with Python 3.10 and newer.

## Evidence

- [readme] ANN-SoLo is a spectral library search engine for fast and accurate open modification searching.: "ANN-SoLo is a spectral library search engine for fast and accurate open modification searching."
- [readme] ANN indexing selects only a limited number of library spectra per query.: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra to compare to an unknown query"
- [abstract] ANN-SoLo outperforms SpectraST and MSFragger in identifications and speed.: "ANN-SoLo confidently identifies more spectra than SpectraST or MSFragger and achieves a speedup of an order of magnitude compared with SpectraST."
- [methods] Cascade search consists of two levels: candidate selection and candidate ranking.: [evidence span withheld — anchor status: not_found]
- [methods] Spectra are preprocessed by removing low-intensity peaks and restricting to 50 most intense peaks.: [evidence span withheld — anchor status: not_found]
- [methods] FDR calculation uses the target-decoy strategy in two phases after both cascade levels.: [evidence span withheld — anchor status: not_found]
- [methods] SSMs are grouped by charge-normalized precursor mass difference with a minimum subgroup size of 20.: [evidence span withheld — anchor status: not_found]
- [methods] The HEK293 data set and MassIVE-KB spectral library are used to benchmark the search.: [evidence span withheld — anchor status: not_found]
