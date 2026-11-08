---
name: spectrum-intensity-normalization
description: Use when spectra loaded from MGF, mzML, mzXML, msp, JSON, or USI carry
  raw (instrument-dependent) peak intensities and you need comparable relative intensities
  before peak filtering or before computing cosine-based similarity scores between
  reference and query spectra.
when_to_use_negative:
- When the spectra already contain relative intensities or have been intensity-normalized
  upstream — re-normalizing would be redundant (though idempotent for max=1 scaling),
  and the effort is better spent on other filters.
- When the analysis metric does not depend on absolute intensity comparisons between
  spectra — e.g. purely m/z-based precursor matching or metadata-only workflows —
  normalization adds no information.
- When you need absolute intensities preserved (e.g. quantification across runs);
  normalize_intensities destroys the original intensity scale, so keep a copy of the
  raw peak data first.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Provides the normalize_intensities filter and default_filters used to rescale
    each spectrum's peak intensities to a maximum of 1 before similarity score calculation
  repo: https://github.com/matchms/matchms
- name: NumPy
  role: Backend on which matchms' score implementations (consumers of normalized intensities)
    are based
- name: pyteomics
  role: Underlying importer for mzML/mzXML/MGF file parsing that produces the spectra
    to be normalized
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:673c7efd2a25dcbf1ede981cb49f5fb6392886a40289103db615ac683026d59d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - NumPy
  - matchms
  - pyteomics
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Spectrum intensity normalization

## Summary

Rescale each MS/MS spectrum's peak intensities so the maximum intensity equals 1, using matchms' normalize_intensities filter so that subsequent similarity scores such as Cosine Greedy compare spectra on a common relative-intensity scale.

## When to use

Use when spectra loaded from MGF, mzML, mzXML, msp, JSON, or USI carry raw (instrument-dependent) peak intensities and you need comparable relative intensities before peak filtering or before computing cosine-based similarity scores between reference and query spectra.

## When NOT to use

- When the spectra already contain relative intensities or have been intensity-normalized upstream — re-normalizing would be redundant (though idempotent for max=1 scaling), and the effort is better spent on other filters.
- When the analysis metric does not depend on absolute intensity comparisons between spectra — e.g. purely m/z-based precursor matching or metadata-only workflows — normalization adds no information.
- When you need absolute intensities preserved (e.g. quantification across runs); normalize_intensities destroys the original intensity scale, so keep a copy of the raw peak data first.

## Inputs

- Spectrum objects parsed from MGF, mzML, mzXML, msp, JSON, or metabolomics-USI sources, each carrying peak lists of (m/z, intensity) pairs plus metadata
- Per-spectrum peak lists such as all_your_spectrums.mgf contents loaded via matchms.load_from_mgf

## Outputs

- Spectrum objects whose peak intensities are rescaled so the maximum peak intensity equals 1
- Normalized per-spectrum peak lists (m/z and intensity pairs) that can be serialized (e.g. normalized_spectra.json) and used as references and queries for similarity scoring

## How to apply

First import the raw spectra with matchms (e.g. matchms.load_from_mgf for an .mgf file) and run each spectrum through matchms.filtering.default_filters to apply baseline metadata and peak cleaning. Then apply matchms.filtering.normalize_intensities to each spectrum; this rescales peak intensities so the maximum peak intensity equals 1 (equivalently, intensities become relative intensities). Apply normalization per spectrum, after default_filters and before any relative-intensity-based peak selection such as select_by_relative_intensity (e.g. intensity_from=0.001) and before score calculations like Cosine Greedy or modified cosine, since those measures assume a comparable intensity scale. In the article's example workflow, the default_filters + normalize_intensities sequence is applied to every spectrum that will serve as either reference or query. Note that matchms >= 0.18.0 also offers a Pipeline class where normalization can be part of a stacked filter pipeline defined per query/reference in a yaml-configured workflow.

## Related tools

- **matchms** (Provides the normalize_intensities filter and default_filters used to rescale each spectrum's peak intensities to a maximum of 1 before similarity score calculation) — https://github.com/matchms/matchms
- **NumPy** (Backend on which matchms' score implementations (consumers of normalized intensities) are based)
- **pyteomics** (Underlying importer for mzML/mzXML/MGF file parsing that produces the spectra to be normalized)

## Examples

```
from matchms.filtering import normalize_intensities, default_filters; spectrum = default_filters(spectrum); spectrum = normalize_intensities(spectrum)
```

## Evaluation signals

- For every normalized spectrum, the maximum peak intensity equals 1 (within numerical tolerance) and the peak list is non-empty.
- The m/z values and the number of peaks are unchanged by normalization; only intensity values differ from the input.
- Peak ordering by m/z is preserved after normalization.
- If select_by_relative_intensity is applied downstream with a threshold such as intensity_from=0.001, the retained peaks fall in the normalized [0,1] intensity range.
- A downstream Cosine Greedy / modified cosine run on normalized reference and query spectra yields scores in the expected range without scale-dependent artifacts.

## Limitations

- Normalization is per-spectrum only; it does not correct instrument- or acquisition-condition differences beyond scaling the intensity axis to a max of 1.
- Spectra whose metadata is inconsistent, wrongly labeled, or incorrect (common in large collected datasets) will still need the extensive metadata cleaning filters — normalization does not address those issues.
- For very noisy or low-peak-count spectra, a max-based rescale can be dominated by a single outlier peak; combining with filters like select_by_relative_intensity or require_minimum_number_of_peaks (e.g. n_required=10) is advisable.

## Evidence

- [abstract] normalize_intensities call in article usage examples: "spectrum = normalize_intensities(spectrum)"
- [abstract] default_filters call in article usage examples: "spectrum = default_filters(spectrum)"
- [readme] README on matchms processing purpose: [evidence span withheld — anchor status: external_doc]
- [abstract] Article finding on workflow from raw spectra to processed spectral data: "It allows to implement and run an easy-to-follow, easy-to-reproduce workflow from raw mass spectra to pre- and post-processed spectral data"
