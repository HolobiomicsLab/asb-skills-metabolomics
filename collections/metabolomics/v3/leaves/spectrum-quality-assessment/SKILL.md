---
name: spectrum-quality-assessment
description: Use when an MGF/mzML/msp MS/MS collection contains sparse or noisy spectra
  — e.g. fewer than ~10 peaks above a relative-intensity threshold — and you must
  flag or drop low-quality spectra before library cleaning or spectral similarity
  comparison.
when_to_use_negative:
- The input is already a cleaned, filtered feature/spectral library — stacking require_minimum_number_of_peaks
  again would only remove legitimately sparse spectra without adding information.
- The goal is metadata-only harmonization (ionmode, adduct, InChIKey correction) with
  no need to assess peak count/intensity quality — a metadata-cleaning pipeline without
  peak filters is the right fit.
- Spectra are raw profile-mode or centroided precursor-only data where peak-count
  thresholds are not meaningful — quality assessment by peak counts presupposes processed
  MS2 peak lists.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Primary library providing the filtering functions (require_minimum_number_of_peaks,
    select_by_mz, select_by_relative_intensity, normalize_intensities, default_filters)
    and the Pipeline class for stacking them into reproducible workflows
  repo: https://github.com/matchms/matchms
- name: pyteomics
  role: Backend importer matchms uses to parse mzML, mzXML, and MGF input files before
    filtering
- name: NumPy
  role: Basis for matchms's fast score implementations, enabling filtered high-quality
    spectra to be compared at scale
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:b96105a26a93333d433d8d50d3d46ac09d7f9f4026f1aac2e835d2d62f8e4241
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Spectrum quality assessment

## Summary

Assess and enforce per-spectrum quality in MS/MS libraries using matchms filter steps — stacking peak-count and intensity/mz selection filters such as require_minimum_number_of_peaks (n_required=10), select_by_mz (mz_from=0, mz_to=1000), and select_by_relative_intensity (intensity_from=0.001) to classify spectra as pass/fail and remove low-quality or sparse spectra before similarity scoring.

## When to use

Use when an MGF/mzML/msp MS/MS collection contains sparse or noisy spectra — e.g. fewer than ~10 peaks above a relative-intensity threshold — and you must flag or drop low-quality spectra before library cleaning or spectral similarity comparison.

## When NOT to use

- The input is already a cleaned, filtered feature/spectral library — stacking require_minimum_number_of_peaks again would only remove legitimately sparse spectra without adding information.
- The goal is metadata-only harmonization (ionmode, adduct, InChIKey correction) with no need to assess peak count/intensity quality — a metadata-cleaning pipeline without peak filters is the right fit.
- Spectra are raw profile-mode or centroided precursor-only data where peak-count thresholds are not meaningful — quality assessment by peak counts presupposes processed MS2 peak lists.

## Inputs

- MS/MS spectra files in commonly used formats: msp, mzML, mzXML, MGF, JSON, or metabolomics-USI (e.g. tests/testdata/pesticides.mgf in the matchms repo)
- Per-spectrum peak lists and metadata (ionmode, adduct, charge, InChI/SMILES/InChIKey) parsed by matchms importers (built on pyteomics)

## Outputs

- Filtered Spectrum objects whose peak lists satisfy the stacked criteria (mz range 0–1000, relative intensity ≥ 0.001, ≥ 10 required peaks)
- A per-spectrum pass/fail quality classification (e.g. CSV report mapping spectrum-id to pass/fail) usable for library cleaning or downstream similarity-score quality control

## How to apply

Load spectra with matchms (load_from_mgf for MGF, or pyteomics-backed importers for mzML/mzXML), then stack matchms.filtering functions in the recommended order: metadata default_filters first, normalize_intensities so relative intensities are comparable across spectra, select_by_mz (mz_from=0, mz_to=1000) to restrict to the instrument's detectable range, select_by_relative_intensity (intensity_from=0.001) to cut noise, and finally require_minimum_number_of_peaks (n_required=10) — spectra retaining fewer than the required peaks should be rejected or flagged. The rationale is that large collected datasets contain a significant fraction of inconsistent or incorrect entries, and matchms's metadata-cleaning plus peak filters ensure spectra entering cosine/modified-cosine scoring are of comparable quality. In matchms >= 0.18, express the same stack through the Pipeline class or the LIBRARY_CLEANING / DEFAULT_FILTERS pipelines, optionally recording per-spectrum pass/fail to a report (e.g. peak_count_filter_report.csv with spectrum-id and classification) for provenance. Judge correctness by confirming every input spectrum is represented exactly once in the report and each classification is consistent with the threshold actually applied (e.g. ≥10 retained peaks = pass).

## Related tools

- **matchms** (Primary library providing the filtering functions (require_minimum_number_of_peaks, select_by_mz, select_by_relative_intensity, normalize_intensities, default_filters) and the Pipeline class for stacking them into reproducible workflows) — https://github.com/matchms/matchms
- **pyteomics** (Backend importer matchms uses to parse mzML, mzXML, and MGF input files before filtering)
- **NumPy** (Basis for matchms's fast score implementations, enabling filtered high-quality spectra to be compared at scale)

## Examples

```
from matchms import Spectrum; from matchms.filtering import require_minimum_number_of_peaks; filtered = require_minimum_number_of_peaks(spectrum, n_required=10)
```

## Evaluation signals

- Every spectrum from the input file appears exactly once in the per-spectrum pass/fail report (e.g. peak_count_filter_report.csv)
- Each classification is consistent with the applied threshold — spectra retaining ≥ 10 peaks after select_by_mz/select_by_relative_intensity are labeled pass, others fail
- Peak intensities are normalized before relative-intensity filtering so the 0.001 cutoff is comparable across spectra from different instruments/conditions
- The stacked filter pipeline is reproducible — with matchms >= 0.18 the same workflow is captured in a yaml workflow file via create_workflow and rerun by Pipeline.run
- After require_minimum_number_of_peaks, no remaining spectrum violates the n_required minimum

## Limitations

- No quantitative quality metric or acceptance threshold is prescribed by the paper beyond the example's n_required=10 — the numeric cutoff is an example value and practitioners must justify their own threshold.
- Large collected datasets contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata, so peak-count filtering alone is insufficient — it must be paired with metadata cleaning filters (compound name, InChI/SMILES/InChIKey, ionmode, adduct, charge).
- Filter results depend on filter order (e.g. normalize_intensities must precede relative-intensity selection); misordered stacks change which peaks survive the 0.001 intensity cutoff.
- Similarity scores downstream (cosine, modified cosine) are only as good as the filtered spectra — scores computed on unfiltered, low-quality spectra will be misleading.

## Evidence

- [abstract] Peak-count filtering with example thresholds: "select_by_mz(s, mz_from=0, mz_to=1000)"
- [abstract] Relative intensity cutoff: "select_by_relative_intensity(s, intensity_from=0.001)"
- [abstract] Minimum peak requirement: "require_minimum_number_of_peaks(s, n_required=10)"
- [abstract] Rationale: metadata/quality issues in collected MS datasets: "can further contain a significant fraction of inconsistent, wrongly labeled, or incorrect"
- [abstract] Normalization before intensity filtering: "spectrum = normalize_intensities(spectrum)"
- [readme] Reproducibility via Pipeline class (README): [evidence span withheld — anchor status: external_doc]
- [readme] Default filter pipelines (README): [evidence span withheld — anchor status: external_doc]
- [abstract] Tool scope statement: "matchms is an open-source Python package to import, process, clean, and compare mass spectrometry data (MS/MS)"
