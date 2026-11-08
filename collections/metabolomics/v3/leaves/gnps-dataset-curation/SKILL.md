---
name: gnps-dataset-curation
description: Use when MS/MS spectra loaded from MGF, mzML, mzXML, or msp files carry
  inconsistent or missing metadata (compound name, InChI, SMILES, InChIKey, ionmode,
  adduct, charge) and must be cleaned before intensity normalization or similarity
  scoring.
when_to_use_negative:
- The goal is peak-level filtering rather than metadata cleaning — matchms provides
  separate peak filters (select_by_mz, select_by_relative_intensity, require_minimum_number_of_peaks)
  for that purpose
- Spectra are already cleaned and harmonized and you only need similarity scoring
  (cosine score, modified cosine score) between them
- The metadata cannot be represented in matchms Spectrum objects, e.g. spectra visualization
  workflows better served by spectrum_utils
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: matchms
  role: Python package providing matchms.filtering.default_filters, the stacked metadata
    cleaning and harmonizing filter pipeline applied per Spectrum; imported spectra
    via matchms.importing.load_from_mgf
  repo: https://github.com/matchms/matchms
- name: pyteomics
  role: Underlies matchms's mzML, mzXML and MGF file importers that produce the Spectrum
    objects fed into default_filters
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:3c4fff8c0d627b1fe41a5f3004b717f2a76688cdd23f60eaa35c891f37ba3694
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - matchms
  - pyteomics
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gnps-dataset-curation

## Summary

Applies matchms's default_filters metadata-cleaning stack to mass spectrometry (MS/MS) spectra so that inconsistent, wrongly labeled, or incorrect metadata fields are corrected before downstream processing. It is the metadata-harmonization step of a reproducible spectral processing workflow built with matchms.

## When to use

Use when MS/MS spectra loaded from MGF, mzML, mzXML, or msp files carry inconsistent or missing metadata (compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge) and must be cleaned before intensity normalization or similarity scoring.

## When NOT to use

- The goal is peak-level filtering rather than metadata cleaning — matchms provides separate peak filters (select_by_mz, select_by_relative_intensity, require_minimum_number_of_peaks) for that purpose
- Spectra are already cleaned and harmonized and you only need similarity scoring (cosine score, modified cosine score) between them
- The metadata cannot be represented in matchms Spectrum objects, e.g. spectra visualization workflows better served by spectrum_utils

## Inputs

- Raw MS/MS mass spectra in MGF format (also supported: msp, mzML, mzXML, JSON, USI) parsed into matchms Spectrum objects
- Spectra with metadata fields: compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge

## Outputs

- Cleaned Spectrum objects with harmonized metadata fields (compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge)
- Filtered spectrum records serialized to filtered_spectra.json

## How to apply

Load parsed mass spectra into matchms Spectrum objects using matchms.importing.load_from_mgf (or the equivalent importer for mzML/mzXML/msp). Iterate over each spectrum individually and apply matchms.filtering.default_filters, which invokes the stacked metadata cleaning and harmonizing pipeline rather than a single transformation. Record the cleaned metadata for the key fields — compound name, structure annotations (InChI, SMILES, InChIKey), ionmode, adduct, and charge — for each spectrum. Emit the cleaned spectrum records (e.g. to filtered_spectra.json) and proceed to subsequent steps such as normalize_intensities and score calculation. The rationale is that large collected fragmentation MS datasets contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata, so cleaning must precede similarity measures like the cosine or modified cosine score.

## Related tools

- **matchms** (Python package providing matchms.filtering.default_filters, the stacked metadata cleaning and harmonizing filter pipeline applied per Spectrum; imported spectra via matchms.importing.load_from_mgf) — https://github.com/matchms/matchms
- **pyteomics** (Underlies matchms's mzML, mzXML and MGF file importers that produce the Spectrum objects fed into default_filters)

## Examples

```
from matchms.filtering import default_filters; spectrum = default_filters(spectrum)
```

## Evaluation signals

- Output records in filtered_spectra.json show cleaned/harmonized values for compound name, InChI, SMILES, InChIKey, ionmode, adduct, and charge, consistent with the documented behavior of default_filters
- Every spectrum loaded from the MGF file has been passed through default_filters individually (loop coverage matches the number of input spectra)
- Downstream steps (normalize_intensities, cosine/modified cosine score calculation) run without metadata-related errors on the cleaned spectra
- The cleaning stack is reproducible: re-running the workflow on the same input MGF yields identical cleaned metadata records

## Limitations

- Large collected fragmentation mass spectrometry datasets can contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata acquired on different instruments and measurement conditions, so cleaning may not resolve every field
- default_filters is a stacked pipeline of individual filters; it must be applied per spectrum (e.g. inside a loop) and may need to be complemented with other stacked filter functions to construct a desired pipeline
- Spectra whose metadata lacks fields such as InChIKey or adduct cannot be fully harmonized; validation requires checking which key fields are present per record

## Evidence

- [abstract] Available filters include extensive cleaning, correcting, checking of key metadata fields such as compound name, structure annotations (InChI, SMILES, InChIKey), ionmode, adduct, or charge: [evidence span withheld — anchor status: not_found]
- [abstract] spectrum = default_filters(spectrum): "spectrum = default_filters(spectrum)"
- [readme] Matchms offers an array of tools for metadata cleaning and validation, alongside basic peak filtering, to ensure data accuracy and integrity.: [evidence span withheld — anchor status: external_doc]
- [abstract] numerous metadata cleaning and harmonizing filter functions that can easily be stacked to construct a desired pipeline: "numerous metadata cleaning and harmonizing filter functions that can easily be stacked to construct a desired pipeline"
