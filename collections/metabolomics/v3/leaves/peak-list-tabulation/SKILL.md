---
name: peak-list-tabulation
description: Use when an MGF-imported MS/MS spectrum has been filtered (e.g. select_by_mz
  with mz_from=0, mz_to=1000) and the retained peaks must be written to a tabular
  file like filtered_peaks_mz.csv for inspection or validation.
when_to_use_negative:
- Input spectra have peaks entirely outside the target m/z window, so the tabulation
  would be empty rather than informative
- The goal is similarity scoring (cosine or modified cosine score) rather than peak-level
  filtering and tabulation
- Raw data is in a format other than the supported msp, mzML, mzXML, MGF, JSON, or
  USI formats
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Provides load_from_mgf for MGF import and the select_by_mz filter (applied
    as select_by_mz(s, mz_from=0, mz_to=1000)) whose retained peaks are tabulated
  repo: https://github.com/matchms/matchms
- name: pyteomics
  role: Underlying library on which matchms' MGF file importer is built, used when
    loading the spectra to be filtered
- name: NumPy
  role: Array basis for matchms score and peak handling implementations used to extract
    retained m/z and intensity values
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:e23ab6a831783c699d1a4869fa46905e69c11c10a428e803449a97b1ee5a7846
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
yes_rate: 0.625
n_claims: 8
---

# peak-list-tabulation

## Summary

Tabulates the peak m/z and intensity values retained after applying matchms peak filters such as select_by_mz(mz_from=0, mz_to=1000), as part of a matchms process_peaks pipeline on MGF-imported spectra. Provides a machine-readable record of which peaks survive each filtering window.

## When to use

Use when an MGF-imported MS/MS spectrum has been filtered (e.g. select_by_mz with mz_from=0, mz_to=1000) and the retained peaks must be written to a tabular file like filtered_peaks_mz.csv for inspection or validation.

## When NOT to use

- Input spectra have peaks entirely outside the target m/z window, so the tabulation would be empty rather than informative
- The goal is similarity scoring (cosine or modified cosine score) rather than peak-level filtering and tabulation
- Raw data is in a format other than the supported msp, mzML, mzXML, MGF, JSON, or USI formats

## Inputs

- MGF file of MS/MS mass spectra (e.g. loaded via matchms.load_from_mgf)
- Filter parameters for select_by_mz (mz_from=0, mz_to=1000) or other matchms peak filters

## Outputs

- CSV file of retained peaks with m/z and intensity columns (e.g. filtered_peaks_mz.csv)
- Record of which original peaks were removed as outside the m/z retention window

## How to apply

Load the spectrum from the MGF input file with matchms.load_from_mgf (MGF import in matchms is built on pyteomics). Apply the matchms select_by_mz filter within the process_peaks pipeline using the paper's example parameters mz_from=0 and mz_to=1000, which restricts the spectrum to peaks whose m/z falls within that window. Extract the m/z and intensity values of the retained peaks and record which peaks fall outside the 0–1000 m/z window. Write the retained peak list to a CSV (e.g. filtered_peaks_mz.csv). The same tabulation pattern extends to other matchms filters such as select_by_relative_intensity(intensity_from=0.001) and require_minimum_number_of_peaks(n_required=10), which can be stacked in a pipeline.

## Related tools

- **matchms** (Provides load_from_mgf for MGF import and the select_by_mz filter (applied as select_by_mz(s, mz_from=0, mz_to=1000)) whose retained peaks are tabulated) — https://github.com/matchms/matchms
- **pyteomics** (Underlying library on which matchms' MGF file importer is built, used when loading the spectra to be filtered)
- **NumPy** (Array basis for matchms score and peak handling implementations used to extract retained m/z and intensity values)

## Examples

```
from matchms.importing import load_from_mgf; from matchms.filtering import select_by_mz; s = select_by_mz(load_from_mgf('input.mgf')[0], mz_from=0, mz_to=1000); print(s.peaks.mz, s.peaks.intensities)
```

## Evaluation signals

- Every peak listed in the output CSV has an m/z value within the [0, 1000] window defined by select_by_mz
- Peaks from the original spectrum with m/z outside [0, 1000] are absent from the retained list
- The number of retained peaks is consistent with the count of peaks in the raw MGF spectrum minus those outside the m/z window
- Intensity values in the tabulated output match the source spectrum's peak intensities for the retained m/z values

## Limitations

- Filtering is restricted to the m/z window given by the parameters; peaks are removed permanently, so discarded peaks are only recoverable by re-reading the raw MGF input
- matchms filtering can only be applied after successful import; the MGF importer is built on pyteomics, so malformed MGF records may fail before tabulation
- The select_by_mz window alone does not address the inconsistent, wrongly labeled, or incorrect metadata that matchms notes is common in large collected datasets

## Evidence

- [abstract] select_by_mz(s, mz_from=0, mz_to=1000): "select_by_mz(s, mz_from=0, mz_to=1000)"
- [abstract] Raw data can be imported from the commonly used formats msp, mzML, mzXML, MGF: "Raw data can be imported from the commonly used formats msp, mzML (Martens et al., 2011), mzXML, MGF"
- [abstract] mzML, mzXML, MGF file importers are built on top of pyteomics: "mzML, mzXML, MGF file importers are built on top of pyteomics"
- [readme] importing, processing, cleaning, and comparing mass spectrometry data (MS/MS): [evidence span withheld — anchor status: external_doc]
- [abstract] numerous metadata cleaning and harmonizing filter functions that can easily be stacked: "numerous metadata cleaning and harmonizing filter functions that can easily be stacked to construct a desired pipeline"
