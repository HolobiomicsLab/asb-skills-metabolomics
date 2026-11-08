---
name: msms-peak-list-scoring
description: Use when open mass spectral library records (.MSP/.MGF/.JSON/.CSV/.XML)
  contain inconsistent, oversized, or low-information peak lists that must be cleaned
  and scored before library integration or MS/MS annotation in MS-Dial.
when_to_use_negative:
- Input is already a clean, deduplicated, polarity-split library produced by a previous
  FragHub run — re-running would reset or duplicate processing unless update.json
  SPLASH keys are preserved.
- Spectra are in-silico or GC-only and not labelled in the filename or metadata —
  FragHub requires the '_insilico' or '_GC' filename suffix to classify them correctly.
- The goal is to filter raw LC–MS feature detection output (e.g. blank-removal, ghost-peak
  removal); that is handled upstream by MS-CleanR (blank ratio 0.8, RSD 40%, relative
  mass defect 50–3500 ppm), not by FragHub peak-list filters.
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FragHub
  role: Implements the peak-list filters (check_minimum_peak_requiered, remove_peak_above_precursormz,
    reduce_peak_list, normalize_intensity, keep_mz_in_range, check_minimum_of_high_peaks_requiered,
    remove_spectrum_under_entropy_score) and exports harmonized libraries
  repo: github.com/eMetaboHUB/FragHub
- name: MS-Dial
  role: Consumes the filtered .MSP libraries for MS/MS-based spectral annotation of
    aligned feature lists (default MS-Dial parameters used in the 435-feature benchmark)
- name: spectra-hash (SPLASH)
  role: Provides the SPLASH spectral hash keys used by FragHub to deduplicate processed
    spectra against update.json
  repo: github.com/berlinguyinca/spectra-hash
- name: MatchMS
  role: Python package in the FragHub ecosystem for spectrum processing referenced
    by the article
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:dd7a80053ff74ec655b8a1eba773fa8c3093edb944e0503ae94dba965a73fca4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - MS-CleanR
  - MS-Dial
  - MatchMS
  - spectra-hash (SPLASH)
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# msms-peak-list-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Score and filter MS/MS peak lists from open mass spectral libraries before integration or spectral search, using FragHub's spectrum-quality filters (minimum peak counts, precursor m/z clipping, intensity normalization, m/z range, and entropy score) so that only consistent, searchable spectra reach downstream annotation tools like MS-Dial.

## When to use

Use when open mass spectral library records (.MSP/.MGF/.JSON/.CSV/.XML) contain inconsistent, oversized, or low-information peak lists that must be cleaned and scored before library integration or MS/MS annotation in MS-Dial.

## When NOT to use

- Input is already a clean, deduplicated, polarity-split library produced by a previous FragHub run — re-running would reset or duplicate processing unless update.json SPLASH keys are preserved.
- Spectra are in-silico or GC-only and not labelled in the filename or metadata — FragHub requires the '_insilico' or '_GC' filename suffix to classify them correctly.
- The goal is to filter raw LC–MS feature detection output (e.g. blank-removal, ghost-peak removal); that is handled upstream by MS-CleanR (blank ratio 0.8, RSD 40%, relative mass defect 50–3500 ppm), not by FragHub peak-list filters.

## Inputs

- Open mass spectral library files in .MSP, .MGF, .JSON, .CSV, or .XML format (e.g. GNPS, MoNA, MassBank, MS-Dial-VS17)
- Aligned feature lists / MS/MS deconvolution output from MS-Dial v5.23 (e.g. the 435-feature MS-CleanR-filtered A. thaliana PI-mode list)
- User-selected filter parameters: minimum peaks, max peaks, mz_from/mz_to, intensity_percent, no_peaks, minimum entropy score

## Outputs

- Consolidated, format-unified spectral libraries split by polarity (positive/negative), chromatographic mode (LC/GC), and experimental/in-silico acquisition, in .MSP, .MGF, .JSON, .CSV, .XML compatible with MS-Dial, MZmine, Flash Entropy Search
- update.json containing SPLASH keys of previously processed spectra for incremental updates
- Per-library annotation counts (e.g. FragHub = 102, MassBank = 41 annotated features) when coupled with MS-Dial search

## How to apply

