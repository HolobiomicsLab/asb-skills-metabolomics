---
name: mass-spectrometry-data-cleaning
description: Use when importing raw MS/MS spectra (mgf, mzML, mzXML, msp, JSON, USI)
  whose metadata may be inconsistent, wrongly labeled, or incorrect, and a harmonized,
  filter-cleaned spectrum set is needed downstream.
when_to_use_negative:
- Spectra whose metadata is already curated and only pairwise similarity scoring is
  needed — go straight to cosine/modified cosine scores instead of re-running cleaning
  filters
- Raw, unprocessed file formats not among the matchms importers (msp, mzML, mzXML,
  MGF, JSON, USI) that would require a different ingestion path first
- Peak-level analysis where metadata harmonization is irrelevant and only intensity
  normalization (normalize_intensities) or peak filtering (select_by_mz) is the goal
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: matchms
  role: Provides load_from_mgf for importing spectra and matchms.filtering.default_filters
    plus stacked filter functions (select_by_mz, select_by_relative_intensity, require_minimum_number_of_peaks,
    normalize_intensities) for metadata cleaning and harmonization
  repo: https://github.com/matchms/matchms
- name: pyteomics
  role: Underlying parser library for mzML, mzXML, and MGF importers in matchms
- name: pymzml
  role: Alternative metadata parsing library noted for handling MS/MS metadata in
    matchms
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:126e7aebcc0a73096ed1cff4985fd754a9a50f0f0c586051ac22c1e4e4a92221
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - matchms
  - pymzml
  - pyteomics
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# mass-spectrometry-data-cleaning

## Summary

Clean and harmonize MS/MS spectrum metadata by stacking matchms filter functions — including default_filters — into a reproducible pipeline that corrects and checks key fields such as compound name, InChI, SMILES, InChIKey, ionmode, adduct, and charge before scoring.

## When to use

Use when importing raw MS/MS spectra (mgf, mzML, mzXML, msp, JSON, USI) whose metadata may be inconsistent, wrongly labeled, or incorrect, and a harmonized, filter-cleaned spectrum set is needed downstream.

## When NOT to use

- Spectra whose metadata is already curated and only pairwise similarity scoring is needed — go straight to cosine/modified cosine scores instead of re-running cleaning filters
- Raw, unprocessed file formats not among the matchms importers (msp, mzML, mzXML, MGF, JSON, USI) that would require a different ingestion path first
- Peak-level analysis where metadata harmonization is irrelevant and only intensity normalization (normalize_intensities) or peak filtering (select_by_mz) is the goal

## Inputs

- MGF-formatted mass spectrometry file (e.g. tests/testdata/pesticides.mgf)
- mzML, mzXML, msp, JSON, or metabolomics-USI spectral data files
- matchms Spectrum objects obtained via load_from_mgf or equivalent importers

## Outputs

- Cleaned matchms Spectrum objects with harmonized metadata fields (compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge)
- Structured JSON record of cleaned spectra, e.g. filtered_spectra.json
- Optional workflow YAML (e.g. my_config_file.yaml) capturing the filter pipeline for reuse

## How to apply

Load spectra from an MGF (or mzML/mzXML/msp/JSON/USI) file with matchms.importing.load_from_mgf, which yields Spectrum objects. Iterate over each Spectrum and apply matchms.filtering.default_filters, which invokes the stacked metadata cleaning and harmonizing pipeline described in the paper. Target the key metadata fields the article enumerates — compound name, structure annotations (InChI, SMILES, InChIKey), ionmode, adduct, and charge. Additional filter steps named in the article (select_by_mz, select_by_relative_intensity, require_minimum_number_of_peaks, normalize_intensities) can be stacked alongside the defaults to build the desired pipeline. Emit the cleaned records (e.g. to filtered_spectra.json) and verify the key fields carry harmonized values before downstream steps like normalize_intensities or score calculation. In newer matchms (≥ 0.18.0), the same cleaning logic is accessible via the Pipeline class, e.g. passing DEFAULT_FILTERS or LIBRARY_CLEANING as query_filters/reference_filters in a create_workflow YAML config, which makes the cleaning pipeline shareable and rerunnable.

## Related tools

- **matchms** (Provides load_from_mgf for importing spectra and matchms.filtering.default_filters plus stacked filter functions (select_by_mz, select_by_relative_intensity, require_minimum_number_of_peaks, normalize_intensities) for metadata cleaning and harmonization) — https://github.com/matchms/matchms
- **pyteomics** (Underlying parser library for mzML, mzXML, and MGF importers in matchms)
- **pymzml** (Alternative metadata parsing library noted for handling MS/MS metadata in matchms)

## Examples

```
from matchms.Pipeline import Pipeline, create_workflow; from matchms.filtering.default_pipelines import DEFAULT_FILTERS; pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS, score_computations=[["cosinegreedy", {"tolerance": 1.0}]])); pipeline.run("tests/testdata/pesticides.mgf")
```

## Evaluation signals

- The output JSON (filtered_spectra.json) exists, is valid JSON, and contains one entry per spectrum from the input MGF
- Each record's metadata keys include at least the fields present in the source MGF (e.g. scans, ionmode where annotated)
- Key fields — compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge — show cleaned/harmonized values consistent with matchms documented filter behavior
- Spectrum count in the output equals the number of spectra entries in the input MGF file
- When using the Pipeline class, the emitted my_config_file.yaml reproduces the workflow on rerun and the pipeline log records the applied filter steps

## Limitations

- Large collected datasets can contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata — cleaning filters address this but may silently drop or alter entries
- Metadata parsing can best be handled via pyteomics or pymzml; other parsers may miss fields
- Python 3.10–3.14 are the systematically tested versions; higher versions work but are not yet tested systematically (README)

## Evidence

- [abstract] metadata cleaning via default_filters: "numerous metadata cleaning and harmonizing filter functions that can easily be stacked to construct a desired pipeline"
- [abstract] key metadata fields cleaned: [evidence span withheld — anchor status: not_found]
- [abstract] rationale for cleaning: "can further contain a significant fraction of inconsistent, wrongly labeled, or incorrect"
- [readme] mgf import entry point: [evidence span withheld — anchor status: external_doc]
- [readme] pipeline class for reusable cleaning workflow: [evidence span withheld — anchor status: external_doc]
- [readme] default filter steps in README example: [evidence span withheld — anchor status: not_found]
