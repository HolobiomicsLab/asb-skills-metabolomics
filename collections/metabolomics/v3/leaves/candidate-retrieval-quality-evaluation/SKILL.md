---
name: candidate-retrieval-quality-evaluation
description: Use when benchmarking an alternative ANN index structure (e.g. HNSW,
  LSH, product quantization) for candidate selection in open spectral library searching,
  to quantify candidate-retrieval quality against a baseline.
when_to_use_negative:
- The goal is a standard (closed) spectral library search without an ANN candidate-selection
  step — retrieval quality evaluation is specific to approximate indexing.
- The search engine performs exact nearest neighbor search (e.g. exhaustive SpectraST
  scoring), where candidate recall is 100% by definition and an ANN baseline is unnecessary.
- The spectral library or query set differs fundamentally from the high-resolution
  human HCD/HEK293 setting (e.g. low-resolution data) without re-validating preprocessing
  parameters such as the 1% intensity cutoff and 50-peak restriction.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ANN-SoLo
  role: Open spectral library search engine providing the random projection tree ANN
    baseline (1000 trees, 200,000 nodes inspected) and the cascade search framework
    against which alternative indexes are compared
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Used to concatenate spectral libraries while removing duplicates and to append
    shuffle-and-reposition decoys in a 1:1 ratio
- name: faiss
  role: Library of alternative ANN index structures (HNSW, product quantization, LSH)
    that can be implemented over the binned spectral vectors for benchmarking
  repo: https://github.com/facebookresearch/faiss
- name: Python
  role: Implementation language for alternative ANN indexes and the benchmark pipeline
    (with NumPy, pandas, Pyteomics)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:8fcbd10562972ef5607d9befbe0d2d7b280ce9cfb5077ff4fffb3caef17229e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - NumPy
  - Pyteomics
  - Python
  - SpectraST
  - faiss
  - pandas
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# candidate-retrieval-quality-evaluation

## Summary

Evaluates how well an approximate nearest neighbor (ANN) index recovers the true best-matching library spectra during open spectral library searching, by comparing candidate retrieval against an exact baseline (e.g. ANN-SoLo's full random projection tree ensemble) on the same query data.

## When to use

Use when benchmarking an alternative ANN index structure (e.g. HNSW, LSH, product quantization) for candidate selection in open spectral library searching, to quantify candidate-retrieval quality against a baseline.

## When NOT to use

- The goal is a standard (closed) spectral library search without an ANN candidate-selection step — retrieval quality evaluation is specific to approximate indexing.
- The search engine performs exact nearest neighbor search (e.g. exhaustive SpectraST scoring), where candidate recall is 100% by definition and an ANN baseline is unnecessary.
- The spectral library or query set differs fundamentally from the high-resolution human HCD/HEK293 setting (e.g. low-resolution data) without re-validating preprocessing parameters such as the 1% intensity cutoff and 50-peak restriction.

## Inputs

- HEK293 MS/MS query spectra (PRIDE PXD001468)
- MassIVE-KB peptide spectral library (version 2017/11/27), deduplicated with SpectraST and augmented with 1:1 shuffle-and-reposition decoys
- Preprocessed spectra: precursor/low-intensity peaks below 1% of base peak removed, top 50 most intense peaks kept, spectra with <10 peaks or <250 Da mass range discarded, intensities rank-transformed, unit-normalized 1 Da-binned sparse vectors

## Outputs

- ann_index_benchmark.csv with per-index candidate-retrieval and identification metrics
- Fraction of true best matches recovered versus the full random projection tree ensemble baseline
- Number of confident SSMs at the chosen FDR threshold per index

## How to apply

Build the alternative ANN index over the same unit-normalized, 1 Da-binned sparse spectral vectors used for candidate selection in ANN-SoLo's two-level cascade search (5 ppm precursor window followed by a 500 Da window, dot product at level one and shifted dot product at level two). Run the index on the HEK293 open search (PRIDE PXD001468 against the MassIVE-KB library version 2017/11/27, duplicates removed with SpectraST, shuffle-and-reposition decoys at 1:1). For each index, record the fraction of true best matches recovered relative to the full random projection tree ensemble baseline (1000 trees, 200,000 nodes inspected) and the number of confident SSMs after target-decoy FDR filtering with subgroup estimation (grouped by charge-normalized precursor mass difference, subgroups with fewer than 20 SSMs combined into a residual group). Write per-index metrics to ann_index_benchmark.csv for comparison of speed versus retrieval fidelity.

## Related tools

- **ANN-SoLo** (Open spectral library search engine providing the random projection tree ANN baseline (1000 trees, 200,000 nodes inspected) and the cascade search framework against which alternative indexes are compared) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Used to concatenate spectral libraries while removing duplicates and to append shuffle-and-reposition decoys in a 1:1 ratio)
- **faiss** (Library of alternative ANN index structures (HNSW, product quantization, LSH) that can be implemented over the binned spectral vectors for benchmarking) — https://github.com/facebookresearch/faiss
- **Python** (Implementation language for alternative ANN indexes and the benchmark pipeline (with NumPy, pandas, Pyteomics))

