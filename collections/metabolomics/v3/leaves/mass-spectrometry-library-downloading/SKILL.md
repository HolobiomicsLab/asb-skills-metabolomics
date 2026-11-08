---
name: mass-spectrometry-library-downloading
description: Use when you need to quantify the adduct/ion-species composition of public
  LC-MS2 spectral libraries (MoNA, GNPS) or need those raw library entries as inputs
  for an ion-species bias analysis.
when_to_use_negative:
- The goal is MS/MS-based molecular networking or library search on sample data (e.g.
  Feature-Based Molecular Networking), not a survey of library composition — use FBMN/library-search
  workflows instead.
- The libraries of interest are already IIMN-derived with propagated annotations across
  ion species, so a bias audit of unconnected adducts is not the question.
- The data are negative-mode or GC-MS library entries — the reference analysis covered
  LC-MS2 positive-mode libraries only.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3334
tools:
- name: GNPS
  role: Public spectral library hosting and distribution — GNPS library endpoints
    (e.g. GNPS-IIMN-PROPOGATED) serve the reference spectra downloaded and tallied
    in this skill.
  repo: http://gnps.ucsd.edu
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:b967ce06f26a9ceebe69685311e71bc9702afa67e8b187d3963bae7152859363
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# mass-spectrometry-library-downloading

## Summary

Retrieves and parses public LC-MS2 spectral reference libraries (MassBank of North America, GNPS) to quantify their ion/adduct-species composition, exposing the [M + H]+ bias in reference libraries that the IIMN workflow aims to correct by propagating annotations across ion species.

## When to use

Use when you need to quantify the adduct/ion-species composition of public LC-MS2 spectral libraries (MoNA, GNPS) or need those raw library entries as inputs for an ion-species bias analysis.

## When NOT to use

- The goal is MS/MS-based molecular networking or library search on sample data (e.g. Feature-Based Molecular Networking), not a survey of library composition — use FBMN/library-search workflows instead.
- The libraries of interest are already IIMN-derived with propagated annotations across ion species, so a bias audit of unconnected adducts is not the question.
- The data are negative-mode or GC-MS library entries — the reference analysis covered LC-MS2 positive-mode libraries only.

## Inputs

- MassBank of North America LC-MS2 positive-mode spectral library (downloaded from the MoNA spectral library repository)
- GNPS positive-mode LC-MS2 spectral library (downloaded from the GNPS website, e.g. GNPS-IIMN-PROPOGATED)
- Library entries with assigned ion/adduct annotations

## Outputs

- ion_identity_tally.csv — summary table with one row per library and ion type, containing absolute entry counts and percentage shares
- Per-library fraction of entries annotated as [M+H]+ and counts of the remaining adduct classes ([M+Na]+, [M+NH4]+, [M+K]+, [M-H2O+H]+, other)

## How to apply

Download the MassBank of North America LC-MS2 positive-mode library from the MoNA spectral library repository and the GNPS spectral library from the GNPS website (libraries such as GNPS-IIMN-PROPOGATED are hosted on the GNPS library server). Parse each library entry and extract its assigned ion/adduct annotation, binning entries into [M+H]+, [M+Na]+, [M+NH4]+, [M+K]+, [M-H2O+H]+ and an 'other adducts' class, grouping entries by ion type per library. Compute the fraction of entries per ion type for each library separately, reporting the share of [M+H]+ and the counts of the remaining adduct classes. Assemble a summary table with one row per library and ion type containing absolute counts and percentage shares, and save it as ion_identity_tally.csv. The analysis is motivated by the paper's finding that spectral libraries with narrow ion-species coverage limit annotation, which IIMN-based libraries with broad ion-species coverage are designed to address.

## Related tools

- **GNPS** (Public spectral library hosting and distribution — GNPS library endpoints (e.g. GNPS-IIMN-PROPOGATED) serve the reference spectra downloaded and tallied in this skill.) — http://gnps.ucsd.edu

## Examples

```
curl -o gnps-iimn-library.mgf "https://gnps.ucsd.edu/ProteoSAFe/gnpslibrary.jsp?library=GNPS-IIMN-PROPOGATED"
```

## Evaluation signals

- The computed [M+H]+ share reproduces the reported values: approximately 66% of MassBank of North America LC-MS2-positive entries and 65% of GNPS entries annotated as [M + H]+.
- In the MoNA LC-MS2-positive library, only four other ion types beyond [M+H]+ exceed 1000 entries — a sanity check on the adduct binning.
- Sum of per-ion-type counts equals the total number of library entries for each library (no entries dropped or double-binned).
- ion_identity_tally.csv contains one row per (library, ion type) pair with both absolute counts and percentage-share columns.
- Every parsed entry maps to exactly one of the defined classes ([M+H]+, [M+Na]+, [M+NH4]+, [M+K]+, [M-H2O+H]+, other).

## Limitations

- The bias statistics reflect the libraries as downloaded; public libraries evolve (e.g. IIMN-propagated libraries like GNPS-IIMN-PROPOGATED broaden ion-species coverage beyond the manually curated GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE).
- Only LC-MS2 positive-mode entries were tallied; the skill does not speak to negative-mode or GC library composition.
- Entries with missing or ambiguous adduct annotations may distort per-ion-type fractions; the 'other adducts' bin absorbs all non-listed classes.

## Evidence

- [abstract] IIMN-based spectral libraries with a broad coverage of ion species are publicly available.: "IIMN-based spectral libraries with a broad coverage of ion species are publicly available."
- [discussion] Library spectra matching criteria are merged into manually created GNPS libraries.: "merged into the existing manually created library GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE"
- [readme] The GNPS Workflows repository documents the GNPS server that hosts the spectral libraries used.: [evidence span withheld — anchor status: external_doc]
