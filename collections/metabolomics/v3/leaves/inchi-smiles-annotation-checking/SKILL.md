---
name: inchi-smiles-annotation-checking
description: Use when imported MS/MS spectra (MGF, msp, mzML, mzXML, JSON) carry inconsistent,
  wrongly labeled, or missing structure annotations (InChI, SMILES, InChIKey) or ionmode/adduct/charge
  fields that must be cleaned before scoring.
when_to_use_negative:
- The spectra are already cleaned and the task is peak filtering or similarity scoring
  (use select_by_mz, normalize_intensities, or cosine/modified cosine scores instead)
- The metadata problem is missing rather than incorrect annotations that need enrichment
  — matchms cleaning checks/corrects fields but does not harvest new metadata (consider
  MSMetaEnhancer for collecting library metadata from web services)
- Input is not mass spectrometry (MS/MS) spectral data with a metadata block to harmonize
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: matchms
  role: Provides the filtering module (default_filters and stacked metadata cleaning/harmonizing
    filter functions, DEFAULT_FILTERS/LIBRARY_CLEANING pipelines) used to check and
    correct InChI/SMILES/InChIKey and related metadata fields
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Used to implement the structure-annotation handling behind the metadata cleaning
    filters (InChI, SMILES, InChIKey checks)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:7c7e71f53408a77b2a0324bb2f74c42c16406ff5d244c659a1d07427b511d2a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - RDKit
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# InChI/SMILES annotation checking

## Summary

A metadata cleaning skill from the matchms ecosystem that checks, corrects, and harmonizes key spectrum metadata fields — compound name, structure annotations (InChI, SMILES, InChIKey), ionmode, adduct, and charge — so downstream similarity scoring runs on consistent annotations.

## When to use

Use when imported MS/MS spectra (MGF, msp, mzML, mzXML, JSON) carry inconsistent, wrongly labeled, or missing structure annotations (InChI, SMILES, InChIKey) or ionmode/adduct/charge fields that must be cleaned before scoring.

## When NOT to use

- The spectra are already cleaned and the task is peak filtering or similarity scoring (use select_by_mz, normalize_intensities, or cosine/modified cosine scores instead)
- The metadata problem is missing rather than incorrect annotations that need enrichment — matchms cleaning checks/corrects fields but does not harvest new metadata (consider MSMetaEnhancer for collecting library metadata from web services)
- Input is not mass spectrometry (MS/MS) spectral data with a metadata block to harmonize

## Inputs

- Mass spectra files in msp, mzML, mzXML, MGF, JSON, or metabolomics-USI format (e.g. tests/testdata/pesticides.mgf)
- matchms Spectrum objects with raw, possibly inconsistent metadata fields (compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge)

## Outputs

- Cleaned Spectrum objects with harmonized metadata for compound name, InChI, SMILES, InChIKey, ionmode, adduct, and charge
- Cleaned spectrum records serialized to JSON (e.g. filtered_spectra.json)
- Optionally a reusable YAML workflow file capturing the filter stack via the Pipeline class

## How to apply

Load parsed mass spectra into matchms Spectrum objects (e.g. from an MGF file via matchms.importing.load_from_mgf), then apply matchms.filtering.default_filters to each spectrum individually; this invokes the stacked pipeline of metadata cleaning and harmonizing filter functions. The filters clean, correct, and check the key metadata fields the article names: compound name, structure annotations (InChI, SMILES, InChIKey), ionmode, adduct, or charge — with structure-annotation handling implemented using rdkit. Rationale: the article notes that large collected fragmentation MS datasets 'can contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata', and cleaning must precede later steps such as intensity normalization (normalize_intensities) and score calculation (cosine / modified cosine). In newer matchms versions (>= 0.18.0) the same filter stacks can be run declaratively through the Pipeline class with predefined stacks like DEFAULT_FILTERS or LIBRARY_CLEANING from matchms.filtering.default_pipelines. Record the cleaned values for the key fields and emit the cleaned spectrum records (e.g. to filtered_spectra.json) for validation.

## Related tools

- **matchms** (Provides the filtering module (default_filters and stacked metadata cleaning/harmonizing filter functions, DEFAULT_FILTERS/LIBRARY_CLEANING pipelines) used to check and correct InChI/SMILES/InChIKey and related metadata fields) — https://github.com/matchms/matchms
- **RDKit** (Used to implement the structure-annotation handling behind the metadata cleaning filters (InChI, SMILES, InChIKey checks))

## Examples

```
from matchms.importing import load_from_mgf; from matchms.filtering import default_filters; cleaned = [default_filters(s) for s in load_from_mgf('pesticides.mgf')]
```

## Evaluation signals

- Output records show cleaned/harmonized values for the key metadata fields (compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge) rather than raw file labels
- Structure annotation fields are internally consistent — InChIKey derivable from the InChI/SMILES (checked via rdkit-based filters)
- Every spectrum in the batch has passed through default_filters exactly once before any normalize_intensities or score-computation step
- Ionmode/adduct/charge fields no longer contain the inconsistent or wrongly labeled values present in the raw imported file
- When run through Pipeline, the emitted YAML workflow file reproduces the same filter stack on rerun

## Limitations

- Cleaning can only check/correct what is present; large collected datasets may still contain inconsistent, wrongly labeled, or incorrect metadata that filters cannot fully repair
- Filter stacks must be chosen/stacked deliberately — default_filters is a specific stack, and matchms requires users to construct 'a desired pipeline' for other needs
- Version-dependent API: since matchms >= 0.18.0, predefined stacks (DEFAULT_FILTERS, LIBRARY_CLEANING) and the Pipeline class are the recommended entry points, so older loop-based snippets may not match current documentation

## Evidence

- [abstract] Available filters include extensive cleaning, correcting, checking of key metadata fields such as compound name, structure annotations (InChI, SMILES, InChIKey), ionmode, adduct, or charge: [evidence span withheld — anchor status: not_found]
- [readme] Matchms offers an array of tools for metadata cleaning and validation, alongside basic peak filtering, to ensure data accuracy and integrity.: [evidence span withheld — anchor status: external_doc]
- [abstract] numerous metadata cleaning and harmonizing filter functions that can easily be stacked to construct a desired pipeline: "numerous metadata cleaning and harmonizing filter functions that can easily be stacked to construct a desired pipeline"
