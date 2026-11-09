---
name: spectrum-metadata-extraction
description: Use when raw mass spectra must be read from msp, mzML, mzXML, MGF, JSON,
  or USI files and their per-spectrum peak counts and metadata fields need to be extracted
  into structured records before filtering or similarity scoring.
when_to_use_negative:
- Input is already preprocessed spectrum data that only needs similarity score computation
  — go straight to the scoring workflow instead of re-importing.
- Input files are in an unsupported raw instrument format not among msp, mzML, mzXML,
  MGF, JSON, or metabolomics-USI.
- The task is spectra visualization; the article points to spectrum_utils as the lightweight,
  visualization-focused package instead.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Provides the importing functions (e.g. load_from_mgf) that parse MS/MS files
    into Spectrum objects with peaks and metadata
  repo: https://github.com/matchms/matchms
- name: pyteomics
  role: Underlying library on which the mzML, mzXML, and MGF file importers are built;
    also an alternative for metadata parsing
- name: pymzml
  role: Alternative Python library mentioned for handling spectrum-related metadata
    parsing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:adfb046429be6155fcf45c7103c2caab632381fabd1fd45508624e46786be48e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - matchms
  - pymzml
  - pyteomics
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# spectrum-metadata-extraction

## Summary

Import spectra stored in common MS/MS file formats (msp, mzML, mzXML, MGF, JSON, metabolomics-USI) into matchms Spectrum objects and harvest their peak lists plus metadata fields (compound name, InChI/SMILES/InChIKey, ionmode, adduct, charge, precursor m/z) for downstream cleaning or scoring.

## When to use

Use when raw mass spectra must be read from msp, mzML, mzXML, MGF, JSON, or USI files and their per-spectrum peak counts and metadata fields need to be extracted into structured records before filtering or similarity scoring.

## When NOT to use

- Input is already preprocessed spectrum data that only needs similarity score computation — go straight to the scoring workflow instead of re-importing.
- Input files are in an unsupported raw instrument format not among msp, mzML, mzXML, MGF, JSON, or metabolomics-USI.
- The task is spectra visualization; the article points to spectrum_utils as the lightweight, visualization-focused package instead.

## Inputs

- MGF-formatted spectrum file (e.g. tests/testdata/pesticides.mgf)
- msp file
- mzML file
- mzXML file
- JSON spectrum file
- metabolomics-USI identifier

## Outputs

- Iterator of matchms Spectrum objects with peak arrays and metadata dictionaries
- Per-spectrum records of peak counts and metadata fields (scans, ionmode, precursor_mz, compound name) suitable for a structured JSON dump

## How to apply

Instantiate the appropriate matchms importer for the file format at hand — e.g. matchms.importing.load_from_mgf for an MGF file — which yields an iterator of Spectrum objects each carrying a peaks array and a metadata dictionary. Iterate over the spectrums and record, per spectrum, the number of peaks and the available metadata entries such as 'scans', 'ionmode', 'precursor_mz', and compound name when annotated. Note that MGF parsing is built on top of pyteomics, so spectrum-related metadata parsing can alternatively be handled with pyteomics or pymzml if deeper control is needed. Because matchms supports a range of formats (mzML, mzXML, msp, metabolomics-USI, MGF, JSON), pick the loader matching the input extension rather than converting files manually. The extracted metadata matters because collected fragmentation datasets can contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata, so the harvested fields feed directly into the subsequent metadata cleaning and harmonizing filter pipeline.

## Related tools

- **matchms** (Provides the importing functions (e.g. load_from_mgf) that parse MS/MS files into Spectrum objects with peaks and metadata) — https://github.com/matchms/matchms
- **pyteomics** (Underlying library on which the mzML, mzXML, and MGF file importers are built; also an alternative for metadata parsing)
- **pymzml** (Alternative Python library mentioned for handling spectrum-related metadata parsing)

## Examples

```
from matchms.importing import load_from_mgf; spectrums = list(load_from_mgf("tests/testdata/pesticides.mgf"))
```

## Evaluation signals

- The parsed output contains one entry per imported spectrum, and the spectrum count equals the number of entries in the source MGF file.
- Each extracted record's metadata keys include at least the fields annotated in the source file (e.g. scans, ionmode, precursor_mz where present).
- Peak counts per spectrum are populated (non-empty peak arrays) for spectra that carry peaks in the input file.
- The loader chosen matches the input format (load_from_mgf for MGF, analogous loaders for mzML/mzXML/msp/JSON/USI), and no format-conversion errors occur.
- Downstream metadata-cleaning filters can be stacked on the extracted spectrums without key errors, indicating metadata fields were read correctly.

## Limitations

- Collected fragmentation mass spectrometry datasets can contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata, so extracted fields are not trustworthy until cleaning filters are applied.
- Users needing custom metadata parsing behavior are advised to rely on pyteomics or pymzml rather than matchms's built-in metadata handling.
- Current Python alternatives such as pyOpenMS may be harder to install for users; matchms exists partly to address that, but ecosystem compatibility (e.g. with spec2vec, ms2deepscore, ms2query) depends on the NumPy version installed.

## Evidence

- [abstract] matchms is an open-source Python package to import, process, clean, and compare mass spectrometry data: "matchms is an open-source Python package to import, process, clean, and compare mass spectrometry data (MS/MS)"
- [abstract] Raw data can be imported from the commonly used formats msp, mzML, mzXML, MGF: "Raw data can be imported from the commonly used formats msp, mzML (Martens et al., 2011), mzXML, MGF"
- [abstract] mzML, mzXML, MGF file importers are built on top of pyteomics: "mzML, mzXML, MGF file importers are built on top of pyteomics"
- [methods] file = load_from_mgf("all_your_spectrums.mgf"): "file = load_from_mgf("all_your_spectrums.mgf")"
- [readme] The software supports a range of popular spectral data formats, including mzML, mzXML, msp, metabolomics-USI, MGF, and JSON.: [evidence span withheld — anchor status: external_doc]
- [readme] tests/testdata/pesticides.mgf: [evidence span withheld — anchor status: not_found]
