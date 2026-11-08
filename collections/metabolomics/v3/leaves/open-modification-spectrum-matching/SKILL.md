---
name: open-modification-spectrum-matching
description: Use when you need to identify modified peptides in high-resolution MS/MS
  spectra via spectral library search, especially for open searches with large precursor
  mass windows such as 500 Da.
when_to_use_negative:
- When the analysis goal is a standard unmodified-peptide search — use ANN-SoLo's
  level-1 plain dot product (5 ppm precursor window) instead of the shifted dot product
  open-search mode.
- When only sequence-database searching is required and no spectral library is available;
  the shifted dot product specifically depends on curated (or predicted) library spectra
  with known modifications.
- When the spectral library is very small — approximate nearest neighbor indexing
  may not be necessary for candidate selection.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0080
tools:
- name: ANN-SoLo
  role: Main search engine implementing the cascade open search with shifted dot product
    scoring and ANN-indexed candidate selection.
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Baseline spectral library search engine used for library deduplication and
    as a comparison for open-search identification counts and runtime.
- name: msconvert
  role: Converts raw mass-spectrometry files (e.g. HEK293 raw data from PXD001468)
    to MGF query spectra.
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:f81d9943b6a1a4caa8348d60c14c0ca9eee75e42c6ff79092e2535c8788266df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Python
  - SpectraST
  - msconvert
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9761904761904762
n_claims: 42
---

# open-modification-spectrum-matching

## Summary

Sensitively match modified query MS/MS spectra to their unmodified spectral-library counterparts using ANN-SoLo's shifted dot product, which compensates for the peak shifts a modification induces in the fragment spectrum.

## When to use

Use when you need to identify modified peptides in high-resolution MS/MS spectra via spectral library search, especially for open searches with large precursor mass windows such as 500 Da.

## When NOT to use

- When the analysis goal is a standard unmodified-peptide search — use ANN-SoLo's level-1 plain dot product (5 ppm precursor window) instead of the shifted dot product open-search mode.
- When only sequence-database searching is required and no spectral library is available; the shifted dot product specifically depends on curated (or predicted) library spectra with known modifications.
- When the spectral library is very small — approximate nearest neighbor indexing may not be necessary for candidate selection.

## Inputs

- Query spectra in MGF format (converted from raw files such as those from PRIDE PXD001468 with msconvert)
- Spectral library in MGF-like spectral library format, e.g. MassIVE-KB version 2017/11/27 or the NIST human HCD library (version 2016/09/12), with 1:1 shuffled decoys appended
- Precursor mass tolerance settings (e.g. 5 ppm standard, 500 Da open window)
- Fragment mass tolerance (0.02 Da)
- FDR threshold (1%)

## Outputs

- Spectrum-spectrum matches (SSMs) with shifted dot product scores for modified peptides
- List of confident identifications at 1% FDR, exportable as e.g. ann_solo_hek293_idents.csv
- FDR estimates per subgroup of SSMs
- Total and per-query runtime statistics

## How to apply

Preprocess both query and library spectra identically: remove precursor peaks, drop noise peaks below 1% of the base peak, keep the 50 most intense peaks, discard spectra with fewer than 10 remaining peaks or a mass range under 250 Da, and rank-transform intensities. For the open (modified) level of ANN-SoLo's cascade search, use the shifted dot product so fragment peaks can align after a constant mass shift, with a fragment mass tolerance of 0.02 Da; apply a precursor mass window such as 5 ppm followed by 500 Da for the two levels. Candidate selection is accelerated by an approximate nearest neighbor index built from 1000 random-projection trees, inspecting 200 000 nodes per query, using the rank-transformed dot product similarity in level 1. Estimate FDR with the target–decoy strategy (decoy over target SSMs), grouping SSMs by charge-normalized precursor mass difference into 0.1 Da subgroups with a minimum of 20 SSMs per subgroup, and filter at 1% FDR. Confident SSMs from both cascade levels are combined to produce the final identification set.

## Related tools

- **ANN-SoLo** (Main search engine implementing the cascade open search with shifted dot product scoring and ANN-indexed candidate selection.) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Baseline spectral library search engine used for library deduplication and as a comparison for open-search identification counts and runtime.)
- **msconvert** (Converts raw mass-spectrometry files (e.g. HEK293 raw data from PXD001468) to MGF query spectra.)

## Examples

```
python ann_solo/search.py --library massivekb.mgf --spectra hek293.mgf --precursor_tolerance 500 --fragment_tolerance 0.02 --fdr 0.01
```

## Evaluation signals

- At 1% subgroup FDR, the number of confident SSMs on HEK293 should exceed those reported for SpectraST and MSFragger open searches.
- The runtime ratio ANN-SoLo/SpectraST on the HEK293 open search should be on the order of 10x, reducing SpectraST's ~day-long search to under two hours.
- On iPRG2012, open search should yield roughly a 25% increase in identifications versus the standard search, with the extra hits corresponding to modified peptides.
- SSMs must pass subgroup FDR filtering: only subgroups containing at least 20 SSMs contribute accepted identifications.
- Appended decoy spectra must be 1:1 relative to targets using the shuffle-and-reposition method, and reported FDR must equal decoy SSM count divided by target SSM count.

## Limitations

- FDR may be underestimated when subgroups contain too few SSMs, and accurate FDR estimation for open searches remains an unresolved research question.
- GPU acceleration and non-metric space indexing for direct shifted-dot-product candidate retrieval are not implemented in the described version.
- Suitability of alternative ANN techniques (HNSW, LSH, product quantization) for spectral library searching was not investigated.
- Applying the 50 most-intense-peak and 10-peak-minimum preprocessing filters can discard spectra with too few informative peaks.

## Evidence

- [intro] the shifted dot product is used to accurately match modified query spectra to their unmodified library counterpart by taking into account peak shifts caused by a modification: "the shifted dot product is used to accurately match modified query spectra to their unmodified library counterpart by taking into account peak shifts caused by a modification"
- [methods] they are grouped based on their charge-normalized precursor mass difference: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
- [readme] ANN-SoLo requires Python 3.6 to 3.9: "ANN-SoLo requires Python 3.6 to 3.9 (Python 3.10 and newer"
- [methods] decoy spectra were added in a 1:1 ratio using the shuffle-and-reposition method: "decoy spectra were added in a 1:1 ratio using the shuffle-and-reposition method [41]"
- [methods] noise peaks with an intensity below 1 % of the intensity of the most intense peak are removed: "noise peaks with an intensity below 1 % of the intensity of the most intense peak are removed"