## Examples

```
python ann_index_benchmark.py --index hnsw --dataset PXD001468 --library MassIVE-KB_2017-11-27 --cascade '5ppm,500Da' --baseline 'rp_trees_1000_200000' --output ann_index_benchmark.csv
```

## Evaluation signals

- Candidate-retrieval quality is reported as the fraction of true best matches recovered relative to the full random projection tree ensemble (1000 trees, 200,000 nodes inspected); values near 1 indicate correct index configuration.
- Number of confident SSMs at the chosen FDR threshold should not degrade substantially relative to the ANN-SoLo baseline, which identifies more spectra than SpectraST or MSFragger on the human cell line data set.
- Target-decoy FDR must be estimated per subgroup (grouped by charge-normalized precursor mass difference), combining subgroups with fewer than 20 SSMs into a residual group; FDR filtering is skipped for subgroups with at least 20 SSMs only when the estimation is valid.
- Search time should be comparable to or better than the ANN-SoLo baseline, which is an order of magnitude faster than SpectraST (<2 hours versus almost a day on the HEK293 open search).
- Output file ann_index_benchmark.csv exists with one row of metrics per evaluated indexing technique.

## Limitations

- The suitability of alternative ANN indexing techniques (HNSW, LSH, product quantization) for spectral library searching was explicitly left as future work in the original paper, so expected retrieval-quality trade-offs are unknown a priori.
- Non-metric space indexing for direct shifted dot product candidate retrieval is only a potential, untested approach; level-two candidates still rely on the cascade strategy.
- The actual FDR may be underestimated when too-small subgroups are used, and accurate FDR estimation for open searches remains a standing research question.
- Search quality is approximate: ANN indexing selects only a limited set of candidate library spectra, so retrieval recall depends on index parameters (e.g. number of trees and nodes inspected in the baseline).

## Evidence

- [intro] During this open search ANN-SoLo uses an approximate nearest neighbor (ANN) index to efficiently find a limited set of the most similar library spectra: "During this open search ANN-SoLo uses an approximate nearest neighbor (ANN) index to efficiently find a limited set of the most similar library spectra for each query spectrum."
- [abstract] ANN-SoLo is an order of magnitude faster than SpectraST: "ANN-SoLo confidently identifies more spectra than SpectraST or MSFragger and achieves a speedup of an order of magnitude compared with SpectraST."
- [methods] they are grouped based on their charge-normalized precursor mass difference: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
- [discussion] An investigation into whether these techniques are suitable to speed up spectral library searching remains as future work.: "An investigation into whether these techniques are suitable to speed up spectral library searching remains as future work."
- [readme] ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra to compare to an unknown query"
