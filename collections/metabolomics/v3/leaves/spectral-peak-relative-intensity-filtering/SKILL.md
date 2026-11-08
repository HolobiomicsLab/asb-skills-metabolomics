---
name: spectral-peak-relative-intensity-filtering
description: Use when an MGF, mzML, or msp spectrum contains noisy low-intensity peaks
  that would contaminate downstream similarity scoring, and you need to keep only
  peaks at or above a relative intensity threshold such as 0.001.
when_to_use_negative:
- The spectrum's intensities are raw and not yet normalized — relative-intensity thresholds
  are only meaningful on a 0–1 relative scale, so run normalize_intensities first
  or use an absolute-intensity filter instead
- The goal is metadata cleaning (InChI, SMILES, InChIKey, ionmode, adduct, charge)
  rather than peak filtering — matchms provides separate metadata cleaning and harmonizing
  filters for that
- The spectrum already has 10 or fewer peaks and is subject to require_minimum_number_of_peaks(s,
  n_required=10); aggressive intensity filtering plus the minimum-peak requirement
  may drop the spectrum entirely
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Provides select_by_relative_intensity, select_by_mz, require_minimum_number_of_peaks,
    and normalize_intensities filters that are stacked into the peak-cleaning pipeline
  repo: https://github.com/matchms/matchms
- name: pyteomics
  role: Underlies matchms's MGF (and mzML/mzXML) file importers used to load the raw
    spectrum before filtering
- name: NumPy
  role: Score/array backend matchms builds its spectrum processing on; retained and
    removed peak arrays are NumPy-derived
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:99bc66e4f399781d2535359e01c8d0f7d4c842b163be79b9cd24cfc7b715ac3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - NumPy
  - matchms
  - pyteomics
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spectral-peak-relative-intensity-filtering

## Summary

A matchms peak-filtering skill that removes peaks below a relative-intensity cutoff (e.g. intensity_from=0.001) from MS/MS spectra, typically stacked with m/z-range selection and minimum-peak-count requirements in a spectral cleaning pipeline.

## When to use

Use when an MGF, mzML, or msp spectrum contains noisy low-intensity peaks that would contaminate downstream similarity scoring, and you need to keep only peaks at or above a relative intensity threshold such as 0.001.

## When NOT to use

- The spectrum's intensities are raw and not yet normalized — relative-intensity thresholds are only meaningful on a 0–1 relative scale, so run normalize_intensities first or use an absolute-intensity filter instead
- The goal is metadata cleaning (InChI, SMILES, InChIKey, ionmode, adduct, charge) rather than peak filtering — matchms provides separate metadata cleaning and harmonizing filters for that
- The spectrum already has 10 or fewer peaks and is subject to require_minimum_number_of_peaks(s, n_required=10); aggressive intensity filtering plus the minimum-peak requirement may drop the spectrum entirely

## Inputs

- Raw mass spectrum loaded from an MGF file (e.g. all_your_spectrums.mgf) via matchms.importing.load_from_mgf; mzML, mzXML, msp, JSON, and USI imports are also supported by matchms
- Peak lists with m/z and intensity arrays on a normalized (relative) intensity scale

## Outputs

- Filtered spectrum retaining only peaks with relative intensity >= intensity_from (0.001 in the paper's example)
- Two peak lists written to filtered_peaks.tsv: retained peaks (from the processed spectrum) and removed peaks (from the raw spectrum) for inspection

## How to apply

Load the raw spectrum from the MGF file using matchms.importing.load_from_mgf (the paper's example file is all_your_spectrums.mgf). Apply matchms.filtering.select_by_relative_intensity with intensity_from=0.001 so only peaks with relative intensity >= 0.001 are retained; relative intensities assume the spectrum's intensities are on a normalized 0–1 scale, so normalize_intensities should run in the same pipeline (the paper's example stacks select_by_mz(s, mz_from=0, mz_to=1000) and require_minimum_number_of_peaks(s, n_required=10) around it). Extract m/z and intensity values of retained peaks from the processed spectrum and the removed peaks from the raw spectrum, then write both lists to filtered_peaks.tsv for inspection. Stack the filters in a pipeline so each spectrum passes through the same reproducible cleaning chain, matching matchms's design of easily stacked filter functions.

## Related tools

- **matchms** (Provides select_by_relative_intensity, select_by_mz, require_minimum_number_of_peaks, and normalize_intensities filters that are stacked into the peak-cleaning pipeline) — https://github.com/matchms/matchms
- **pyteomics** (Underlies matchms's MGF (and mzML/mzXML) file importers used to load the raw spectrum before filtering)
- **NumPy** (Score/array backend matchms builds its spectrum processing on; retained and removed peak arrays are NumPy-derived)

## Examples

```
from matchms.filtering import select_by_relative_intensity; spectrum = select_by_relative_intensity(spectrum, intensity_from=0.001)
```

## Evaluation signals

- Every retained peak's relative intensity must be >= 0.001 (the configured intensity_from); every removed peak must be < 0.001
- The union of retained and removed peaks must exactly reconstruct the raw spectrum's peak set (no peaks lost or duplicated across the two lists in filtered_peaks.tsv)
- All retained peaks should also fall within the m/z window of the stacked select_by_mz(s, mz_from=0, mz_to=1000) filter
- If require_minimum_number_of_peaks(s, n_required=10) is stacked, the post-filter spectrum must still contain at least 10 peaks or it is flagged/emptied
- The number of peaks in the processed spectrum must be less than or equal to the raw spectrum's peak count

## Limitations

- The cutoff value (0.001) is dataset- and instrument-dependent; the paper's example thresholds (intensity_from=0.001, mz_to=1000, n_required=10) are illustrative defaults, not universal constants
- Large collected fragmentation mass spectrometry datasets can contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata, so peak filtering alone does not guarantee dataset quality
- Relative-intensity filtering presumes intensities have been normalized (e.g. via normalize_intensities); applying it to raw intensities yields incorrect retention decisions

## Evidence

- [abstract] select_by_relative_intensity removes low relative-intensity peaks: "select_by_relative_intensity(s, intensity_from=0.001)"
- [abstract] matchms filters are stacked into a reproducible cleaning pipeline: "numerous metadata cleaning and harmonizing filter functions that can easily be stacked to construct a desired pipeline"
- [abstract] Companion filters define the stacked pipeline context: "select_by_mz(s, mz_from=0, mz_to=1000)"
- [readme] matchms performs basic peak filtering as part of data integrity checks: [evidence span withheld — anchor status: external_doc]
- [abstract] Minimum peak count requirement companion filter: "require_minimum_number_of_peaks(s, n_required=10)"
