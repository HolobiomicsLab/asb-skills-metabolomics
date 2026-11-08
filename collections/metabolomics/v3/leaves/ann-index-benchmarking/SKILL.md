---
name: ann-index-benchmarking
description: Use when an open spectral library search is too slow with exact candidate
  selection and alternative ANN index structures must be compared against the ANN-SoLo
  random projection tree baseline on large peptide spectral libraries.
when_to_use_negative:
- When the task is a standard (closed) search for unmodified peptides rather than
  an open modification search — the cascade's second 500 Da window and shifted dot
  product exist for modification-tolerant matching
- When searching small spectral libraries where exhaustive candidate scoring is already
  fast enough that ANN approximation and its recall loss add no value
- When the target is sequence database searching (e.g. MSFragger-style fragment indexing)
  rather than spectral library searching — prior multidimensional indexing work targeted
  sequence databases, not spectral libraries
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: ANN-SoLo
  role: Provides the baseline random projection tree ANN index and cascade open search
    (dot product level one, shifted dot product level two) against which alternative
    indexes are benchmarked
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Used to concatenate the spectral library while removing duplicate peptide
    ions before benchmarking
- name: Faiss
  role: Library for implementing alternative ANN index structures (e.g. HNSW, product
    quantization) over the binned spectral vectors
  repo: https://github.com/facebookresearch/faiss
- name: Pyteomics
  role: Python library used for handling mass spectrometry data during the benchmark
    workflow
- name: NumPy
  role: Array computation for building and querying ANN indexes over unit-normalized
    sparse spectral vectors
- name: pandas
  role: Writing and managing the per-index comparison metrics in ann_index_benchmark.csv
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:77ba33ccf6533495cdb9412683833fa03b19ab679d5d5e2bc48380409568b01f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Faiss
  - NumPy
  - Pyteomics
  - Python
  - SpectraST
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# ann-index-benchmarking

## Summary

Benchmark alternative approximate nearest neighbor (ANN) indexing structures (e.g. HNSW graphs, locality-sensitive hashing, product quantization) against the ANN-SoLo random projection tree baseline for candidate selection in two-level cascade open spectral library searching on the HEK293 data set.

## When to use

Use when an open spectral library search is too slow with exact candidate selection and alternative ANN index structures must be compared against the ANN-SoLo random projection tree baseline on large peptide spectral libraries.

## When NOT to use

- When the task is a standard (closed) search for unmodified peptides rather than an open modification search — the cascade's second 500 Da window and shifted dot product exist for modification-tolerant matching
- When searching small spectral libraries where exhaustive candidate scoring is already fast enough that ANN approximation and its recall loss add no value
- When the target is sequence database searching (e.g. MSFragger-style fragment indexing) rather than spectral library searching — prior multidimensional indexing work targeted sequence databases, not spectral libraries

## Inputs

- HEK293 human cell line MS/MS spectra downloaded from PRIDE project PXD001468
- MassIVE-KB peptide spectral library (version 2017/11/27), deduplicated with SpectraST
- Shuffle-and-reposition decoy library spectra in 1:1 target:decoy ratio
- Alternative ANN index implementations (HNSW, LSH, or product quantization, e.g. via FAISS) over unit-normalized 1 Da-binned sparse spectral vectors

## Outputs

- ann_index_benchmark.csv with candidate-retrieval quality and identification metrics for each indexing technique
- Fraction of true best matches recovered relative to the full random projection tree ensemble (1000 trees, 200,000 nodes inspected)
- Number of confident SSMs at the chosen FDR threshold per index
- Search time per index on the HEK293 open search

## How to apply

Download the HEK293 MS/MS spectra (PRIDE PXD001468) and the MassIVE-KB peptide spectral library (version 2017/11/27), removing duplicate peptide ions with SpectraST and appending shuffle-and-reposition decoys in a 1:1 ratio as in the ANN-SoLo paper. Preprocess query and library spectra identically: remove noise peaks below 1% of the most intense peak, keep the 50 most intense peaks, discard spectra with fewer than 10 peaks or a mass range under 250 Da, and rank-transform intensities before unit-normalizing into 1 Da-binned sparse spectral vectors for candidate selection. Implement the alternative ANN index (hierarchical navigable small world graphs, locality-sensitive hashing, or product quantization) over these vectors and run the two-level cascade open search — a 5 ppm precursor window level one scored with the dot product, followed by a 500 Da window level two scored with the shifted dot product. Filter SSMs via the target-decoy strategy with subgroup FDR estimation, grouping by charge-normalized precursor mass difference and pooling subgroups with fewer than 20 SSMs into a residual group. Record candidate-retrieval quality (fraction of true best matches recovered versus the full random projection tree ensemble of 1000 trees with 200,000 nodes inspected) and confident identifications at the chosen FDR threshold, writing per-index metrics to ann_index_benchmark.csv.

## Related tools

- **ANN-SoLo** (Provides the baseline random projection tree ANN index and cascade open search (dot product level one, shifted dot product level two) against which alternative indexes are benchmarked) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Used to concatenate the spectral library while removing duplicate peptide ions before benchmarking)
- **Faiss** (Library for implementing alternative ANN index structures (e.g. HNSW, product quantization) over the binned spectral vectors) — https://github.com/facebookresearch/faiss
- **Pyteomics** (Python library used for handling mass spectrometry data during the benchmark workflow)
- **NumPy** (Array computation for building and querying ANN indexes over unit-normalized sparse spectral vectors)
- **pandas** (Writing and managing the per-index comparison metrics in ann_index_benchmark.csv)

## Evaluation signals

- Candidate-retrieval quality: the fraction of true best matches recovered should approach the full random projection tree ensemble baseline (1000 trees, 200,000 nodes inspected)
- Number of confident SSMs at the chosen FDR threshold should be comparable to ANN-SoLo baseline results on HEK293
- FDR estimation must only be applied to subgroups containing at least 20 SSMs, with smaller groups combined into a residual group
- Search time should be recorded per index; the baseline context is ANN-SoLo reducing search time from almost a day (SpectraST) to under two hours
- ann_index_benchmark.csv exists with one row of comparison metrics per indexing technique

## Limitations

- The suitability of alternative ANN techniques (HNSW, LSH, product quantization) for spectral library searching was explicitly left as future work in the source paper, so no published validation exists
- Actual FDR may be underestimated when too-small subgroups are used; pooling subgroups with fewer than 20 SSMs mitigates but does not eliminate this
- Accurate FDR estimation for open searches is a standing research question, so cross-index identification counts may not be strictly comparable
- The ANN index approximates candidate retrieval, so some true best matches will be missed relative to exhaustive search
- ANN-SoLo requires Python 3.6 to 3.9 (Python 3.10 and newer not supported per README), which constrains the environment for re-implementation

## Evidence

- [abstract] Speedup baseline: "ANN-SoLo confidently identifies more spectra than SpectraST or MSFragger and achieves a speedup of an order of magnitude compared with SpectraST."
- [methods] Decoy construction: "decoy spectra were added in a 1:1 ratio using the shuffle-and-reposition method [41]"
- [methods] Peak preprocessing thresholds: "noise peaks with an intensity below 1 % of the intensity of the most intense peak are removed"
- [methods] FDR subgrouping rule: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
- [results] Open-search sensitivity gain: "ANN-SoLo achieves a 25 % increase in identifications when performing an open search compared to a standard search"
- [discussion] Future-work gap motivating the benchmark: "An investigation into whether these techniques are suitable to speed up spectral library searching remains as future work."
- [readme] ANN role in open search: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra"
