---
name: ann-index-configuration-tuning
description: Use when configuring ANN-SoLo's ANN index (ensemble of random projection
  trees, nodes inspected per query) for an open modification spectral library search,
  to trade identification counts toward the full-search baseline against search speed.
when_to_use_negative:
- The search is a standard (closed) search of unmodified peptides — ANN-SoLo's first
  cascade level with a narrow precursor mass filter handles this without the open-level
  ANN tuning sweep
- Exact nearest-neighbor results are required (e.g. establishing the full-search baseline
  itself); ANN indexing only approximates the true nearest neighbors
- The spectral library is small enough that exhaustive candidate scoring is tractable,
  so the approximate index provides no speed advantage
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: ANN-SoLo
  role: Spectral library search engine whose ANN index (Annoy random projection trees)
    is swept over tree count and nodes inspected per query
  repo: https://github.com/bittremieux/ANN-SoLo
- name: Python
  role: Language used to orchestrate the (trees, nodes) sweep and record identifications
    and wall-clock runtimes
- name: SpectraST
  role: Used to remove duplicate peptide ions from the spectral library before concatenating
    targets and decoys; also serves as the full-search runtime baseline
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:4860fbe678e323055119808ef9194462b9a78be5f1e3a77fba2d65e2d281769a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Python
  - SpectraST
  - msconvert
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# ann-index-configuration-tuning

## Summary

Sweep and select the hyperparameters of ANN-SoLo's approximate nearest neighbor index — the number of random projection trees and tree nodes inspected per query — to balance open-modification spectral library search identification sensitivity against wall-clock runtime.

## When to use

Use when configuring ANN-SoLo's ANN index (ensemble of random projection trees, nodes inspected per query) for an open modification spectral library search, to trade identification counts toward the full-search baseline against search speed.

## When NOT to use

- The search is a standard (closed) search of unmodified peptides — ANN-SoLo's first cascade level with a narrow precursor mass filter handles this without the open-level ANN tuning sweep
- Exact nearest-neighbor results are required (e.g. establishing the full-search baseline itself); ANN indexing only approximates the true nearest neighbors
- The spectral library is small enough that exhaustive candidate scoring is tractable, so the approximate index provides no speed advantage

## Inputs

- Query spectra in MGF format (e.g. iPRG2012, MassIVE accession MSV000078492)
- Concatenated target-decoy spectral library (.sptxt), e.g. NIST human HCD (version 2016/09/12) plus TripleTOF yeast library from Selevsek et al., duplicates removed with SpectraST and shuffle-and-reposition decoys appended 1:1
- ANN-SoLo (Python/C++, Apache 2.0) from github.com/bittremieux/ANN-SoLo

## Outputs

- Hyperparameter sweep table ann_hyperparameter_sweep.csv with columns (trees, nodes, identifications, search time)
- Per-configuration confident SSM counts at subgroup target-decoy FDR threshold
- Wall-clock runtime per (trees, nodes) setting for speedup comparison

## How to apply

Build ANN indices as ensembles of random projection trees (Annoy) over the target-decoy spectral library, sweeping the number of trees across roughly 100–1000 and, for each index, the number of tree nodes inspected per query across roughly 20,000–400,000. Run ANN-SoLo's two-level cascade open search per configuration — e.g. on iPRG2012 (MSV000078492) against the concatenated NIST human HCD / TripleTOF yeast target-decoy library of 1,188,168 spectra with a 20 ppm first-level precursor mass window, 300 Da open-search level window, and 0.02 Da fragment mass tolerance — and filter spectrum-spectrum matches with the target-decoy subgroup FDR strategy (filtering only subgroups with at least 20 SSMs). Record confident identifications and wall-clock search time for every (trees, nodes) pair and write the sweep table to ann_hyperparameter_sweep.csv. Increasing trees and inspected nodes yields a more faithful approximation of the true nearest neighbors, raising identification counts toward the full-search baseline at the cost of increased runtime; the paper's HEK293 runtime reproduction uses 1000 trees with 200,000 nodes inspected per query, which delivers an order-of-magnitude speedup over SpectraST. Validate that the sweep exhibits this accuracy-speed trade-off; if the empirical ratio deviates materially, report the measured value with its run configuration for expert review.

## Related tools

- **ANN-SoLo** (Spectral library search engine whose ANN index (Annoy random projection trees) is swept over tree count and nodes inspected per query) — https://github.com/bittremieux/ANN-SoLo
- **Python** (Language used to orchestrate the (trees, nodes) sweep and record identifications and wall-clock runtimes)
- **SpectraST** (Used to remove duplicate peptide ions from the spectral library before concatenating targets and decoys; also serves as the full-search runtime baseline)

## Examples

```
python ann_solo.py -q iprg2012.mgf -l library_targetdecoy.sptxt -f 0.02 -o ann_hits.csv --mode open --precursor_tolerance_level1 20ppm --ann_trees 1000 --ann_nodes 200000
```

## Evaluation signals

- The sweep table must show identification counts rising monotonically toward the full-search baseline as trees (100→1000) and inspected nodes (20,000→400,000) increase
- Wall-clock search time must increase monotonically with both trees and nodes inspected, demonstrating the reported accuracy-speed trade-off
- With the paper's high-fidelity HEK293 configuration (1000 trees, 200,000 nodes inspected per query, 5 ppm precursor windows, 0.02 Da fragment tolerance), the ANN-SoLo/SpectraST runtime ratio should be roughly one order of magnitude (~10x)
- FDR filtering must be applied per subgroup (charge-normalized precursor mass difference grouping) and only for subgroups containing at least 20 SSMs
- All library and query spectra should survive the preprocessing invariants: ≥10 peaks, mass range ≥250 Da, top-50 most intense peaks retained, noise below 1% of base peak removed

## Limitations

- Fewer trees or inspected nodes produce a less faithful approximation of the true nearest neighbors, reducing identifications relative to the full search
- The actual FDR might be underestimated when too-small subgroups are used; subgrouping-based FDR estimation for open searches remains an unresolved research question
- This ANN-SoLo version uses Annoy random projection trees only — suitability of HNSW, LSH, product quantization, and GPU acceleration for candidate selection was left as future work

## Evidence

- [intro] During this open search ANN-SoLo uses an approximate nearest neighbor (ANN) index to efficiently find a limited set of the most similar library spectra for each query spectrum.: "During this open search ANN-SoLo uses an approximate nearest neighbor (ANN) index to efficiently find a limited set of the most similar library spectra for each query spectrum."
- [readme] ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra"
- [methods] The resulting SSMs are filtered on FDR, and the confident SSMs below the FDR threshold are retained: "The resulting SSMs are filtered on FDR, and the confident SSMs below the FDR threshold are retained"
- [discussion] Caution has to be observed though as the actual FDR might be underestimated when too small groups are used: "Caution has to be observed though as the actual FDR might be underestimated when too small groups are used"
- [discussion] An investigation into whether these techniques are suitable to speed up spectral library searching remains as future work.: "An investigation into whether these techniques are suitable to speed up spectral library searching remains as future work."
