---
name: proteomics-repository-data-retrieval
description: Use when an analysis needs raw spectra or spectral libraries identified
  only by repository accession numbers (e.g. MassIVE MSV000078492, PRIDE PXD001468/PXD009861)
  and they must be downloaded and combined into a searchable library.
when_to_use_negative:
- The data are already local spectral library files in searchable format and only
  need preprocessing, not retrieval or library construction
- The task is sequence database searching (e.g. MSFragger-style) rather than spectral
  library searching, where a spectral library build is unnecessary
- Only a subset of spectra is needed for a quick test and downloading full repository
  datasets would be excessive
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: SpectraST
  role: Concatenates the NIST human HCD and Selevsek yeast spectral libraries while
    removing duplicates (keeping the best replicate spectrum per peptide ion)
- name: ANN-SoLo
  role: Consumes the assembled target-decoy spectral library and downloaded raw spectra
    for cascade open-search identification
  repo: https://github.com/bittremieux/ANN-SoLo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:9c2206012a9e2c494860b325fd8218e3e46185848d23a51c6fe43537d2104d61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Python
  - SpectraST
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# proteomics-repository-data-retrieval

## Summary

Retrieve raw mass spectrometry spectra and reference spectral libraries from public proteomics repositories (MassIVE, PRIDE/ProteomeXchange) and assemble them into a search-ready target-decoy spectral library. This is the data-provisioning step that underpins downstream open-modification spectral library searches such as ANN-SoLo's.

## When to use

Use when an analysis needs raw spectra or spectral libraries identified only by repository accession numbers (e.g. MassIVE MSV000078492, PRIDE PXD001468/PXD009861) and they must be downloaded and combined into a searchable library.

## When NOT to use

- The data are already local spectral library files in searchable format and only need preprocessing, not retrieval or library construction
- The task is sequence database searching (e.g. MSFragger-style) rather than spectral library searching, where a spectral library build is unnecessary
- Only a subset of spectra is needed for a quick test and downloading full repository datasets would be excessive

## Inputs

- Repository accession numbers (e.g. MassIVE MSV000078492, PRIDE PXD001468, PXD009861)
- Raw mass spectrometry data files (e.g. iPRG2012 TripleTOF spectra, HEK293 human cell line spectra)
- Reference spectral libraries (e.g. NIST human HCD library version 2016/09/12, Selevsek et al. TripleTOF yeast library, MassIVE-KB library version 2017/11/27)
- Protein sequence databases (e.g. neXtProt version 2018/01/17, cRAP contaminants version 2012/01/01)

## Outputs

- Downloaded raw spectra files
- Concatenated target-decoy spectral library with 1:1 decoy ratio (e.g. 1,188,168 library spectra)
- Deduplicated library in SpectraST format ready for search

## How to apply

Identify each dataset's accession and repository: raw files (e.g. the iPRG2012 TripleTOF data) are downloaded from MassIVE (accession MSV000078492), while raw files for other studies (e.g. the HEK293 human cell line data) are downloaded from PRIDE via ProteomeXchange (PXD001468, PXD009861). Next, build the spectral search library: remove decoy-protein matches from the source library (the Selevsek et al. TripleTOF yeast library), concatenate with a reference library such as the NIST human HCD spectral library (version 2016/09/12) using SpectraST while keeping only the best replicate per peptide ion, then add decoy spectra in a 1:1 ratio with the shuffle-and-reposition method, yielding ~1.19 million library spectra. Pair downloaded raw spectra with appropriate protein sequence databases for validation (e.g. neXtProt version 2018/01/17 complemented with cRAP version 2012/01/01 for HEK293). Verify success by checking accession resolution, library size, and that no duplicate peptide-ion entries or missing decoys remain.

## Related tools

- **SpectraST** (Concatenates the NIST human HCD and Selevsek yeast spectral libraries while removing duplicates (keeping the best replicate spectrum per peptide ion))
- **ANN-SoLo** (Consumes the assembled target-decoy spectral library and downloaded raw spectra for cascade open-search identification) — https://github.com/bittremieux/ANN-SoLo

## Examples

```
spectrast -cBNIST_human_HCD_2016-09-12 -cJyeast_library.mgf -cNsearch_library human_hcd.mgf yeast.mgf
```

## Evaluation signals

- All accessions (MSV000078492, PXD001468, PXD009861) resolve to downloadable files at their respective repositories
- The assembled library contains the expected number of spectra (1,188,168 for the iPRG2012 search library) with a 1:1 target-to-decoy ratio
- SpectraST deduplication retained only the best replicate spectrum per peptide ion, with no duplicate peptide-ion entries
- Decoy spectra were generated with the shuffle-and-reposition method and are present for every target spectrum
- Library provenance is documented: source libraries (NIST 2016/09/12, Selevsek yeast), versions, and any decoy removal steps are recorded

## Limitations

- Concatenating and deduplicating large libraries via SpectraST can drop replicate spectra that might contain valid identifications; only the best replicate per peptide ion is kept
- Decoy generation by shuffling assumes target spectra are correctly annotated; misassigned peptides propagate into decoys
- Downloaded raw data formats and instrument types (e.g. TripleTOF) must match the search engine's expectations; version mismatches between library versions (e.g. NIST 2016/09/12 vs MassIVE-KB 2017/11/27) affect comparability

## Evidence

- [methods] All data was downloaded from the MassIVE data repository (accession MSV000078492): "All data was downloaded from the MassIVE data repository (accession MSV000078492)"
- [methods] decoy spectra were added in a 1:1 ratio using the shuffle-and-reposition method: "decoy spectra were added in a 1:1 ratio using the shuffle-and-reposition method"
- [methods] both spectral libraries were concatenated using SpectraST while removing duplicates: "both spectral libraries were concatenated using SpectraST [42] while removing duplicates"
- [methods] Raw files were downloaded from PRIDE (project PXD001468): "Raw files were downloaded from PRIDE [63] (project PXD001468)"
- [methods] First, matches to decoy proteins were removed from the yeast spectral library: "First, matches to decoy proteins were removed from the yeast spectral library"
