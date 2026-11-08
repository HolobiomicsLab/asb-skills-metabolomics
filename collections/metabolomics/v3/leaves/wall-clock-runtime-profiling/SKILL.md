---
name: wall-clock-runtime-profiling
description: Use when benchmarking the speed of open modification spectral library
  searches across engines, or when verifying that an ANN-indexed search engine reproduces
  the reported ~10x runtime speedup over SpectraST.
when_to_use_negative:
- When the goal is to compare identification sensitivity (number of confident SSMs
  at a given FDR) rather than search speed
- When benchmarking a sequence database search engine (e.g. MSFragger) rather than
  spectral library search engines, since indexing strategies and candidate sets are
  not comparable
- When the two engines are run on different hardware or with materially different
  preprocessing (e.g. peak count or fragment tolerance), making the wall-clock ratio
  meaningless
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ANN-SoLo
  role: Open spectral library search engine whose wall-clock time is measured; uses
    a two-level cascade search and an approximate nearest neighbor index (1000 random-projection
    trees, 200,000 nodes inspected per query)
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Baseline spectral library search engine (version 5.0, TPP 5.1.0) timed with
    identical open-search settings for the runtime comparison; also used for deduplicating
    the spectral library
- name: msconvert
  role: Converts PRIDE PXD001468 raw files to MGF query spectra used as input for
    the timed searches
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:3ff02075a3d61acdc9c21551cf17fc904ad59ffc3f6fc8372017a6d3d4da3408
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - SpectraST
  - msconvert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# wall-clock-runtime-profiling

## Summary

Measures and compares total wall-clock search time of spectral library search engines performing open modification searches (e.g. ANN-SoLo vs SpectraST on the HEK293 data set), to quantify the order-of-magnitude speedup from approximate nearest neighbor indexing.

## When to use

Use when benchmarking the speed of open modification spectral library searches across engines, or when verifying that an ANN-indexed search engine reproduces the reported ~10x runtime speedup over SpectraST.

## When NOT to use

- When the goal is to compare identification sensitivity (number of confident SSMs at a given FDR) rather than search speed
- When benchmarking a sequence database search engine (e.g. MSFragger) rather than spectral library search engines, since indexing strategies and candidate sets are not comparable
- When the two engines are run on different hardware or with materially different preprocessing (e.g. peak count or fragment tolerance), making the wall-clock ratio meaningless

## Inputs

- HEK293 query spectra in MGF format (converted from PRIDE PXD001468 raw files with msconvert)
- MassIVE-KB human HCD spectral library (version 2017/11/27) after duplicate removal and 1:1 shuffled-decoy appending (~3,009,902 spectra)
- ANN-SoLo open-search configuration (cascade windows, ANN index parameters)
- SpectraST open-search configuration (precursor window, fragment tolerance, scoring mode)

## Outputs

- Per-engine total wall-clock search times
- ANN-SoLo/SpectraST runtime ratio (speedup factor)
- Runtime comparison table, e.g. ann_solo_spectrast_runtime_comparison.csv with runtimes, configuration parameters, and speedup factor

## How to apply

Run both engines on the same query spectra (e.g. HEK293 MGF spectra converted from PRIDE PXD001468 raw files with msconvert) against the same library (MassIVE-KB human HCD, deduplicated with SpectraST and 1:1 shuffled decoys via shuffle-and-reposition, ~3,009,902 spectra), after applying identical preprocessing: remove precursor peaks and peaks below 1% of the base peak, keep the 50 most intense peaks, discard spectra with fewer than 10 peaks or mass range under 250 Da, and rank-transform intensities. For ANN-SoLo's two-level cascade open search use a 5 ppm precursor mass window at the first level and 5 ppm followed by 500 Da at the open level, 0.02 Da fragment mass tolerance, and an ANN index of 1000 random-projection trees with 200,000 nodes inspected per query. Run SpectraST 5.0 (Trans-Proteomic Pipeline 5.1.0) with library caching enabled, rank-based scoring, 50 most intense peaks, 0.02 Da fragment tolerance, and a 500 Da precursor mass window for the open search. Record total wall-clock time for each engine and compute the ANN-SoLo/SpectraST runtime ratio; the expected result is a ratio of roughly one order of magnitude (~10x), with ANN-SoLo reducing search time from almost a day to under two hours.

## Related tools

- **ANN-SoLo** (Open spectral library search engine whose wall-clock time is measured; uses a two-level cascade search and an approximate nearest neighbor index (1000 random-projection trees, 200,000 nodes inspected per query)) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Baseline spectral library search engine (version 5.0, TPP 5.1.0) timed with identical open-search settings for the runtime comparison; also used for deduplicating the spectral library)
- **msconvert** (Converts PRIDE PXD001468 raw files to MGF query spectra used as input for the timed searches)

## Evaluation signals

- The computed ANN-SoLo/SpectraST runtime ratio is roughly one order of magnitude (~10x), consistent with the paper's reported speedup
- ANN-SoLo's open search on HEK293 completes in under two hours where SpectraST takes almost a day
- Both engines ran on identical inputs and matched preprocessing (50 most intense peaks, 0.02 Da fragment tolerance), so the timing difference is attributable to the search strategy
- The ANN index was built with the reported settings (1000 random-projection trees, 200,000 nodes inspected per query) before timing
- The output table records per-engine runtimes alongside the full run configuration so the measurement can be reproduced or audited

## Limitations

- Version numbers of the compared search engines are not consistently stated in the article's results section, complicating exact reproduction
- Wall-clock times are hardware-dependent; the order-of-magnitude speedup must be re-validated on the local system, and any material deviation should be reported with the run configuration
- Further speedups via GPU acceleration were not implemented in the published version of ANN-SoLo

## Evidence

- [abstract] Order-of-magnitude speedup over SpectraST on human cell line data set: "ANN-SoLo confidently identifies more spectra than SpectraST or MSFragger and achieves a speedup of an order of magnitude compared with SpectraST."
- [methods] Cascade search strategy with two levels underlies the timed open search: "To perform an open search ANN-SoLo employs a cascade search strategy consisting of two levels"
- [results] SpectraST is the speed baseline in the comparison: "We compared ANN-SoLo to SpectraST, a commonly used spectral library search engine, and MSFragger, a recent sequence database search engine optimized for OMS, in terms of speed"
- [readme] ANN indexing is the mechanism responsible for the runtime reduction: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra"
