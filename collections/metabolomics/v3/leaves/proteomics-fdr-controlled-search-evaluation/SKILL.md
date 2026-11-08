---
name: proteomics-fdr-controlled-search-evaluation
description: Use when converting spectrum-spectrum matches from an open modification
  spectral library search into confident identifications, e.g. estimating FDR via
  target–decoy decoys appended 1:1 and grouping SSMs by charge-normalized precursor
  mass difference before filtering.
when_to_use_negative:
- The search was run without decoy spectra, so no target–decoy FDR estimate can be
  formed
- Every subgroup of SSMs (grouped by charge-normalized precursor mass difference)
  contains fewer than 20 SSMs, so per-subgroup FDR cannot be reliably estimated
- The goal is only to benchmark wall-clock search time (e.g. ANN-SoLo vs SpectraST
  runtime), not to assess identification confidence
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: ANN-SoLo
  role: Runs the two-level cascade open search and performs the two-phase target–decoy
    FDR calculation with subgroup filtering of SSMs
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Used to remove duplicate peptide ions when preparing the target spectral library
    before decoy appending
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:fc9e9858165943372850562b67f2f3d3c8044b941f42326337d56a5ae64d0a6f
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# proteomics-fdr-controlled-search-evaluation

## Summary

Estimates and controls false discovery rate for open modification spectral library search results using a target–decoy strategy, including the two-phase FDR calculation after both cascade levels and subgroup-wise FDR estimation by charge-normalized precursor mass difference. It is how ANN-SoLo-style open searches turn raw spectrum-spectrum matches into confident identifications.

## When to use

Use when converting spectrum-spectrum matches from an open modification spectral library search into confident identifications, e.g. estimating FDR via target–decoy decoys appended 1:1 and grouping SSMs by charge-normalized precursor mass difference before filtering.

## When NOT to use

- The search was run without decoy spectra, so no target–decoy FDR estimate can be formed
- Every subgroup of SSMs (grouped by charge-normalized precursor mass difference) contains fewer than 20 SSMs, so per-subgroup FDR cannot be reliably estimated
- The goal is only to benchmark wall-clock search time (e.g. ANN-SoLo vs SpectraST runtime), not to assess identification confidence

## Inputs

- Query spectra in MGF format (e.g. HEK293 spectra converted from PRIDE PXD001468 raw files with msconvert)
- Target spectral library with appended shuffled decoys in a 1:1 ratio (shuffle-and-reposition method)
- SSM results from both levels of a cascade search (standard + open modification)

## Outputs

- Confident spectrum-spectrum matches filtered at the FDR threshold, with modified peptide identifications
- Per-subgroup FDR estimates grouped by charge-normalized precursor mass difference
- Comparison table of identifications (e.g. open vs standard search counts, per-engine results)

## How to apply

Prepare the search library by appending decoy spectra in a 1:1 ratio with the target spectra using the shuffle-and-reposition method (e.g. 3,009,902 total spectra for the MassIVE-KB human HCD library on the HEK293 data set). Run the cascade search (standard level first, then open modification level, e.g. 5 ppm followed by 500 Da precursor windows) and compute FDR in two phases after both levels of the cascade search using the target–decoy strategy. For the open search, group SSMs based on their charge-normalized precursor mass difference between the query spectra and library spectra, since different mass shifts carry different modification populations. Estimate FDR within each subgroup, applying FDR filtering only for subgroups that contain at least 20 SSMs, then retain confident SSMs below the FDR threshold. Report identified (including modified) peptides and check identification gains, e.g. the 25% increase from open vs standard search on iPRG2012.

## Related tools

- **ANN-SoLo** (Runs the two-level cascade open search and performs the two-phase target–decoy FDR calculation with subgroup filtering of SSMs) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Used to remove duplicate peptide ions when preparing the target spectral library before decoy appending)

## Evaluation signals

- Decoys were appended in a 1:1 ratio with targets using the shuffle-and-reposition method before searching
- FDR was computed in two phases, one after each level of the cascade search, not only once at the end
- Subgroups with fewer than 20 SSMs were excluded from FDR filtering
- Open search yields more identifications than standard search (e.g. ~25% increase on iPRG2012), with additional IDs corresponding to modified peptides
- Final SSM list contains only matches below the chosen FDR threshold

## Limitations

- The actual FDR might be underestimated when too-small subgroups are used
- Whether alternative approaches are needed for accurate FDR estimation of open searches remains a standing research question

## Evidence

- [methods] Two-phase cascade FDR: "False discovery rates are calculated in two phases after both levels of the cascade search using the target–decoy strategy"
- [methods] Decoy construction: "decoy spectra were added in a 1:1 ratio using the shuffle-and-reposition method [41]"
- [methods] Subgroup FDR: "FDR filtering is only done for subgroups that contain at least 20 SSMs"
- [discussion] Small subgroup caveat: "Caution has to be observed though as the actual FDR might be underestimated when too small groups are used [27]."
- [results] Open vs standard gain: "ANN-SoLo achieves a 25 % increase in identifications when performing an open search compared to a standard search"
