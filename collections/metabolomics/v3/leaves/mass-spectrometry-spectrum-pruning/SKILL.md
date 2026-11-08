---
name: mass-spectrometry-spectrum-pruning
description: Decision support for cleaning a large, multi-source mass spectral library
  (e.g. .MSP, .MGF, .JSON, .
when_to_use_negative:
- The spectra have already been pruned (e.g. they are the FragHub cleaned output in
  the output directory) and re-filtering them would only remove additional valid peaks
  — the README warns not to delete files in the output directory but also not to re-filter
  cleaned copies
- The library consists of in-silico or GC spectra whose filename does not carry the
  '_insilico' or '_GC' suffix — FragHub cannot classify them correctly, so pruning
  would be applied to spectra placed in the wrong ionization/chromatography/acquisition
  bucket
- The input is not a mass spectral library but raw chromatographic data or a compound-only
  table (no peak lists), since the filters operate exclusively on peak arrays and
  precursor m/z
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FragHub
  role: The integrative workflow that exposes the spectrum-pruning filters via a graphical
    user interface; it parses multi-format OMSL inputs, applies the selected filters
    with user-set parameters, and writes the pruned library to the chosen output format.
  repo: https://github.com/eMetaboHUB/FragHub
- name: Python
  role: Implementation language for FragHub's filter functions (check_minimum_peak_requiered,
    check_minimum_of_high_peaks_requiered, remove_peak_above_precursormz, reduce_peak_list,
    normalize_intensity, keep_mz_in_range, remove_spectrum_under_entropy_score); the
    code is publicly available under a MIT license.
  repo: https://github.com/eMetaboHUB/FragHub
- name: MatchMS
  role: MatchMS satellite package used in the FragHub ecosystem; the spectra-hash
    (SPLASH) integration is planned to be handled directly into FragHub, but MatchMS
    is the spectral-processing library the article cites for the spectral work.
- name: spectra-hash (SPLASH)
  role: Generates SPLASH keys for each spectrum so FragHub can record them in update.json
    and prune/deduplicate only new spectra on subsequent runs.
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:79672b7874ece5458de96318e6e38ba3b0ab33fdd87862523edba2dc3140aa66
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  repo_url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - MatchMS
  - Python
  - spectra-hash (SPLASH)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# mass-spectrometry-spectrum-pruning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A mass spectral library curation skill for pruning low-information or low-quality spectra during OMSL integration: it drops spectra failing a minimum peak-count requirement, keeps only spectra with a minimum number of 'high peaks' (intensity above a percentage of the maximum intensity), and can additionally remove peaks above the precursor m/z, cap peak-list length by intensity, restrict m/z range, normalize intensities, and delete spectra whose entropy score falls below a minimum.

## When to use

Decision support for cleaning a large, multi-source mass spectral library (e.g. .MSP, .MGF, .JSON, .CSV from GNPS, MoNA, MassBank, MS-Dial) before merging it into a unified in-house database: prune spectra that are too sparse, dominated by noise, or inconsistent with the precursor m/z so that downstream library search (MS-Dial, MZmine, Flash Entropy Search) is not degraded.

## When NOT to use

- The spectra have already been pruned (e.g. they are the FragHub cleaned output in the output directory) and re-filtering them would only remove additional valid peaks — the README warns not to delete files in the output directory but also not to re-filter cleaned copies
- The library consists of in-silico or GC spectra whose filename does not carry the '_insilico' or '_GC' suffix — FragHub cannot classify them correctly, so pruning would be applied to spectra placed in the wrong ionization/chromatography/acquisition bucket
- The input is not a mass spectral library but raw chromatographic data or a compound-only table (no peak lists), since the filters operate exclusively on peak arrays and precursor m/z

## Inputs

- Mass spectra from Open Mass Spectra Libraries in FragHub-supported formats: .MSP, .MGF, .JSON, .CSV (and .XML), each with peak lists (m/z, intensity) and metadata such as precursor m/z, adduct, and chemical identifiers (SMILES, InChI, InChIKey)
- Per-filter parameter values selected in the FragHub GUI: minimum number of peaks (n_peaks, e.g. 3), high-peak intensity percentage (intensity_percent, e.g. 5%), minimum number of high peaks (no_peaks, e.g. 2), precursor m/z tolerance (+5 Da), maximum peak count, m/z window, and minimum entropy score
- User-chosen output file format(s) and, optionally, an existing output directory whose update.json file records previously treated SPLASH keys so only new spectra are pruned and processed

## Outputs

- A pruned, deduplicated mass spectral library written in the user-selected format(s) (e.g. filtered_spectra.msp), containing only spectra that passed the selected peak-count and high-peak-intensity criteria
- Peak lists restricted by the applied filters: peaks above precursor m/z + 5 Da removed, peak list capped at the maximum number of most-intense peaks, peaks outside the m/z window removed, intensities normalized to the maximum intensity
- A update.json file recording SPLASH keys of previously processed spectra, enabling incremental re-runs where only new spectra are pruned and added

