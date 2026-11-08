---
name: adduct-annotation-tallying
description: Decision support for quantifying ion-species bias in LC-MS2 positive-mode
  spectral libraries (e.g. MassBank of North America, GNPS) by tallying adduct annotations
  such as [M+H]+ versus less common ion types.
when_to_use_negative:
- When the goal is to detect or collapse adducts/isotopes within a single LC-MS feature
  table (use Ion Identity Molecular Networking / IIN feature grouping instead).
- When analyzing negative-mode or GC-MS spectral libraries where adduct annotation
  conventions (e.g. [M-H]-) differ and the referenced [M+H]+ bias does not apply.
- When the input is already an aligned feature quantification table with adduct annotations
  per feature — the tally targets library entries, not feature lists.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Source of the GNPS spectral library whose entries are parsed for adduct annotations,
    and host of the GNPS-IIMN-PROPOGATED library that expands ion-species coverage
  repo: https://gnps.ucsd.edu/ProteoSAFe/gnpslibrary.jsp?library=GNPS-IIMN-PROPOGATED
- name: mzmine
  role: Feature-finding tool that performs MS1-based ion identity annotation (adduct
    assignment per feature) upstream of IIMN; its adduct-annotation conventions align
    with the ion types tallied in the library analysis
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:bc10901374f36770b3e5eb93e970f810e03ff427d456e21416735c290083425d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - mzmine
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# adduct-annotation-tallying

## Summary

Quantifies the distribution of ion/adduct annotations across public spectral library entries by parsing each library record's assigned adduct (e.g. [M+H]+, [M+Na]+, [M+NH4]+) and reporting per-library counts and percentage shares, exposing ion-species bias in reference libraries.

## When to use

Decision support for quantifying ion-species bias in LC-MS2 positive-mode spectral libraries (e.g. MassBank of North America, GNPS) by tallying adduct annotations such as [M+H]+ versus less common ion types.

## When NOT to use

- When the goal is to detect or collapse adducts/isotopes within a single LC-MS feature table (use Ion Identity Molecular Networking / IIN feature grouping instead).
- When analyzing negative-mode or GC-MS spectral libraries where adduct annotation conventions (e.g. [M-H]-) differ and the referenced [M+H]+ bias does not apply.
- When the input is already an aligned feature quantification table with adduct annotations per feature — the tally targets library entries, not feature lists.

## Inputs

- Positive-mode LC-MS2 spectral library files from the MassBank of North America (MoNA) spectral library repository
- GNPS spectral library files downloaded from the GNPS website
- Parsed library entries with assigned ion/adduct annotations ([M+H]+, [M+Na]+, [M+NH4]+, [M+K]+, [M-H2O+H]+, etc.)

## Outputs

- ion_identity_tally.csv — summary table with one row per library and ion type, containing absolute counts and percentage shares
- Per-library fraction of [M+H]+ entries (validation target: ~66% MoNA, ~65% GNPS)

## How to apply

Download the target positive-mode LC-MS2 spectral libraries (e.g. MassBank of North America from the MoNA repository and the GNPS spectral library from the GNPS website) in their native library format. Parse each entry and extract its assigned ion/adduct annotation, grouping entries into ion-type classes such as [M+H]+, [M+Na]+, [M+NH4]+, [M+K]+, [M-H2O+H]+, and an 'other' bucket for remaining adducts. Compute absolute entry counts and the fraction of entries per ion type, separately for each library, and assemble a summary table with one row per library and ion type. Validate the tally by comparing the computed [M+H]+ share against the reference expectation — for this analysis, ~66% of MoNA entries and ~65% of GNPS entries were annotated as [M+H]+ — and note that only four other ion types in MoNA exceeded 1000 entries. The rationale is that public libraries are dominated by protonated species, so an adduct-biased library limits the ion-species diversity that workflows such as IIMN can exploit for annotation propagation; the article's IIMN-based spectral libraries were generated to provide 'a broad coverage of ion species'.

## Related tools

- **GNPS** (Source of the GNPS spectral library whose entries are parsed for adduct annotations, and host of the GNPS-IIMN-PROPOGATED library that expands ion-species coverage) — https://gnps.ucsd.edu/ProteoSAFe/gnpslibrary.jsp?library=GNPS-IIMN-PROPOGATED
- **mzmine** (Feature-finding tool that performs MS1-based ion identity annotation (adduct assignment per feature) upstream of IIMN; its adduct-annotation conventions align with the ion types tallied in the library analysis) — https://github.com/mzmine/mzmine

## Examples

```
python tally_adducts.py --library mona_lc_ms2_pos.sdf gnps_library.mgf --output ion_identity_tally.csv
```

## Evaluation signals

- The computed [M+H]+ share reproduces the reported ~66% for the MassBank of North America LC-MS2-positive library and ~65% for the GNPS library.
- In the MoNA tally, only four ion types other than [M+H]+ exceed 1000 entries.
- ion_identity_tally.csv contains one row per (library, ion type) pair, with absolute counts and percentage shares summing to 100% per library.
- Each parsed library entry maps to exactly one ion-type bucket (no entries counted in multiple adduct classes).

## Limitations

- The tally depends on how library entries were annotated; entries lacking an explicit ion/adduct annotation cannot be classified and may bias the computed shares.
- Only positive-mode adducts were enumerated in this analysis; extending to negative-mode libraries requires a different ion-type vocabulary (e.g. [M-H]-, [M+HCOO]-).
- Ion-species coverage figures are a snapshot of the libraries at analysis time; public libraries are updated continuously, so re-running the tally may yield different percentages.

## Evidence

- [results] Two-thirds of the MoNA LC-MS2-positive library are [M+H]+ with only four other ion types above 1000 entries: "Two-thirds of the MassBank of North America LC-MS2-positive ion mode library entries were entered as [M + H]+ while only four other ion types reached more than 1000 entries"
- [abstract] IIMN-based spectral libraries with a broad coverage of ion species are publicly available: "IIMN-based spectral libraries with a broad coverage of ion species are publicly available."
- [intro] IIMN workflow comprises feature grouping, feature shape correlation, and identification of ion species: "The IIMN workflow comprises feature grouping, feature shape correlation, and identification of ion species using a variety of feature-finding software tools"
- [discussion] Repository-hosted GNPS library URL used for library retrieval: "https://gnps.ucsd.edu/ProteoSAFe/gnpslibrary.jsp?library=GNPS-IIMN-PROPOGATED"
