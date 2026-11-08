---
name: msms-peak-list-extraction
description: Use when a raw MS/MS spectrum loaded from an MGF file must have low relative-intensity
  peaks removed via matchms, and you need the retained and filtered-out peak lists
  for inspection.
when_to_use_negative:
- When the goal is metadata cleaning (compound name, InChI, SMILES, InChIKey, ionmode,
  adduct, charge) rather than peak-list filtering — use the metadata cleaning/harmonizing
  filters instead
- When spectra must be similarity-scored directly (cosine score, modified cosine score);
  this skill only produces the cleaned peak lists, not scores
- When the input is not raw mass spectral data (e.g. already a processed feature table)
  since matchms expects raw spectra from msp, mzML, mzXML, MGF, JSON, or USI
tools:
- name: matchms
  role: Provides load_from_mgf for MGF import and the select_by_relative_intensity
    filter (with intensity_from parameter) used to retain/remove peaks by relative
    intensity
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:d984996dbfc8e01281a48f6edb2ef691e05d60b7bfe1f4317a794243ab71072a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - matchms
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# msms-peak-list-extraction

## Summary

Extracts the m/z and intensity values of peaks retained versus removed by the matchms select_by_relative_intensity filter from a raw MS/MS spectrum, so that the effect of the intensity threshold (e.g. intensity_from=0.001) can be inspected peak-by-peak.

## When to use

Use when a raw MS/MS spectrum loaded from an MGF file must have low relative-intensity peaks removed via matchms, and you need the retained and filtered-out peak lists for inspection.

## When NOT to use

- When the goal is metadata cleaning (compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge) rather than peak-list filtering — use the metadata cleaning/harmonizing filters instead
- When spectra must be similarity-scored directly (cosine score, modified cosine score); this skill only produces the cleaned peak lists, not scores
- When the input is not raw mass spectral data (e.g. already a processed feature table) since matchms expects raw spectra from msp, mzML, mzXML, MGF, JSON, or USI

## Inputs

- Raw MS/MS spectrum loaded from an MGF file (e.g. all_your_spectrums.mgf) via load_from_mgf
- Relative-intensity threshold parameter, e.g. intensity_from=0.001

## Outputs

- List of retained peaks (m/z and intensity values) from the processed spectrum
- List of peaks removed by the relative-intensity filter, taken from the raw spectrum
- filtered_peaks.tsv containing both retained and removed peak lists for inspection

## How to apply

Load the raw spectrum from the MGF file with matchms.importing.load_from_mgf (the paper's example uses all_your_spectrums.mgf), then apply matchms.filtering.select_by_relative_intensity with intensity_from=0.001 so only peaks with relative intensity >= 0.001 are kept. This filter sits in the paper's peak-processing pipeline alongside select_by_mz(s, mz_from=0, mz_to=1000) and require_minimum_number_of_peaks(s, n_required=10). Extract the m/z and intensity values of the retained peaks from the processed spectrum, and collect the peaks removed by the filter from the raw (pre-filter) spectrum. Write both peak lists (retained and removed) to filtered_peaks.tsv for inspection. Rationale: large collected fragmentation datasets contain noise peaks and inconsistent metadata, so matchms stacks filter functions into a pipeline to clean spectra before similarity scoring.

## Related tools

- **matchms** (Provides load_from_mgf for MGF import and the select_by_relative_intensity filter (with intensity_from parameter) used to retain/remove peaks by relative intensity) — https://github.com/matchms/matchms

## Examples

```
from matchms.importing import load_from_mgf; from matchms.filtering import select_by_relative_intensity; s = select_by_relative_intensity(load_from_mgf("all_your_spectrums.mgf")[0], intensity_from=0.001)
```

## Evaluation signals

- Number of retained peaks equals the number of raw-spectrum peaks with relative intensity >= 0.001 (here intensity_from=0.001)
- The removed-peak list is the exact complement of the retained-peak list with respect to the raw spectrum's peak set
- If combined with select_by_mz, every retained peak's m/z falls within mz_from=0 to mz_to=1000
- The processed spectrum still satisfies require_minimum_number_of_peaks(s, n_required=10) after filtering
- Retained peaks' intensities and m/z values are unchanged from the raw spectrum (filtering removes peaks but does not alter surviving peak values)

## Limitations

- The intensity_from threshold is spectrum- and pipeline-dependent; the relative-intensity scale assumes intensities have been normalized (the paper shows normalize_intensities as a separate filter step)
- Filtering can remove peaks needed downstream; the pipeline also enforces a minimum peak count (n_required=10), so over-filtering can disqualify a spectrum entirely
- Only formats matchms supports (msp, mzML, mzXML, MGF, JSON, USI) can be used as input; other raw formats require conversion first

## Evidence

- [abstract] select_by_relative_intensity(s, intensity_from=0.001): "select_by_relative_intensity(s, intensity_from=0.001)"
- [abstract] mgf import: "Raw data can be imported from the commonly used formats msp, mzML (Martens et al., 2011), mzXML, MGF"
- [readme] matchms purpose: [evidence span withheld — anchor status: external_doc]
- [abstract] pipeline stacking: "numerous metadata cleaning and harmonizing filter functions that can easily be stacked to construct a desired pipeline"
