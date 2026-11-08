---
name: spectrum-peak-threshold-inspection
description: Use when raw MS/MS spectra (MGF, mzML, msp, JSON) contain low-abundance
  noise peaks and you need to reproduce or audit exactly which peaks a relative-intensity
  cutoff removes before similarity scoring.
when_to_use_negative:
- The goal is to compute similarity scores (cosine or modified cosine) between already-cleaned
  spectra rather than inspect peak-level filtering
- The task is metadata cleaning or validation (compound name, InChI, SMILES, InChIKey,
  ionmode, adduct, charge) rather than peak filtering
- The spectra have already been processed by this filter and only downstream pipeline
  steps (e.g. score computations via the Pipeline class) remain
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Provides load_from_mgf for importing the spectrum and the select_by_relative_intensity
    peak filter (plus select_by_mz and require_minimum_number_of_peaks) that are stacked
    into the filtering pipeline
  repo: https://github.com/matchms/matchms
- name: pyteomics
  role: Underlying importer on which matchms' MGF file loading is built
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:b2c18fe770a6a8a40ae2d0811c2734acc1e536c136b0dc2f4cc040015b0c52ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - matchms
  - pyteomics
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# spectrum-peak-threshold-inspection

## Summary

Applies the matchms select_by_relative_intensity peak filter to an MS/MS spectrum (e.g. with intensity_from=0.001) and reconstructs which peaks were retained versus removed, writing both lists to a TSV for inspection.

## When to use

Use when raw MS/MS spectra (MGF, mzML, msp, JSON) contain low-abundance noise peaks and you need to reproduce or audit exactly which peaks a relative-intensity cutoff removes before similarity scoring.

## When NOT to use

- The goal is to compute similarity scores (cosine or modified cosine) between already-cleaned spectra rather than inspect peak-level filtering
- The task is metadata cleaning or validation (compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge) rather than peak filtering
- The spectra have already been processed by this filter and only downstream pipeline steps (e.g. score computations via the Pipeline class) remain

## Inputs

- Raw mass spectrum file in MGF format (e.g. all_your_spectrums.mgf); matchms also imports msp, mzML, mzXML, JSON, and USI
- Filter threshold parameters, e.g. intensity_from=0.001 for select_by_relative_intensity
- Optional companion filter parameters: mz_from=0, mz_to=1000 (select_by_mz) and n_required=10 (require_minimum_number_of_peaks)

## Outputs

- A processed matchms Spectrum object containing only peaks with relative intensity >= the cutoff
- A TSV file (filtered_peaks.tsv) listing the retained peaks (m/z, intensity) and the peaks removed by the filter

## How to apply

Load the raw spectrum from the MGF file using matchms.importing.load_from_mgf (the paper's example file is all_your_spectrums.mgf). Apply matchms.filtering.select_by_relative_intensity to the spectrum with intensity_from=0.001 so that only peaks with relative intensity >= 0.001 are kept; in the paper's example pipeline this filter is stacked with select_by_mz(s, mz_from=0, mz_to=1000) and require_minimum_number_of_peaks(s, n_required=10) in a single filtering pipeline. Extract m/z and intensity values of the retained peaks from the processed spectrum and of the removed peaks by diffing against the raw spectrum. Write both peak lists (retained and removed) to filtered_peaks.tsv. The rationale is that large collected fragmentation datasets contain noise and inconsistent peaks, and matchms provides stackable filter functions so the filtering step is explicit and reproducible.

## Related tools

- **matchms** (Provides load_from_mgf for importing the spectrum and the select_by_relative_intensity peak filter (plus select_by_mz and require_minimum_number_of_peaks) that are stacked into the filtering pipeline) — https://github.com/matchms/matchms
- **pyteomics** (Underlying importer on which matchms' MGF file loading is built)

## Examples

```
from matchms.importing import load_from_mgf; from matchms.filtering import select_by_relative_intensity; spectrum = select_by_relative_intensity(load_from_mgf('all_your_spectrums.mgf'), intensity_from=0.001)
```

## Evaluation signals

- Every retained peak in the processed spectrum has relative intensity >= 0.001; any peak below that cutoff appears only in the removed-peaks list
- The union of retained and removed peak lists equals the full set of peaks in the raw spectrum (no peaks lost or duplicated in the diff)
- If select_by_mz was applied in the same pipeline, all retained peaks additionally fall within m/z 0 to 1000
- filtered_peaks.tsv contains both peak lists (retained and removed) with m/z and intensity columns and is non-empty for each input spectrum
- If require_minimum_number_of_peaks(n_required=10) is part of the pipeline, spectra keeping fewer than 10 peaks after filtering are flagged/removed

## Limitations

- The cutoff is on relative intensity, so results depend on how intensities were normalized (e.g. normalize_intensities) before filtering
- The specific values (intensity_from=0.001, mz_to=1000, n_required=10) are the paper's example pipeline parameters, not universal defaults; they must be chosen per dataset
- matchms' strength is the pipeline context; since version >= 0.18.0 typical workflows including filtering and score computations are handled by the Pipeline class with yaml-configurable workflows, so a one-off manual filter invocation may need to be adapted

## Evidence

- [abstract] select_by_relative_intensity(s, intensity_from=0.001): "select_by_relative_intensity(s, intensity_from=0.001)"
- [abstract] numerous metadata cleaning and harmonizing filter functions that can easily be stacked to construct a desired pipeline: "numerous metadata cleaning and harmonizing filter functions that can easily be stacked to construct a desired pipeline"
- [readme] Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS).: [evidence span withheld — anchor status: external_doc]
- [readme] Matchms offers an array of tools for metadata cleaning and validation, alongside basic peak filtering, to ensure data accuracy and integrity.: [evidence span withheld — anchor status: external_doc]
