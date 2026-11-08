---
name: cascade-search-configuration
description: Use when performing open modification spectral library searches where
  unmodified and modified peptides must be identified separately with strict FDR control,
  e.g. searching query spectra against a large target-decoy spectral library with
  ANN-SoLo.
when_to_use_negative:
- When only unmodified peptide identifications are needed, a standard closed spectral
  library search without the open-search cascade level suffices
- When the spectral library is small enough that exhaustive open searching is computationally
  feasible, the ANN-indexed candidate selection step of the cascade is unnecessary
- When accurate FDR estimation for very small mass-difference subgroups is critical
  — the article cautions that FDR may be underestimated when too-small subgroups are
  used
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ANN-SoLo
  role: Runs the two-level cascade open search with ANN-indexed candidate selection,
    shifted dot product scoring, and two-phase target-decoy FDR calculation
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Used to concatenate the human HCD and yeast spectral libraries while removing
    duplicates before the cascade search
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:5d4f86660267430e8c61eed428a92a8ad06b5da0abe46e747b9f6bccdcafef4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Python
  - SpectraST
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# cascade-search-configuration

## Summary

Configure and run ANN-SoLo's two-level cascade open search: first identify unmodified peptides within a narrow precursor mass window, then open-search remaining spectra for modified peptides via ANN-indexed candidate selection and the shifted dot product, with two-phase target–decoy FDR control.

## When to use

Use when performing open modification spectral library searches where unmodified and modified peptides must be identified separately with strict FDR control, e.g. searching query spectra against a large target-decoy spectral library with ANN-SoLo.

## When NOT to use

- When only unmodified peptide identifications are needed, a standard closed spectral library search without the open-search cascade level suffices
- When the spectral library is small enough that exhaustive open searching is computationally feasible, the ANN-indexed candidate selection step of the cascade is unnecessary
- When accurate FDR estimation for very small mass-difference subgroups is critical — the article cautions that FDR may be underestimated when too-small subgroups are used

## Inputs

- Query spectra from the iPRG2012 data set (MassIVE accession MSV000078492)
- Concatenated NIST human HCD spectral library (version 2016/09/12) and TripleTOF yeast spectral library (Selevsek et al.) as a target-decoy library with decoys added 1:1 via shuffle-and-reposition

## Outputs

- Spectrum-spectrum matches (SSMs) filtered at the target FDR threshold, split between unmodified (level 1) and modified (level 2) identifications
- Per-SSM shifted dot product scores and charge-normalized precursor mass differences used for subgrouping

## How to apply

Preprocess query and library spectra identically (remove peaks below 1% of the most intense peak, keep the 50 most intense peaks, discard spectra with fewer than 10 peaks or a mass range below 250 Da). Run level 1 as a standard search using a narrow precursor mass window (e.g. 20 ppm for iPRG2012 high-resolution data) to identify unmodified peptides. Run level 2 as an open search on the remaining spectra using an ANN index (ensembles of random projection trees via Annoy) to select candidate library spectra, scoring candidates with the shifted dot product to match modified query spectra to unmodified library counterparts; the article's iPRG2012 setup used a 300 Da mass-difference window for the open-search level and 0.02 Da fragment mass tolerance. Decoy spectra are added in a 1:1 ratio using the shuffle-and-reposition method, and FDR is calculated in two phases after both cascade levels using the target–decoy strategy, with SSMs subgrouped by charge-normalized precursor mass difference and FDR filtering applied only to subgroups containing at least 20 SSMs.

## Related tools

- **ANN-SoLo** (Runs the two-level cascade open search with ANN-indexed candidate selection, shifted dot product scoring, and two-phase target-decoy FDR calculation) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Used to concatenate the human HCD and yeast spectral libraries while removing duplicates before the cascade search)

## Examples

```
ann_solo --query spectra.mgf --library concatenated_target_decoy.sptxt --precursor_tolerance 20 ppm --open_mods --mass_diff 300 Da --fragment_tolerance 0.02 Da
```

## Evaluation signals

- The open (cascade) search on iPRG2012 yields ~25% more identifications than the standard search, with the additional identifications corresponding to modified peptides
- Confident SSMs pass the two-phase target-decoy FDR threshold, with FDR filtering applied only to subgroups containing at least 20 SSMs
- Level 1 identifications fall within the narrow precursor mass window (e.g. 20 ppm) and level 2 identifications carry large charge-normalized precursor mass shifts (within the 300 Da open-search window)
- Decoy matches are present at roughly 1:1 ratio with targets via the shuffle-and-reposition method, allowing FDR estimation after both cascade levels

## Limitations

- The actual FDR might be underestimated when too-small subgroups are used for subgroup-wise FDR filtering
- Whether alternative approaches are needed for accurate FDR estimation of open searches is a standing research question
- FDR filtering is only done for subgroups that contain at least 20 SSMs, so spectra in smaller subgroups are not FDR-controlled the same way

## Evidence

- [intro] Cascade search strategy: "Using a cascade search strategy [38] ANN-SoLo first identifies any unmodified peptides, followed by an open search to identify the modified peptides."
- [methods] Two-phase FDR: "False discovery rates are calculated in two phases after both levels of the cascade search using the target–decoy strategy"
- [methods] Subgroup FDR: "FDR filtering is only done for subgroups that contain at least 20 SSMs"
- [results] Open search gain: "Based on the iPRG2012 data set ANN-SoLo achieves a 25 % increase in identifications when performing an open search compared to a standard search"
- [discussion] Subgroup caution: "Caution has to be observed though as the actual FDR might be underestimated when too small groups are used [27]."
