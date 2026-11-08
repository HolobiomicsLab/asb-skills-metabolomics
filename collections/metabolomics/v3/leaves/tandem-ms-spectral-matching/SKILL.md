---
name: tandem-ms-spectral-matching
description: Use when newly acquired tandem MS/MS spectra (e.g. from synthesized compounds)
  must be searched repository-wide against public metabolomics data to find which
  public datasets contain the compounds.
when_to_use_negative:
- The metabolite of interest is absent or ionizes poorly in MS/MS — reverse metabolomics
  requires an MS/MS spectrum, and the ~1.2 billion MASST spectra do not encompass
  all molecules
- Structural isomer resolution is required for the biological claim and no NMR or
  X-ray analysis is available — MASST matching cannot exclude different isomers
- Input is already aggregate-level data (feature tables or abundance matrices) rather
  than raw tandem MS/MS spectra
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: MASST
  role: Searches each compound's MS/MS spectrum against ~1.2 billion public spectra
    in GNPS/MassIVE; returns spectral matches and matched dataset files per job
- name: Bruker Q-ToF
  role: Acquired conjugated bile acid MS/MS spectra in positive mode (MassIVE deposit
    MSV000087522) used as MASST search queries
- name: Thermo QE Orbitrap
  role: Acquired conjugated bile acid MS/MS spectra in negative mode (MassIVE deposit
    MSV000087523) used as MASST search queries
- name: Agilent 6560 IM-Q-ToF MS
  role: Acquired conjugated bile acid MS/MS spectra in negative mode from bacterial
    cultures for library/searching use
- name: Nextflow
  role: Workflow engine for the Reverse_metabolomics_library_generation pipeline that
    generates/deploys the GNPS2 library-generation workflow
  repo: https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:d0c3f81fcb95b05301db7d54e661ad26137dd76034f8d94d651ba5583db4d2b1
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
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tandem-ms-spectral-matching

## Summary

Searches MS/MS spectra of synthesized or candidate compounds (here conjugated bile acids/bile amidates) against ~1.2 billion public GNPS/MassIVE spectra using MASST, recovering spectral matches and unique public data files to connect structures to phenotypes.

## When to use

Use when newly acquired tandem MS/MS spectra (e.g. from synthesized compounds) must be searched repository-wide against public metabolomics data to find which public datasets contain the compounds.

## When NOT to use

- The metabolite of interest is absent or ionizes poorly in MS/MS — reverse metabolomics requires an MS/MS spectrum, and the ~1.2 billion MASST spectra do not encompass all molecules
- Structural isomer resolution is required for the biological claim and no NMR or X-ray analysis is available — MASST matching cannot exclude different isomers
- Input is already aggregate-level data (feature tables or abundance matrices) rather than raw tandem MS/MS spectra

## Inputs

- Reference tandem MS/MS spectra of synthesized compounds (from MassIVE deposits MSV000087522 Bruker Q-ToF positive mode and MSV000087523 Thermo QE Orbitrap negative mode)
- USI and MASST job identifiers per compound from supplementary material
- Instrument/acquisition polarity annotation per spectrum (positive vs negative mode)

## Outputs

- Per-compound MASST match table (compound identifier, MASST job/USI, instrument platform, total match count, number of unique matched files)
- Deduplicated set of unique public data files matching each compound
- Aggregate match summary (e.g. 16,587 matches / 6,603 unique files / 145 compounds)
- Isomer-ambiguity flags for structurally unresolved matches

## How to apply

Acquire reference tandem MS/MS spectra for the candidate compounds on the platforms used in the study (Bruker Q-ToF positive mode, Thermo QE Orbitrap negative mode, or Agilent 6560 IM-Q-ToF negative mode), using the collision values specified for Q-ToF acquisition. For each compound, submit its MS/MS spectrum as a MASST search against GNPS/MassIVE (~1.2 billion public spectra), matching the acquisition polarity to the instrument platform; retrieve the compound's USI and MASST job identifiers from the supplementary material for reproducibility. Parse the MASST results per compound and deduplicate matched dataset files so each public file counts once per compound. Compile a per-compound table (compound identifier, MASST job/USI, instrument platform, total match count, unique matched file count); the bile amidate reference run returned 16,587 spectral matches from 6,603 unique public data files across 145 synthesized compounds. Flag matches that cannot resolve isomer identity without NMR or X-ray analysis.

## Related tools

- **MASST** (Searches each compound's MS/MS spectrum against ~1.2 billion public spectra in GNPS/MassIVE; returns spectral matches and matched dataset files per job)
- **Bruker Q-ToF** (Acquired conjugated bile acid MS/MS spectra in positive mode (MassIVE deposit MSV000087522) used as MASST search queries)
- **Thermo QE Orbitrap** (Acquired conjugated bile acid MS/MS spectra in negative mode (MassIVE deposit MSV000087523) used as MASST search queries)
- **Agilent 6560 IM-Q-ToF MS** (Acquired conjugated bile acid MS/MS spectra in negative mode from bacterial cultures for library/searching use)
- **Nextflow** (Workflow engine for the Reverse_metabolomics_library_generation pipeline that generates/deploys the GNPS2 library-generation workflow) — https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation

## Examples

```
make run
```

## Evaluation signals

- Aggregate per-compound match table reproduces the reported summary: 16,587 spectral matches from 6,603 unique public data files covering 145 synthesized compounds
- Per-compound MASST job and USI identifiers match those published in the supplementary material
- Each matched public dataset file is counted at most once per compound after deduplication
- Search polarity matches acquisition polarity (positive mode for Bruker Q-ToF spectra, negative mode for Thermo QE Orbitrap and Agilent 6560 spectra)
- Matches that cannot be structurally resolved to a single isomer are explicitly flagged

## Limitations

- MASST matching does not guarantee against false discovery of closely related isomers or compounds, although it prevents completely inaccurate identifications
- Without isolation and NMR or X-ray structural analyses, different isomers cannot be excluded from the spectral matches
- The ~1.2 billion spectra in MASST do not encompass all molecules, limiting re-analysis of public data in the discovery phase
- Combinatorial synthesis requires a prior hypothesis about structures of interest, and complex molecules reduce throughput

## Evidence

- [abstract] Reverse metabolomics: acquire tandem MS spectra from newly synthesized compounds and search them in public metabolomics datasets to uncover phenotypic associations: "whereby tandem mass spectrometry spectra acquired from newly synthesized compounds are searched for in public metabolomics datasets to uncover phenotypic associations"
- [results] Aggregate match summary: [evidence span withheld — anchor status: not_found]
- [discussion] Isomer ambiguity limitation: "However, without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data."
- [discussion] MASST coverage limitation: "Last, the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules."
- [readme] Workflow execution via Nextflow/Makefile: "To run the workflow to test simply do

make run"
