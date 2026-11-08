---
name: spectral-library-entry-parsing
description: Use when a library of MS/MS spectra entries (e.g., MoNA, GNPS LC-MS2
  positive mode) needs each entry's assigned adduct/ion annotation extracted and counted
  to quantify ion-species bias or coverage.
when_to_use_negative:
- Input is already a tally or aggregated adduct-distribution table rather than raw
  spectral library entries
- The task is spectral matching or network propagation (use GNPS library search or
  FBMN/IIMN workflows) rather than counting adduct annotations across library entries
- Libraries are in negative ion mode or untargeted formats lacking per-entry ion/adduct
  annotations
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Source of the GNPS spectral library and web platform where parsed entries
    and propagated libraries (GNPS-IIMN-PROPOGATED) are hosted and searched
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: MZmine
  role: Feature-finding tool whose IIMN outputs supply extracted consensus MS2 spectra
    that become new library entries whose ion-species distribution can be tallied
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:fe14e7bdf00db983ce7a8ab3f69509fcb67a51eb8c61ee8ccd2efe2953f53631
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - MZmine
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# spectral-library-entry-parsing

## Summary

Parses MS/MS spectral library entries (e.g., MassBank of North America and GNPS LC-MS2 positive-mode libraries) to extract per-entry ion/adduct annotations and tally the distribution of ion species such as [M+H]+, [M+Na]+, [M+NH4]+, and [M+K]+. This quantifies ion-species bias in public reference libraries, a key motivation for the IIMN-based propagated spectral libraries.

## When to use

Use when a library of MS/MS spectra entries (e.g., MoNA, GNPS LC-MS2 positive mode) needs each entry's assigned adduct/ion annotation extracted and counted to quantify ion-species bias or coverage.

## When NOT to use

- Input is already a tally or aggregated adduct-distribution table rather than raw spectral library entries
- The task is spectral matching or network propagation (use GNPS library search or FBMN/IIMN workflows) rather than counting adduct annotations across library entries
- Libraries are in negative ion mode or untargeted formats lacking per-entry ion/adduct annotations

## Inputs

- MassBank of North America LC-MS2 positive-mode spectral library (from the MoNA spectral library repository)
- GNPS LC-MS2 positive-mode spectral library (from the GNPS website)
- Optionally, IIMN-propagated library entries such as GNPS-IIMN-PROPOGATED for comparison of ion-species coverage

## Outputs

- ion_identity_tally.csv — summary table with one row per library and ion type, containing absolute counts and percentage shares
- Per-library fraction of [M+H]+ entries and counts of remaining adduct classes

## How to apply

Download the target spectral libraries (e.g., the MassBank of North America LC-MS2 positive-mode library from the MoNA repository and the GNPS spectral library). Parse each library entry and extract its assigned ion/adduct annotation, grouping entries into ion-type classes such as [M+H]+, [M+Na]+, [M+NH4]+, [M+K]+, [M-H2O+H]+, and other adducts. Compute the fraction of entries per ion type for each library separately, then assemble a summary table (one row per library × ion type, with absolute counts and percentage shares) saved as ion_identity_tally.csv. This tally motivates the IIMN approach: the article found that public libraries are dominated by [M+H]+ entries while IIMN propagation broadens coverage across adducts and in-source fragments. Validate by checking that computed [M+H]+ shares reproduce reported values (e.g., 66% for MoNA and 65% for GNPS).

## Related tools

- **GNPS** (Source of the GNPS spectral library and web platform where parsed entries and propagated libraries (GNPS-IIMN-PROPOGATED) are hosted and searched) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **MZmine** (Feature-finding tool whose IIMN outputs supply extracted consensus MS2 spectra that become new library entries whose ion-species distribution can be tallied) — https://github.com/mzmine/mzmine

## Evaluation signals

- Computed [M+H]+ share reproduces the reported ~66% for the MoNA LC-MS2 positive-mode library
- Computed [M+H]+ share reproduces the reported ~65% for the GNPS spectral library
- Only four other ion types exceed 1000 entries in the MoNA positive-mode library, consistent with the reported finding
- ion_identity_tally.csv has one row per library × ion type with both absolute counts and percentage shares, and the percentage column sums to 100% per library
- For IIMN-propagated libraries, [M+H]+ share drops to ~24% with multimeric species ~22%, [M+Na]+ ~17%, in-source fragments ~15%, and [M+NH4]+ ~13%, confirming broadened ion-species coverage

## Limitations

- Public reference libraries are heavily biased toward [M+H]+, so tallies mostly confirm a known imbalance rather than resolving it; broadened coverage requires IIMN propagation
- Reproducibility depends on library snapshots: the article notes specific versions/parameters for the 24 datasets were only summarized in Supplementary Table 4 and are not pinned in the text
- Library entries with missing or ambiguous ion/adduct annotations must be grouped into an 'other adducts' class; the IIN edge format is generic CSV with columns ID1 and ID2 and has no formal schema or validator tool

## Evidence

- [results] Two-thirds of the MassBank of North America LC-MS2-positive ion mode library entries were entered as [M + H]+: "Two-thirds of the MassBank of North America LC-MS2-positive ion mode library entries were entered as [M + H]+ while only four other ion types reached more than 1000 entries"
- [results] Propagation yielded IIMN spectral libraries with broadened ion species coverage: [evidence span withheld — anchor status: not_found]
- [abstract] IIMN-based spectral libraries with a broad coverage of ion species are publicly available: "IIMN-based spectral libraries with a broad coverage of ion species are publicly available."
- [supplementary] The text format follows a generic comma-separated style: "The text format follows a generic comma-separated style with the columns ID1 and ID2"
