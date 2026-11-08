---
name: matchms-filter-pipeline-construction
description: Use when raw MS/MS spectra (e.g. from an MGF file) must be cleaned before
  scoring or reporting — filtering peaks by m/z range, relative intensity, and minimum
  peak count, or harmonizing metadata like ionmode and adduct.
when_to_use_negative:
- Input is already a fully processed/filtered feature table or cleaned spectral library
  — this skill operates on raw or minimally processed spectra containers, not tabular
  feature matrices.
- The goal is similarity scoring itself (cosine/modified cosine) rather than pre-processing;
  filtering here is a prerequisite step, not the scoring workflow.
- Peak-selection thresholds would discard scientifically relevant peaks — e.g. spectra
  with abundant low-intensity fragments or m/z values above 1000 that the select_by_mz(mz_to=1000)
  window would remove.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Provides the filter functions (select_by_mz, select_by_relative_intensity,
    require_minimum_number_of_peaks, normalize_intensities, default_filters) and the
    Pipeline class that stacks them into a workflow on imported spectra
  repo: https://github.com/matchms/matchms
- name: pyteomics
  role: Underlies the MGF/mzML/mzXML file importers that load the spectra the filter
    pipeline consumes
- name: pymzml
  role: Alternative parser for metadata parsing of imported spectra prior to filtering
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:d7f23bf4e05851cd57afb6ade72f1f5a74a66de10035d8458ee32977e86a670e
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# matchms-filter-pipeline-construction

## Summary

Constructs a stacked spectral filtering pipeline in matchms that combines peak-level filters (select_by_mz, select_by_relative_intensity, require_minimum_number_of_peaks) with metadata and default filters, applied to MS/MS spectra imported from MGF (or msp/mzML/mzXML/JSON/USI) files.

## When to use

Use when raw MS/MS spectra (e.g. from an MGF file) must be cleaned before scoring or reporting — filtering peaks by m/z range, relative intensity, and minimum peak count, or harmonizing metadata like ionmode and adduct.

## When NOT to use

- Input is already a fully processed/filtered feature table or cleaned spectral library — this skill operates on raw or minimally processed spectra containers, not tabular feature matrices.
- The goal is similarity scoring itself (cosine/modified cosine) rather than pre-processing; filtering here is a prerequisite step, not the scoring workflow.
- Peak-selection thresholds would discard scientifically relevant peaks — e.g. spectra with abundant low-intensity fragments or m/z values above 1000 that the select_by_mz(mz_to=1000) window would remove.

## Inputs

- Raw MS/MS spectra in MGF format (also msp, mzML, mzXML, JSON, or metabolomics-USI), e.g. tests/testdata/pesticides.mgf
- Filter configuration parameters: mz_from=0, mz_to=1000, intensity_from=0.001, n_required=10

## Outputs

- Filtered Spectrum objects retaining only peaks within the m/z window and above the relative-intensity cutoff, and only if they meet the minimum peak count
- Per-spectrum pass/fail classifications (e.g. spectrum-id and pass/fail written to peak_count_filter_report.csv)
- Optionally a yaml workflow file (my_config_file.yaml) recording the filter stack for reuse

## How to apply

Import spectra with matchms.importing.load_from_mgf (matchms also supports msp, mzML, mzXML, JSON, and USI). Stack filter functions in a deliberate order: first run default_filters, then restrict peaks to the m/z window with select_by_mz(mz_from=0, mz_to=1000), remove low-intensity noise with select_by_relative_intensity(intensity_from=0.001), and finally enforce quality with require_minimum_number_of_peaks(n_required=10); normalize_intensities can be added when downstream scores require comparable intensity scales. The rationale for stacking is that matchms is designed so 'numerous metadata cleaning and harmonizing filter functions can easily be stacked to construct a desired pipeline', yielding reproducible pre-processing before similarity computations such as cosine or modified cosine scores. Decision point: spectra retaining fewer than n_required peaks after the earlier steps fail the filter (e.g. classify as 'fail' in a pass/fail report), so threshold choice (here 10 peaks, intensity_from=0.001) directly controls which spectra survive. Newer matchms versions (>= 0.18.0) also offer a Pipeline class and yaml-configurable workflows where these filters appear as query_filters/reference_filters lists such as DEFAULT_FILTERS or LIBRARY_CLEANING.

## Related tools

- **matchms** (Provides the filter functions (select_by_mz, select_by_relative_intensity, require_minimum_number_of_peaks, normalize_intensities, default_filters) and the Pipeline class that stacks them into a workflow on imported spectra) — https://github.com/matchms/matchms
- **pyteomics** (Underlies the MGF/mzML/mzXML file importers that load the spectra the filter pipeline consumes)
- **pymzml** (Alternative parser for metadata parsing of imported spectra prior to filtering)

## Examples

```
from matchms.importing import load_from_mgf; from matchms.filtering import select_by_mz, select_by_relative_intensity, require_minimum_number_of_peaks; spectra = [require_minimum_number_of_peaks(select_by_relative_intensity(select_by_mz(s, 0, 1000), 0.001), 10) for s in load_from_mgf('pesticides.mgf')]
```

## Evaluation signals

- Every spectrum in the input file appears exactly once in the pass/fail report (e.g. peak_count_filter_report.csv) — no duplicates, no missing IDs.
- Each pass/fail classification is consistent with the 10-peak minimum: a spectrum passes only if it retains at least n_required=10 peaks after the select_by_mz and select_by_relative_intensity steps.
- No peak in any output spectrum lies outside the selected m/z window (0–1000) or below the relative-intensity cutoff (0.001).
- Re-running the stacked pipeline (or replaying the exported yaml workflow via Pipeline) reproduces identical filtered spectra, demonstrating the desired reproducible workflow.

## Limitations

- Large collected fragmentation mass spectrometry datasets can contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata, so peak filtering alone does not guarantee metadata correctness.
- Fixed thresholds (mz_to=1000, intensity_from=0.001, n_required=10) are example values from the tutorial workflow; other datasets may need different windows, and no quantitative acceptance metric is reported for the pass/fail criterion.
- matchms version matters: the Pipeline class and yaml-based workflows only exist in matchms >= 0.18.0; earlier versions use stacked function calls directly.

## Evidence

- [abstract] Stacked filters example: "select_by_mz(s, mz_from=0, mz_to=1000)"
- [abstract] Minimum peak count filter: "require_minimum_number_of_peaks(s, n_required=10)"
- [abstract] Pipeline design rationale: "numerous metadata cleaning and harmonizing filter functions that can easily be stacked to construct a desired pipeline"
- [abstract] Reproducible workflow claim: "It allows to implement and run an easy-to-follow, easy-to-reproduce workflow from raw mass spectra to pre- and post-processed spectral data"
- [readme] Pipeline class availability: [evidence span withheld — anchor status: external_doc]
