---
name: spectral-peak-mz-filtering
description: Use when raw MS/MS spectra imported from an MGF (or msp/mzML) file must
  be restricted to a defined m/z range and minimum peak count before further processing
  or similarity scoring.
when_to_use_negative:
- The goal is metadata cleaning (compound name, InChI, SMILES, InChIKey, ionmode,
  adduct, charge) rather than peak-level filtering — matchms provides separate metadata
  filters for that.
- Spectra are already pre-filtered or the analysis needs similarity scoring (cosine/modified
  cosine) rather than peak retention.
- Input is not importable mass spectral data (mzML, mzXML, msp, MGF, JSON, USI) but
  an already-processed peak table.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Python package supplying the select_by_mz, select_by_relative_intensity, and
    require_minimum_number_of_peaks filters and the load_from_mgf importer used to
    apply the filter pipeline
  repo: https://github.com/matchms/matchms
- name: pyteomics
  role: Underlying parser that matchms builds its MGF file importer on
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:033066a55e1945fec283e4f9625d56d0edb622d17e243e9c515b2caa3fd13584
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spectral-peak-mz-filtering

## Summary

Reconstructs matchms peak-level filtering as demonstrated in the paper's process_peaks pipeline: retaining peaks in a 0–1000 m/z window via select_by_mz, optionally combined with select_by_relative_intensity (intensity_from=0.001) and require_minimum_number_of_peaks (n_required=10).

## When to use

Use when raw MS/MS spectra imported from an MGF (or msp/mzML) file must be restricted to a defined m/z range and minimum peak count before further processing or similarity scoring.

## When NOT to use

- The goal is metadata cleaning (compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge) rather than peak-level filtering — matchms provides separate metadata filters for that.
- Spectra are already pre-filtered or the analysis needs similarity scoring (cosine/modified cosine) rather than peak retention.
- Input is not importable mass spectral data (mzML, mzXML, msp, MGF, JSON, USI) but an already-processed peak table.

## Inputs

- MGF-format mass spectra file (e.g. pesticides.mgf-style input), importable via matchms.load_from_mgf
- Filter parameter configuration (mz_from=0, mz_to=1000; optionally intensity_from=0.001, n_required=10)

## Outputs

- Retained peak lists (m/z and intensity pairs) per spectrum within the 0–1000 m/z window, e.g. filtered_peaks_mz.csv
- Per-spectrum pass/fail classification against the minimum peak count, e.g. peak_count_filter_report.csv (spectrum id + pass/fail)

## How to apply

Load spectra from the MGF input using matchms.importing.load_from_mgf (MGF import is built on pyteomics), then apply the matchms.filtering select_by_mz filter with mz_from=0 and mz_to=1000, matching the paper's example parameters inside a stacked process_peaks function. Optionally extend the stack with select_by_relative_intensity(intensity_from=0.001) and require_minimum_number_of_peaks(n_required=10), applied in that order since later filters act on peaks retained by earlier ones. Extract the m/z and intensity values of retained peaks and record which peaks fall outside the window. The rationale is that large collected fragmentation datasets can contain a significant fraction of inconsistent or noisy peaks, so a stacked, reproducible filter pipeline yields cleaner spectra before cosine or modified cosine score computation.

## Related tools

- **matchms** (Python package supplying the select_by_mz, select_by_relative_intensity, and require_minimum_number_of_peaks filters and the load_from_mgf importer used to apply the filter pipeline) — https://github.com/matchms/matchms
- **pyteomics** (Underlying parser that matchms builds its MGF file importer on)

## Examples

```
from matchms.filtering import select_by_mz; s = select_by_mz(s, mz_from=0, mz_to=1000)
```

## Evaluation signals

- Every peak written to filtered_peaks_mz.csv has an m/z value within [0, 1000].
- Peaks originally outside the 0–1000 m/z window are absent from the retained list.
- Every spectrum in the input MGF appears exactly once in peak_count_filter_report.csv.
- Each pass/fail classification is consistent with the n_required=10 threshold (pass iff at least 10 peaks retained).
- Filter stack order is preserved: select_by_mz → select_by_relative_intensity → require_minimum_number_of_peaks, as in the paper's process_peaks function.

## Limitations

- Filtering parameters (mz 0–1000, intensity_from=0.001, n_required=10) come from the paper's example; other datasets may need different windows.
- No quantitative acceptance metric is reported in the paper; consistency checks (peak-count threshold, in-range m/z) serve as the acceptance criteria.
- Filters operate on the peak list only — they do not address the inconsistent, wrongly labeled, or incorrect metadata that the paper notes in large collected datasets.

## Evidence

- [abstract] select_by_mz(s, mz_from=0, mz_to=1000): "select_by_mz(s, mz_from=0, mz_to=1000)"
- [abstract] stacked filter pipeline: "numerous metadata cleaning and harmonizing filter functions that can easily be stacked to construct a desired pipeline"
- [abstract] MGF import: "Raw data can be imported from the commonly used formats msp, mzML (Martens et al., 2011), mzXML, MGF"
- [readme] package purpose: [evidence span withheld — anchor status: external_doc]
