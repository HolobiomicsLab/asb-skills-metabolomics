---
name: spectrum-intensity-normalization
description: Use when raw library spectra (e.g. MGF from GNPS or matchms-loaded entries)
  must be cleaned before training or scoring — e.g. noisy low-intensity peaks and
  unbounded peak counts would otherwise dominate model input.
when_to_use_negative:
- When spectra have already been preprocessed (e.g. the deposited cleaned_spectra.mgf
  from Zenodo record 4699300) — re-applying the 0.1% / 1000-peak / square-root chain
  would double-filter and distort intensities.
- When the input is not raw peak lists but already-binned spectra or embedding vectors
  — this skill operates on per-spectrum peak intensity arrays, not on derived features.
- When the downstream model expects raw intensities (e.g. some non-MS2DeepScore tools)
  — the square-root transform here is specific to the MS2DeepScore preprocessing pipeline.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Python library used to load, filter, and normalize the GNPS MS/MS spectra
    (peak list handling, intensity transforms, MGF I/O)
  repo: https://github.com/matchms/matchms
- name: MS2DeepScore
  role: Downstream Siamese neural network that consumes the cleaned spectra after
    intensity normalization and 10,000-bin binning
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:3d17714c89cd79c920b9795e6da962af04d074ef6c7b2e919b72df4f51105cc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spectrum-intensity-normalization

## Summary

Cleans MS/MS spectra from GNPS (or similar) libraries by removing low-intensity peaks (<0.1% of maximum peak intensity), capping at the 1000 highest-intensity peaks, and applying a square-root intensity transform, as performed in the MS2DeepScore data-preparation workflow.

## When to use

Use when raw library spectra (e.g. MGF from GNPS or matchms-loaded entries) must be cleaned before training or scoring — e.g. noisy low-intensity peaks and unbounded peak counts would otherwise dominate model input.

## When NOT to use

- When spectra have already been preprocessed (e.g. the deposited cleaned_spectra.mgf from Zenodo record 4699300) — re-applying the 0.1% / 1000-peak / square-root chain would double-filter and distort intensities.
- When the input is not raw peak lists but already-binned spectra or embedding vectors — this skill operates on per-spectrum peak intensity arrays, not on derived features.
- When the downstream model expects raw intensities (e.g. some non-MS2DeepScore tools) — the square-root transform here is specific to the MS2DeepScore preprocessing pipeline.

## Inputs

- annotated MS/MS spectra in MGF (or other matchms-supported) format, e.g. GNPS library entries with InChIKey and SMILES and/or InChI annotations
- matchms Spectrum objects (positive-mode spectra with >=5 peaks between 10 and 1000 m/z)

## Outputs

- cleaned_spectra.mgf — MGF file with peaks filtered (<0.1% max intensity), capped at 1000 peaks, and square-root-transformed intensities, metadata preserved
- matchms Spectrum objects ready for the 10,000-bin (10–1000 m/z) binning step

## How to apply

Load the annotated spectra (e.g. matchms-readable MGF from the GNPS-derived dataset of 210,407 spectra, Zenodo record 4699300) and iterate over each spectrum. For each one, compute the maximum peak intensity and remove any peak whose relative intensity is below 0.1% of that maximum, then keep at most the 1000 highest-intensity peaks to bound the number of bins fed to downstream binning (10,000 equally-sized bins from 10 to 1000 m/z). Finally apply a square-root transformation to the remaining peak intensities so that low- and high-intensity peaks are better balanced for the Siamese network. Retain the annotated metadata (InChIKey, SMILES and/or InChI) alongside the cleaned peaks so the cleaned spectra remain traceable to their molecular annotations. The result, e.g. written as cleaned_spectra.mgf, is the direct input to spectrum binning and subsequent MS2DeepScore training.

## Related tools

- **matchms** (Python library used to load, filter, and normalize the GNPS MS/MS spectra (peak list handling, intensity transforms, MGF I/O)) — https://github.com/matchms/matchms
- **MS2DeepScore** (Downstream Siamese neural network that consumes the cleaned spectra after intensity normalization and 10,000-bin binning) — https://github.com/matchms/ms2deepscore

## Examples

```
from matchms.importing import load_from_mgf; from matchms.filtering import normalize_intensities, select_by_relative_intensity, select_by_intensity, reduce_to_number_of_peaks
spectra = list(load_from_mgf('gnps_library.mgf'))
spectra = [select_by_intensity(select_by_relative_intensity(reduce_to_number_of_peaks(normalize_intensities(s), n_max=1000), intensity_from=0.001), 10) for s in spectra]
```

## Evaluation signals

- No remaining peak in a cleaned spectrum has an intensity below 0.1% of that spectrum's maximum peak intensity.
- Every output spectrum contains at most 1000 peaks (and, following the curation pipeline, at least 5 peaks between 10 and 1000 m/z).
- Peak intensities in the output are square-root transformed: sorting order is unchanged but magnitude differences are compressed compared to the raw values.
- Metadata fields (InChIKey, SMILES, InChI, precursor m/z, ion mode) are unchanged for each spectrum after cleaning.
- Downstream training reaches the expected scale: the cleaned set of 210,407 spectra (184,698 annotated) still supports a training/validation/test split of 14,062/500/500 unique InChIKeys.

## Limitations

- Filtering thresholds (0.1% relative intensity, 1000-peak cap, square-root transform) were tuned for the GNPS-derived dataset; other instruments or library types may need different cutoffs.
- The cleaned dataset retains instrument and quality heterogeneity ('spectra come from different instrumentation types, are of varying quality'), which limits downstream prediction accuracy.
- The square-root transform means output intensities are no longer linear in the raw signal — downstream steps must not interpret them as physical counts.

## Evidence

- [discussion] Intensity cap and relative-intensity threshold: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [discussion] Square-root intensity transform: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [discussion] Cleaned dataset availability and scale: "The full cleaned dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) can be found on zenodo: https://zenodo.org/record/4699300."
- [readme] Downstream model expects cleaned spectra as input: [evidence span withheld — anchor status: not_found]