## How to apply

Parse the input spectra (FragHub handles .MSP, .MGF, .JSON, .CSV, and .XML files) and, in the filter-selection tab of the FragHub GUI, tick the specific pruning filters and set their parameters. First apply check_minimum_peak_requiered with a minimum number of peaks (n_peaks) — FragHub's default stringent criterion keeps spectra only if they contain at least three MS/MS signals; spectra with fewer peaks are ignored. Then apply check_minimum_of_high_peaks_requiered with intensity_percent set to 5% of the maximum intensity and no_peaks set to two, so that a spectrum must show at least two peaks above that relative-intensity threshold to survive. Optionally tighten the pruning further with remove_peak_above_precursormz (drops peaks with m/z greater than the precursor m/z plus 5 Da), reduce_peak_list (retains only the most intense peaks up to a maximum count), keep_mz_in_range (mz_from/mz_to window), normalize_intensity (scales all peaks to the maximum intensity), and remove_spectrum_under_entropy_score (deletes spectra whose entropy score is below the minimum). The rationale is to streamline the peak lists — reducing data size without compromising relevance for analysis — and to remove inconsistent spectra before the library is written to the FragHub output format (e.g. filtered_spectra.msp) for downstream search tools.

## Related tools

- **FragHub** (The integrative workflow that exposes the spectrum-pruning filters via a graphical user interface; it parses multi-format OMSL inputs, applies the selected filters with user-set parameters, and writes the pruned library to the chosen output format.) — https://github.com/eMetaboHUB/FragHub
- **Python** (Implementation language for FragHub's filter functions (check_minimum_peak_requiered, check_minimum_of_high_peaks_requiered, remove_peak_above_precursormz, reduce_peak_list, normalize_intensity, keep_mz_in_range, remove_spectrum_under_entropy_score); the code is publicly available under a MIT license.) — https://github.com/eMetaboHUB/FragHub
- **MatchMS** (MatchMS satellite package used in the FragHub ecosystem; the spectra-hash (SPLASH) integration is planned to be handled directly into FragHub, but MatchMS is the spectral-processing library the article cites for the spectral work.)
- **spectra-hash (SPLASH)** (Generates SPLASH keys for each spectrum so FragHub can record them in update.json and prune/deduplicate only new spectra on subsequent runs.) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
python -c "from fraghub import filters; filters.check_minimum_peak_requiered(peak_array, n_peaks=3); filters.check_minimum_of_high_peaks_requiered(peak_array, intensity_percent=5, no_peaks=2)"
```

## Evaluation signals

- Spectra with fewer than the configured minimum number of peaks (n_peaks, default criterion: three) must be absent from the output library.
- Every surviving spectrum must contain at least the configured number of 'high peaks' (no_peaks, default: two) whose intensity is at least the configured intensity_percent (default: 5%) of the maximum intensity.
- No peak in a surviving spectrum may have an m/z value greater than the precursor m/z plus 5 Da when remove_peak_above_precursormz was enabled.
- Peak lists must be capped at the configured maximum number of peaks, retaining the most intense ones, and intensities must be normalized to the maximum intensity when those filters were applied.
- The update.json file must contain the SPLASH key of every processed spectrum, and the output library's spectrum count should decrease from the input count (e.g. from 794,985 input spectra down to 602,744 after the full FragHub workflow on the GNPS/MoNA/MassBank/MS-Dial data set).

## Limitations

- The default stringent filter (three MS/MS signals and two peaks above 5% relative intensity) is only a threshold-based heuristic; the article notes that collision energy harmonization was not addressed because of their varied and nonstandardized measures (~70 different formats), so spectra pruned by these filters still vary in collision energy metadata.
- All spectra deemed inconsistent — lacking SMILES and InChI, precursor m/z, and adduct information — are removed during FragHub processing, which is a separate, metadata-driven removal on top of the peak-list pruning.
- Fragmentation of the integrated library into ionization modes and chromatography techniques is dynamic and relies on the '_insilico' / '_GC' filename suffixes when not specified inside the file; mislabeled inputs will be pruned into the wrong bucket.
- The article acknowledges that integrating multiple OMSLs introduces redundancy, which may lead to inconsistencies with dot product and reverse dot product scoring that are highly sensitive to fragment number and intensity — pruning thresholds interact with this sensitivity.
- The GitHub discussion notes that the license was changed from MIT to CC-BY-NC 4.0 and that MacOS and Linux support was removed in the 20_12_2024 release, which limits where the Python filter code can be redistributed and run.

## Evidence

- [readme] The three-signal / two-high-peak (>5% relative intensity) criterion is explicitly stated as the second, more stringent filter.: [evidence span withheld — anchor status: not_found]
