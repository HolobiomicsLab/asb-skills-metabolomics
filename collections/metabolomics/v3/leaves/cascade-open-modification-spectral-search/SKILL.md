---
name: cascade-open-modification-spectral-search
description: Use when searching high-resolution query spectra (e.g. iPRG2012 TripleTOF
  mzML) against a large target-decoy spectral library and you need to confidently
  identify both unmodified and modified peptides via open modification searching at
  1% FDR.
when_to_use_negative:
- Query spectra are low-resolution or the goal is a closed/unmodified-only search
  — a standard narrow-window library search suffices, since the cascade's open level
  targets modified peptides
- No suitable target spectral library exists for the sample's organism/instrument
  — the skill requires library spectra (library-based, not sequence database search
  like MSFragger)
- Only exact (unmodified) matches are of interest and the ANN index/shifted dot product
  overhead is unnecessary
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: ANN-SoLo
  role: Primary search engine implementing the two-level cascade search, ANN candidate
    selection (random projection trees), shifted dot product scoring, and subgroup
    target-decoy FDR estimation
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Used to concatenate the NIST human HCD and Selevsek yeast libraries while
    removing duplicates (keeping best replicate spectrum per peptide ion)
- name: Faiss
  role: Reference approximate nearest neighbor search library relevant to the ANN
    indexing approach used for level-2 candidate selection
  repo: https://github.com/facebookresearch/faiss
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:97a002334665465855ad8d62b9418e60c2260a6023002d949ffb3c81e2f56b84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Faiss
  - Python
  - SpectraST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# cascade-open-modification-spectral-search

## Summary

Two-level cascade spectral library search (ANN-SoLo) that first identifies unmodified peptides in a narrow 20 ppm precursor window with the standard dot product, then runs an open search in a 300 Da window using approximate nearest neighbor indexing and the shifted dot product to identify modified peptides at controlled FDR.

## When to use

Use when searching high-resolution query spectra (e.g. iPRG2012 TripleTOF mzML) against a large target-decoy spectral library and you need to confidently identify both unmodified and modified peptides via open modification searching at 1% FDR.

## When NOT to use

- Query spectra are low-resolution or the goal is a closed/unmodified-only search — a standard narrow-window library search suffices, since the cascade's open level targets modified peptides
- No suitable target spectral library exists for the sample's organism/instrument — the skill requires library spectra (library-based, not sequence database search like MSFragger)
- Only exact (unmodified) matches are of interest and the ANN index/shifted dot product overhead is unnecessary

## Inputs

- High-resolution query spectra (e.g. iPRG2012 TripleTOF spectra from MassIVE MSV000078492)
- Target spectral library (e.g. NIST human HCD library version 2016/09/12 concatenated with Selevsek et al. TripleTOF yeast library via SpectraST, best replicate per peptide ion)
- Decoy spectral library (1:1 target:decoy ratio, shuffle-and-reposition method)
- Search parameters: 20 ppm level-1 precursor window, 300 Da level-2 window, 0.02 Da fragment tolerance, 1% FDR threshold

## Outputs

- SSM (spectrum-spectrum match) list at 1% FDR from both cascade levels, with precursor mass differences for level-2 hits
- Count of confidently identified spectra per search mode (e.g. iprg2012_open_search_identifications.csv)
- Open vs standard search identification comparison (ANN-SoLo reported ~25% increase on iPRG2012, additional hits corresponding to modified peptides)

## How to apply

Build a concatenated target-decoy spectral library (e.g. NIST human HCD v2016/09/12 plus Selevsek TripleTOF yeast, decoys at 1:1 via shuffle-and-reposition, ~1,188,168 spectra). Preprocess query and library spectra identically: remove precursor and noise peaks below 1% of the base peak, keep at most the 50 most intense peaks, discard spectra with fewer than 10 peaks or mass range below 250 Da, and rank-transform intensities. Run the ANN-SoLo cascade: level 1 is a standard search with a 20 ppm precursor mass window and dot product scoring; level 2 is an open search with a 300 Da window where approximate nearest neighbor candidate selection (ensemble of random projection trees, 100-1000 trees, 20,000-400,000 nodes inspected per query) is followed by shifted dot product scoring, using 0.02 Da fragment mass tolerance throughout. Estimate FDR with the target-decoy strategy in two phases, grouping level-2 SSMs by charge-normalized precursor mass difference in 0.1 Da bins and pooling subgroups with fewer than 20 SSMs into a residual group. Retain SSMs at 1% FDR from both cascade levels and count confidently identified spectra.

## Related tools

- **ANN-SoLo** (Primary search engine implementing the two-level cascade search, ANN candidate selection (random projection trees), shifted dot product scoring, and subgroup target-decoy FDR estimation) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Used to concatenate the NIST human HCD and Selevsek yeast libraries while removing duplicates (keeping best replicate spectrum per peptide ion))
- **Faiss** (Reference approximate nearest neighbor search library relevant to the ANN indexing approach used for level-2 candidate selection) — https://github.com/facebookresearch/faiss

## Evaluation signals

- Open search on iPRG2012 should reproduce the reported ~25% increase in identifications over a standard 20 ppm search, with additional identifications corresponding to modified peptides
- Every retained SSM must pass the 1% FDR threshold via the target-decoy strategy applied after both cascade levels
- Subgroup FDR structure: level-2 SSMs grouped by charge-normalized precursor mass difference in 0.1 Da ranges; subgroups with fewer than 20 SSMs pooled into a residual group (over-small groups can underestimate FDR)
- Library construction check: decoys added at 1:1 ratio via shuffle-and-reposition, yielding ~1,188,168 library spectra for the iPRG2012 setup
- Preprocessing invariants: no precursor peaks, no peaks below 1% of base peak, at most 50 peaks per spectrum, spectra with <10 peaks or <250 Da mass range discarded, intensities rank-transformed

## Limitations

- Accurate FDR estimation for open searches remains an open research question; actual FDR may be underestimated when subgroups are too small
- Alternative ANN indexing techniques (HNSW, LSH, product quantization) and direct non-metric-space retrieval of shifted dot product candidates were not investigated in the source work
- GPU acceleration for candidate selection and SSM evaluation is not implemented in this version of ANN-SoLo; ANN-SoLo requires Python 3.6-3.9
- Approximate candidate selection trades a limited candidate set for speed, so very distant modifications depend on the ANN index recall and the shifted dot product's ability to align shifted peaks

## Evidence

- [intro] cascade first unmodified then open: "Using a cascade search strategy [38] ANN-SoLo first identifies any unmodified peptides, followed by an open search to identify the modified peptides."
- [readme] ANN indexing purpose: "uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra"
- [methods] FDR procedure: "False discovery rates are calculated in two phases after both levels of the cascade search using the target–decoy strategy"
- [methods] subgroup FDR grouping: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
- [results] expected result on iPRG2012: "ANN-SoLo achieves a 25 % increase in identifications when performing an open search compared to a standard search (table 1), with the additional identifications corresponding to modified peptides."
- [methods] noise filtering threshold: "noise peaks with an intensity below 1 % of the intensity of the most intense peak are removed"
