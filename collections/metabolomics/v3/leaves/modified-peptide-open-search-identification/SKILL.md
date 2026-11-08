---
name: modified-peptide-open-search-identification
description: Use when MS/MS spectra (e.g. HEK293 human cell line data from PRIDE PXD001468)
  must be searched against a large spectral library (e.g. MassIVE-KB 2017/11/27) for
  peptides carrying any modification, with FDR-controlled spectrum-spectrum matches.
when_to_use_negative:
- A standard closed search suffices (no unknown modifications expected) — ANN-SoLo
  achieves a 25% increase in identifications for open search, but a narrow precursor
  window search is the simpler choice otherwise
- The analysis is a sequence database search rather than spectral library searching
  (e.g. MSFragger-style theoretical fragment indexing)
- The spectral library is small enough that exhaustive candidate scoring is already
  fast, making ANN candidate selection unnecessary
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: ANN-SoLo
  role: Open modification spectral library search engine executing the cascade search,
    ANN candidate selection, and shifted dot product scoring
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Concatenating spectral libraries while removing duplicate peptide ions before
    searching
- name: faiss
  role: Source of alternative ANN index structures (e.g. HNSW, product quantization)
    to benchmark against the random projection tree baseline
  repo: https://github.com/facebookresearch/faiss
- name: Python
  role: Implementation and scripting language for preprocessing, search orchestration,
    and benchmark metric reporting (with NumPy, pandas, Pyteomics)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:007311eb283410764f5938299d2f01ffe369b80efcc32a92f18667e51266763e
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# modified-peptide-open-search-identification

## Summary

Identify modified peptides from MS/MS spectra by ANN-SoLo's two-level cascade open spectral library search: unmodified peptides first via dot product in a narrow precursor window, then modified peptides via the shifted dot product in a 500 Da window, with target-decoy subgroup FDR control.

## When to use

Use when MS/MS spectra (e.g. HEK293 human cell line data from PRIDE PXD001468) must be searched against a large spectral library (e.g. MassIVE-KB 2017/11/27) for peptides carrying any modification, with FDR-controlled spectrum-spectrum matches.

## When NOT to use

- A standard closed search suffices (no unknown modifications expected) — ANN-SoLo achieves a 25% increase in identifications for open search, but a narrow precursor window search is the simpler choice otherwise
- The analysis is a sequence database search rather than spectral library searching (e.g. MSFragger-style theoretical fragment indexing)
- The spectral library is small enough that exhaustive candidate scoring is already fast, making ANN candidate selection unnecessary

## Inputs

- HEK293 human cell line MS/MS spectra downloaded from PRIDE project PXD001468
- MassIVE-KB peptide spectral library (version 2017/11/27), deduplicated with SpectraST
- Shuffle-and-reposition decoy spectra appended in a 1:1 target:decoy ratio
- Preprocessed query spectra files (precursor- and noise-filtered, top-50 peaks, rank-transformed intensities)

## Outputs

- Confident spectrum-spectrum matches (SSMs) with modified and unmodified peptide identifications at the chosen FDR threshold
- Per-level cascade search results (unmodified dot product matches; open-search shifted dot product matches)
- Search-time and candidate-retrieval quality metrics, e.g. written to ann_index_benchmark.csv

## How to apply

Preprocess query and library spectra identically: remove precursor and low-intensity peaks below 1% of the most intense peak, restrict to the 50 most intense peaks, discard spectra with fewer than 10 peaks or a mass range under 250 Da, and rank-transform intensities. Build the spectral library by removing duplicate peptide ions with SpectraST and appending shuffle-and-reposition decoys in a 1:1 ratio. Run the cascade search: level one identifies unmodified peptides with the dot product under a 5 ppm precursor window, while level two performs the open search over a 500 Da window using the shifted dot product to match modified query spectra to unmodified library counterparts, with candidate selection restricted via an approximate nearest neighbor index (ANN-SoLo baseline: random projection trees, 1000 trees, 200,000 nodes inspected) over unit-normalized 1 Da-binned sparse spectral vectors. Compute FDR in two phases after both cascade levels using the target-decoy strategy, grouping SSMs by charge-normalized precursor mass difference and combining subgroups with fewer than 20 SSMs into a residual group before filtering at the chosen FDR threshold.

## Related tools

- **ANN-SoLo** (Open modification spectral library search engine executing the cascade search, ANN candidate selection, and shifted dot product scoring) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Concatenating spectral libraries while removing duplicate peptide ions before searching)
- **faiss** (Source of alternative ANN index structures (e.g. HNSW, product quantization) to benchmark against the random projection tree baseline) — https://github.com/facebookresearch/faiss
- **Python** (Implementation and scripting language for preprocessing, search orchestration, and benchmark metric reporting (with NumPy, pandas, Pyteomics))

## Evaluation signals

- Candidate-retrieval quality: fraction of true best matches recovered compared against the full random projection tree ensemble baseline (1000 trees, 200,000 nodes inspected)
- Number of confident identifications at the chosen FDR threshold matches or exceeds the ANN-SoLo baseline on HEK293
- Search time on HEK293 is on the order of the ANN-SoLo baseline (under two hours) rather than the SpectraST scale (almost a day)
- FDR is computed in two phases, one after each cascade level, with subgroups of at least 20 SSMs and residual grouping for smaller subgroups
- Target:decoy library ratio is exactly 1:1 via shuffle-and-reposition decoys

## Limitations

- The actual FDR might be underestimated when too-small subgroups are used for FDR estimation
- Accurate FDR estimation for open searches is a standing research question
- ANN indexing is approximate: candidate retrieval can miss true best matches, and suitability of alternative ANN techniques (HNSW, LSH, product quantization) for spectral library searching was left as future work in the paper
- ANN-SoLo requires Python 3.6 to 3.9 (per the README), and no tool versions are stated in the paper

## Evidence

- [methods] cascade search strategy: "To perform an open search ANN-SoLo employs a cascade search strategy consisting of two levels"
- [methods] MassIVE-KB library version: "To search the HEK293 data set the MassIVE-KB peptide spectral library (version 2017/11/27) was used"
- [methods] noise peak filtering: "noise peaks with an intensity below 1 % of the intensity of the most intense peak are removed"
- [methods] decoy generation: "decoy spectra were added in a 1:1 ratio using the shuffle-and-reposition method [41]"
- [methods] FDR subgrouping: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
- [readme] ANN candidate selection: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra"
- [discussion] alternative ANN indexes as future work: "An investigation into whether these techniques are suitable to speed up spectral library searching remains as future work."
