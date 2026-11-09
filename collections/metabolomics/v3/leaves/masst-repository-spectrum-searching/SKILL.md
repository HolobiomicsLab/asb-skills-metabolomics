---
name: masst-repository-spectrum-searching
description: Use when you hold a reference tandem MS/MS spectrum from a synthesized
  standard (e.g., bile amidate acquired in positive Q-ToF or negative Orbitrap mode)
  and need the set of public GNPS/MassIVE data files containing that molecule.
when_to_use_negative:
- The target compound has no MS/MS spectrum — it must be present in sufficient quantities
  or ionize well to be detected and selected for MS/MS acquisition.
- A definitive structural assignment is required — without isolation and NMR or X-ray
  analysis, MASST matches cannot exclude different isomers.
- The goal is discovering entirely novel structures with no prior structural hypothesis
  — the combinatorial-synthesis-plus-MASST strategy requires hypotheses about structures
  of interest.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MASST
  role: Submits each compound's reference MS/MS spectrum as a search against the ~1.2
    billion public spectra in GNPS/MassIVE and returns per-compound matched datasets
    and files
- name: Bruker Q-ToF
  role: Acquired the positive-mode bile amidate reference spectra deposited as MassIVE
    MSV000087522 that are searched via MASST
- name: Thermo QE Orbitrap
  role: Acquired the negative-mode conjugated bile acid reference spectra (MassIVE
    MSV000087523, e.g., iHMP2 stool samples) that are searched via MASST
- name: Agilent 6560 IM-Q-ToF MS
  role: Negative-mode instrument used to confirm synthesized conjugated bile acid
    spectra in bacterial cultures, providing additional reference spectra for MASST
    searches
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:cbf2c8489156566022596eabed522e0eeb74cd2ae4489712a2e5c067dbe3fbbc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Agilent 6560 IM-Q-ToF MS
  - Bruker Q-ToF
  - MASST
  - Thermo QE Orbitrap
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# masst-repository-spectrum-searching

## Summary

Searches each candidate compound's tandem MS/MS spectrum (here, synthesized conjugated bile acids/bile amidates) against the ~1.2 billion public spectra in GNPS/MassIVE using MASST, returning the public data files containing that molecule. This repository-scale reverse metabolomics step links molecular structures to phenotypes such as IBD.

## When to use

Use when you hold a reference tandem MS/MS spectrum from a synthesized standard (e.g., bile amidate acquired in positive Q-ToF or negative Orbitrap mode) and need the set of public GNPS/MassIVE data files containing that molecule.

## When NOT to use

- The target compound has no MS/MS spectrum — it must be present in sufficient quantities or ionize well to be detected and selected for MS/MS acquisition.
- A definitive structural assignment is required — without isolation and NMR or X-ray analysis, MASST matches cannot exclude different isomers.
- The goal is discovering entirely novel structures with no prior structural hypothesis — the combinatorial-synthesis-plus-MASST strategy requires hypotheses about structures of interest.

## Inputs

- Reference tandem MS/MS spectra of synthesized bile amidates from MassIVE deposits MSV000087522 (Bruker Q-ToF, positive mode) and MSV000087523 (Thermo QE Orbitrap, negative mode)
- Published USI and MASST job identifiers for all synthesized compounds (supplementary material)
- Acquisition polarity and instrument platform metadata per compound (Bruker Q-ToF positive mode; Thermo QE Orbitrap negative mode; Agilent 6560 IM-Q-ToF MS negative mode)

## Outputs

- Per-compound MASST match table (compound identifier, MASST job/USI, instrument platform, total spectral match count, number of unique matched public data files)
- Aggregate match summary (e.g., 16,587 spectral matches from 6,603 unique files across 145 synthesized compounds)
- Isomer-ambiguity flags for matches that cannot be structurally resolved

## How to apply

Obtain the reference tandem MS/MS spectra by downloading the synthesized bile amidate mixtures from the MassIVE deposits MSV000087522 (Bruker Q-ToF, positive mode) and MSV000087523 (Thermo QE Orbitrap, negative mode), or by retrieving the published USI and MASST job identifiers for all synthesized compounds from the supplementary material. Submit each compound's MS/MS spectrum as a MASST search against the ~1.2 billion public spectra in GNPS/MassIVE, matching the acquisition polarity to the instrument platform that produced the spectrum. Parse the returned MASST results per compound and deduplicate matched dataset files so each public file is counted once per compound. Compile a per-compound table with compound identifier, MASST job/USI, instrument platform, total match count, and number of unique matched files. Validate by confirming the aggregate reproduces the reported summary — 16,587 spectral matches from 6,603 unique public data files covering 145 synthesized compounds — and flag any match whose isomer identity cannot be resolved without NMR or X-ray analysis.

## Related tools

- **MASST** (Submits each compound's reference MS/MS spectrum as a search against the ~1.2 billion public spectra in GNPS/MassIVE and returns per-compound matched datasets and files)
- **Bruker Q-ToF** (Acquired the positive-mode bile amidate reference spectra deposited as MassIVE MSV000087522 that are searched via MASST)
- **Thermo QE Orbitrap** (Acquired the negative-mode conjugated bile acid reference spectra (MassIVE MSV000087523, e.g., iHMP2 stool samples) that are searched via MASST)
- **Agilent 6560 IM-Q-ToF MS** (Negative-mode instrument used to confirm synthesized conjugated bile acid spectra in bacterial cultures, providing additional reference spectra for MASST searches)

## Examples

```
make run
```

## Evaluation signals

- Aggregate output reproduces the published benchmark: 16,587 spectral matches from 6,603 unique public data files covering 145 of the synthesized compounds.
- Per-compound match rows are deduplicated so each public data file is counted once per compound.
- Every row carries compound identifier, MASST job/USI, instrument platform, total match count, and unique matched file count.
- Search polarity matches the acquisition polarity of each reference spectrum (positive-mode Bruker Q-ToF vs negative-mode Thermo QE Orbitrap / Agilent 6560 IM-Q-ToF).
- Matches confounded by structural isomers are flagged as ambiguous rather than assigned a single structure.

## Limitations

- Isomer ambiguity: without isolation and NMR or X-ray structural analyses, different isomers cannot be excluded from a match.
- The ~1.2 billion spectra in MASST searches do not encompass all molecules, limiting re-analysis of public data in the discovery phase.
- The protocol does not guarantee against false discovery of closely related isomers or compounds, although it prevents completely inaccurate identifications.
- Combinatorial synthesis of candidate spectra requires a prior hypothesis about structures of interest, and more complex molecules reduce throughput.

## Evidence

- [abstract] whereby tandem mass spectrometry spectra acquired from newly synthesized compounds are searched for in public metabolomics datasets to uncover phenotypic associations: "whereby tandem mass spectrometry spectra acquired from newly synthesized compounds are searched for in public metabolomics datasets to uncover phenotypic associations"
- [abstract] Using repository-scale analysis 1,2 , we discovered that some conjugated bile acids are associated with inflammatory bowel disease (IBD).: "Using repository-scale analysis 1,2 , we discovered that some conjugated bile acids are associated with inflammatory bowel disease (IBD)."
- [supplementary] USI and MASST job identifiers for all synthesized compounds.: "USI and MASST job identifiers for all synthesized compounds."
- [supplementary] conjugated bile acids detected in iHMP2 stool samples using a Thermo QE Orbitrap in negative mode.: "conjugated bile acids detected in iHMP2 stool samples using a Thermo QE Orbitrap in negative mode."
- [discussion] However, without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data.: "However, without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data."
- [discussion] Last, the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules.: "Last, the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules."
- [discussion] Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications.: "Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications."