Load each library record (e.g. GNPS .MGF, MoNA .MSP, MassBank, MS-Dial-VS17 .MSP) through FragHub, which parses the multiple file formats and assigns a SPLASH key to every spectrum so duplicates are caught against the update.json repository. Then apply the configurable peak-list filters via the GUI: check_minimum_peak_requiered to drop spectra below a minimum number of peaks; remove_peak_above_precursormz to clip peaks above precursor m/z + 5 Da; reduce_peak_list to cap the peak count at a specified maximum, retaining the most intense peaks; normalize_intensity to scale intensities to the maximum; keep_mz_in_range to bound m/z between mz_from and mz_to; check_minimum_of_high_peaks_requiered to require a minimum number of peaks above a given intensity_percent; and remove_spectrum_under_entropy_score to discard spectra whose entropy score falls below the threshold. Spectra lacking SMILES and InChI, precursor m/z, or adduct information are removed outright as inconsistent. The rationale, per the Nicolle et al. A. thaliana benchmark, is that default filtering on the consolidated FragHub-processed libraries allows MS-Dial v5.23 to annotate 102 of 435 MS-CleanR-filtered PI-mode features (24%), versus only 41 with MassBank alone.

## Related tools

- **FragHub** (Implements the peak-list filters (check_minimum_peak_requiered, remove_peak_above_precursormz, reduce_peak_list, normalize_intensity, keep_mz_in_range, check_minimum_of_high_peaks_requiered, remove_spectrum_under_entropy_score) and exports harmonized libraries) — github.com/eMetaboHUB/FragHub
- **MS-Dial** (Consumes the filtered .MSP libraries for MS/MS-based spectral annotation of aligned feature lists (default MS-Dial parameters used in the 435-feature benchmark))
- **spectra-hash (SPLASH)** (Provides the SPLASH spectral hash keys used by FragHub to deduplicate processed spectra against update.json) — github.com/berlinguyinca/spectra-hash
- **MatchMS** (Python package in the FragHub ecosystem for spectrum processing referenced by the article)

## Examples

```
# In FragHub GUI: select .msp/.mgf/.json/.csv inputs -> filters tab: check_minimum_peak_requiered(n_peaks), remove_peak_above_precursormz, reduce_peak_list(max_peaks), normalize_intensity, keep_mz_in_range(mz_from, mz_to), check_minimum_of_high_peaks_requiered(intensity_percent, no_peaks), remove_spectrum_under_entropy_score(score) -> START
```

## Evaluation signals

- Every retained spectrum passes all selected filters: peak count >= minimum, no peaks above precursor m/z + 5 Da, peak list <= max_peaks, intensities normalized to max, all m/z within [mz_from, mz_to], and entropy score >= threshold.
- update.json SPLASH keys cover all processed spectra; re-running yields no duplicate processing of previously seen spectra.
- Output libraries are correctly partitioned into polarity (PI vs negative), LC/GC chromatographic mode, and experimental vs in-silico acquisition bins.
- All exported records retain consistent chemical identifiers — standardized SMILES, InChI, InChIKey and recalculated exact/average molecular masses via RDKit.
- Benchmark check: MS-Dial default-parameter search against the FragHub-unified library annotates 102/435 (24%) of the MS-CleanR-filtered A. thaliana PI features, versus 41 for MassBank alone and 55–81 for GNPS/MoNA/MS-Dial-VS17.

## Limitations

- All spectra lacking SMILES and InChI, precursor m/z, or adduct information are removed — coverage drops: the 794,985-spectrum / 35,673-identifier combined set is refined to 602,744 spectra / 32,193 identifiers.
- Collision-energy harmonization is not addressed due to ~70 different nonstandardized measures, which may affect cross-library scoring consistency.
- Redundancy across OMSLs may lead to inconsistencies with dot product and reverse dot product scoring, which are highly sensitive to fragment number and intensity.
- CSV inputs must be separated by ';' or tab with quotechar '"' and have a 'peaks' column.
- FragHub requires chemical identifiers and adduct info; it is not a general-purpose spectral cleanup for records missing those fields.

## Evidence

- [readme] SPLASH keys drive deduplication across processing runs: [evidence span withheld — anchor status: not_found]
- [results] Benchmark result showing the downstream value of filtered libraries: [evidence span withheld — anchor status: not_found]
