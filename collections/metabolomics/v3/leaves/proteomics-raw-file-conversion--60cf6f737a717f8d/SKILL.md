---
name: proteomics-raw-file-conversion
description: Use when starting from vendor RAW files (e.g. HEK293 data downloaded
  from PRIDE project PXD001468) that must be converted to MGF before open spectral
  library searching with ANN-SoLo.
when_to_use_negative:
- The query spectra are already in MGF or another peak-list format the downstream
  search engine accepts directly — reconversion adds no information.
- The downstream tool requires a richer container format (e.g. mzML with full metadata)
  rather than the MGF peak lists consumed by the ANN-SoLo cascade search.
edam_operation: http://edamontology.org/operation_3434
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: msconvert
  role: Converts the downloaded HEK293 vendor RAW files to MGF peak-list format as
    the first workflow step before open spectral library search.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:20d20483c9edd7a846911fd0fbb103a662201c01b0b7cc9d3255b39a087909d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANN-SoLo
  - Python
  - SpectraST
  - msconvert
  edam_operation: http://edamontology.org/operation_3434
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# proteomics-raw-file-conversion

## Summary

Converts vendor mass-spectrometry RAW files (e.g. the HEK293 human cell line data from PRIDE PXD001468) into MGF format with msconvert so they can be consumed by the ANN-SoLo spectral library search engine.

## When to use

Use when starting from vendor RAW files (e.g. HEK293 data downloaded from PRIDE project PXD001468) that must be converted to MGF before open spectral library searching with ANN-SoLo.

## When NOT to use

- The query spectra are already in MGF or another peak-list format the downstream search engine accepts directly — reconversion adds no information.
- The downstream tool requires a richer container format (e.g. mzML with full metadata) rather than the MGF peak lists consumed by the ANN-SoLo cascade search.

## Inputs

- Vendor RAW files downloaded from PRIDE project PXD001468 (HEK293 human cell line data set)
- msconvert (ProteoWizard) installation

## Outputs

- MGF-format peak list files, one per raw input file, ready for spectral library search with ANN-SoLo

## How to apply

Download the raw files from the PRIDE project (PXD001468 for the HEK293 human cell line data set) and run msconvert to export each raw file to MGF format, preserving peak lists for subsequent spectrum preprocessing. The MGF output is the input format expected by ANN-SoLo's cascade open search, which will later apply its own preprocessing (removing noise peaks below 1% of the most intense peak, retaining the 50 most intense peaks, discarding spectra with fewer than 10 peaks or a mass range under 250 Da). Conversion is the first step of the workflow; downstream library preparation (duplicate removal with SpectraST and 1:1 decoy appending via the shuffle-and-reposition method) happens separately on the MassIVE-KB library, not on the query files. Verify the conversion by checking that each MGF contains the expected spectra and usable TITLE/scan identifiers so identifications can later be traced back to the original raw files.

## Related tools

- **msconvert** (Converts the downloaded HEK293 vendor RAW files to MGF peak-list format as the first workflow step before open spectral library search.)

## Examples

```
msconvert PXD001468_raws/*.raw --mgf -o mgf/
```

## Evaluation signals

- Each output MGF file corresponds 1:1 to an input raw file from PXD001468 and contains the expected number of spectra.
- MGF TITLE lines (or equivalent headers) retain scan identifiers so SSMs can be traced back to the original raw files.
- The MGF files are accepted by ANN-SoLo without format errors and pass its spectrum preprocessing (spectra with <10 peaks or <250 Da mass range are discarded downstream, so most converted spectra should survive).
- Downstream identification counts at 1% FDR can be compared against the results deposited at PXD009861, which is only possible if conversion was lossless enough for the search to reproduce published identifications.

## Limitations

- No version number for msconvert is stated anywhere in the article or repository context, so exact tool version reproducibility must be resolved independently.
- This skill covers only raw-to-MGF conversion; library-side preparation (removing duplicate spectra with SpectraST and appending decoys in a 1:1 ratio via shuffle-and-reposition) is a separate step on the MassIVE-KB library.

## Evidence

- [methods] Raw files were downloaded from PRIDE (project PXD001468): "Raw files were downloaded from PRIDE [63] (project PXD001468)"
- [other] Workflow step 1 from source card task_001: [evidence span withheld — anchor status: not_found]
- [methods] ANN-SoLo consumes preprocessed query spectra: "Prior to spectral library searching both the query spectra and library spectra are similarly preprocessed"
