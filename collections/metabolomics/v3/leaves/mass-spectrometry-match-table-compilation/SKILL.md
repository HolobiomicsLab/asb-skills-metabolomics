---
name: mass-spectrometry-match-table-compilation
description: Use when you have MASST search results (per-compound job/USI identifiers)
  for a panel of reference tandem MS spectra and must aggregate them into a per-compound
  match table with deduplicated file counts.
when_to_use_negative:
- When the input is raw LC-MS/MS instrument data rather than already-completed MASST
  search results for reference spectra — this skill tabulates matches, it does not
  run spectra acquisition or library creation
- 'When you need definitive structural assignment of a matched compound: without isolation
  and NMR or X-ray structural analyses, isomers cannot be excluded, so the table only
  supports candidate-level claims'
- When the target molecules are not represented in public repositories — MASST's ~1.2
  billion spectra do not encompass all molecules, so zero-match compounds may reflect
  coverage limits rather than absence of the compound
edam_operation: http://edamontology.org/operation_3799
tools:
- name: MASST
  role: Search engine whose per-compound job results (USI-retrievable) supply the
    spectral matches and matched public data files to be tabulated and deduplicated
- name: Nextflow
  role: Workflow engine used by the authors' repository to run the library-generation/matching
    workflow reproducibly
  repo: https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation
- name: Thermo QE Orbitrap
  role: Source instrument for negative-mode reference MS/MS spectra of conjugated
    bile acids (e.g. iHMP2 stool samples) that determine the MASST polarity setting
- name: Bruker Q-ToF
  role: Source instrument for positive-mode reference MS/MS spectra (e.g. bacterial
    cultures and human IBD fecal samples) that determine the MASST polarity setting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:01c7bedead55ab24996e6b7ef732feeb2d05f5bf76e71b64ed780775c67911ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation
  tools:
  - Agilent 6560 IM-Q-ToF MS
  - Bruker Q-ToF
  - MASST
  - Nextflow
  - Thermo QE Orbitrap
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mass-spectrometry-match-table-compilation

## Summary

Compile a per-compound table of spectral matches and unique matched data files returned by MASST searches of reference MS/MS spectra (e.g. synthesized conjugated bile acids) against the ~1.2 billion public spectra in GNPS/MassIVE, enabling aggregate reuse-validation and downstream disease-association analysis.

## When to use

Use when you have MASST search results (per-compound job/USI identifiers) for a panel of reference tandem MS spectra and must aggregate them into a per-compound match table with deduplicated file counts.

## When NOT to use

- When the input is raw LC-MS/MS instrument data rather than already-completed MASST search results for reference spectra — this skill tabulates matches, it does not run spectra acquisition or library creation
- When you need definitive structural assignment of a matched compound: without isolation and NMR or X-ray structural analyses, isomers cannot be excluded, so the table only supports candidate-level claims
- When the target molecules are not represented in public repositories — MASST's ~1.2 billion spectra do not encompass all molecules, so zero-match compounds may reflect coverage limits rather than absence of the compound

## Inputs

- Reference MS/MS spectra of synthesized compounds (e.g. conjugated bile acids) from MassIVE deposits MSV000087522 and MSV000087523
- Published USI and MASST job identifiers for each synthesized compound (supplementary material)
- Raw MASST search results per compound against GNPS/MassIVE public spectra, matched per acquisition polarity (positive Bruker Q-ToF vs negative Thermo QE Orbitrap / Agilent 6560 IM-Q-ToF)

## Outputs

- Per-compound match table with columns: compound identifier, MASST job/USI, instrument platform, total match count, number of unique matched files
- Aggregate summary statistics (e.g. 16,587 spectral matches from 6,603 unique files across 145 compounds)
- Flags annotating matches with isomer ambiguity

## How to apply

Download the reference tandem MS spectra (here, synthesized bile amidates from MassIVE deposits MSV000087522, Bruker Q-ToF positive mode, and MSV000087523, Thermo QE Orbitrap negative mode), retrieve the published USI and MASST job identifiers for all synthesized compounds from the supplementary material, and submit each compound's MS/MS spectrum as a MASST search against public data, matching the acquisition polarity of the originating instrument platform. Parse the returned MASST results for each compound, deduplicating matched dataset files so each public file is counted once per compound. Populate one row per compound with columns: compound identifier, MASST job/USI, instrument platform, total spectral match count, and number of unique matched files. For the bile amidate case this aggregate table should reproduce 16,587 spectral matches from 6,603 unique public data files covering 145 synthesized compounds, and any match that cannot be structurally resolved (isomer ambiguity) must be explicitly flagged rather than counted as a definitive identification.

## Related tools

- **MASST** (Search engine whose per-compound job results (USI-retrievable) supply the spectral matches and matched public data files to be tabulated and deduplicated)
- **Nextflow** (Workflow engine used by the authors' repository to run the library-generation/matching workflow reproducibly) — https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation
- **Thermo QE Orbitrap** (Source instrument for negative-mode reference MS/MS spectra of conjugated bile acids (e.g. iHMP2 stool samples) that determine the MASST polarity setting)
- **Bruker Q-ToF** (Source instrument for positive-mode reference MS/MS spectra (e.g. bacterial cultures and human IBD fecal samples) that determine the MASST polarity setting)

## Evaluation signals

- Aggregate table reproduces the published summary of 16,587 spectral matches from 6,603 unique public data files covering 145 synthesized compounds
- Deduplication invariant: each matched public data file is counted exactly once per compound, verified by comparing file counts before and after deduplication
- Every table row carries all five required columns (compound identifier, MASST job/USI, instrument platform, total match count, unique matched files)
- Compounds are searched under the correct acquisition polarity for their instrument platform (Bruker Q-ToF positive vs Thermo QE Orbitrap negative)
- Matches that cannot be structurally resolved without NMR or X-ray analysis carry an explicit isomer-ambiguity flag

## Limitations

- Without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data, so match counts are not unique-structure counts
- The MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds, although it prevents completely inaccurate identifications
- The ~1.2 billion spectra in MASST searches do not encompass all molecules, limiting re-analysis coverage for the discovery phase
- Metabolites must be present in sufficient quantities or ionize well to be detected and selected for MS/MS acquisition; compounds lacking reference MS/MS spectra cannot be tabulated

## Evidence

- [supplementary] USI and MASST job identifiers for all synthesized compounds.: "USI and MASST job identifiers for all synthesized compounds."
- [discussion] MASST coverage limitation: "Last, the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules."
- [discussion] Isomer ambiguity caveat: "without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers"
- [discussion] False-discovery caveat: "Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications."
- [supplementary] Instrument/polarity context for inputs: "conjugated bile acids detected in iHMP2 stool samples using a Thermo QE Orbitrap in negative mode."
