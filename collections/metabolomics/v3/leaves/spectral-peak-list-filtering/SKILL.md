---
name: spectral-peak-list-filtering
description: Use when preparing an integrated spectral library (.MSP/.MGF) where low-information
  spectra with too few MS/MS peaks or few intense fragments must be dropped to improve
  downstream dereplication/annotation quality.
when_to_use_negative:
- Spectra are already feature-quantification tables or peak-picked LC-MS run data,
  not library peak lists
- The goal is metadata harmonization (SMILES/InChI/InChIKey normalization) rather
  than peak-list quality filtering
- Collision-energy or other metadata harmonization is the target — the paper explicitly
  did not address harmonizing ~70 nonstandardized collision-energy formats
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FragHub
  role: Runs the generic peak-list filters (check_minimum_peak_requiered, check_minimum_of_high_peaks_requiered,
    etc.) on integrated OMSL spectra via its GUI filter-selection tab
  repo: https://github.com/eMetaboHUB/FragHub
- name: Python
  role: Language in which the FragHub filter functions are implemented and scriptable
  repo: https://github.com/eMetaboHUB/FragHub
- name: MatchMS
  role: Python package ecosystem that FragHub's filtering approach complements for
    spectral processing
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:97ea479fb3dd7be5062d9105d1c3273c811aff287407e05fa87fdc70af35872f
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# spectral-peak-list-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies FragHub's user-selectable generic peak-list filters (e.g. check_minimum_peak_requiered, check_minimum_of_high_peaks_requiered) to drop low-quality mass spectra from an integrated OMSL library before output, retaining only spectra with sufficient MS/MS signal content.

## When to use

Use when preparing an integrated spectral library (.MSP/.MGF) where low-information spectra with too few MS/MS peaks or few intense fragments must be dropped to improve downstream dereplication/annotation quality.

## When NOT to use

- Spectra are already feature-quantification tables or peak-picked LC-MS run data, not library peak lists
- The goal is metadata harmonization (SMILES/InChI/InChIKey normalization) rather than peak-list quality filtering
- Collision-energy or other metadata harmonization is the target — the paper explicitly did not address harmonizing ~70 nonstandardized collision-energy formats

## Inputs

- Integrated spectral library file (.MSP, .MGF, .JSON, or .CSV) with per-spectrum peak lists in a 'peaks' column/array
- User-selected filter set and parameters via the FragHub GUI (e.g. n_peaks=3, intensity_percent=5, no_peaks=2)

## Outputs

- Filtered spectra written to a new output library file (e.g. filtered_spectra.msp) in user-selected formats
- Counts of removed versus retained spectra for QC

## How to apply

Load the integrated spectral library into FragHub's GUI (third filter-selection tab), where each spectrum's peak array is read from the 'peaks' field. Apply check_minimum_peak_requiered with n_peaks=3, dropping any spectrum with fewer than 3 MS/MS signals. Then apply check_minimum_of_high_peaks_requiered with intensity_percent=5 and no_peaks=2, dropping spectra that lack at least two peaks above 5% of the maximum intensity. Optionally chain further filters such as remove_peak_above_precursormz (peaks above precursor m/z + 5 Da), reduce_peak_list (keep most-intense peaks up to a max), normalize_intensity, keep_mz_in_range, and remove_spectrum_under_entropy_score. The rationale is to streamline peak lists so only informative fragments survive, reducing library size without compromising annotation relevance.

## Related tools

- **FragHub** (Runs the generic peak-list filters (check_minimum_peak_requiered, check_minimum_of_high_peaks_requiered, etc.) on integrated OMSL spectra via its GUI filter-selection tab) — https://github.com/eMetaboHUB/FragHub
- **Python** (Language in which the FragHub filter functions are implemented and scriptable) — https://github.com/eMetaboHUB/FragHub
- **MatchMS** (Python package ecosystem that FragHub's filtering approach complements for spectral processing)

## Examples

```
# In FragHub GUI third tab: select check_minimum_peak_requiered (n_peaks=3) and check_minimum_of_high_peaks_requiered (intensity_percent=5, no_peaks=2), then click START on input.msp
```

## Evaluation signals

- Every retained spectrum's peak array contains at least 3 MS/MS peaks (check_minimum_peak_requiered with n_peaks=3)
- Every retained spectrum has at least 2 peaks with intensity ≥5% of its maximum intensity (check_minimum_of_high_peaks_requiered)
- Removed-spectrum count plus retained-spectrum count equals the input spectrum count (no spectra lost or duplicated)
- Filtered output file exists and parses in the chosen output format (.MSP/.MGF/.JSON/.CSV)
- Spectra lacking SMILES and InChI, precursor m/z, and adduct information were removed as inconsistent per FragHub processing rules

## Limitations

- Filter parameters are user-defined; the three-peak and two-high-peak (>5% max intensity) thresholds are the paper's examples and may not suit every library
- Aggressive peak-list filtering (e.g. reduce_peak_list keeping only most-intense peaks) can interact with dot-product and reverse dot-product scoring, which are sensitive to fragment number and intensity
- Spectra lacking SMILES/InChI, precursor m/z, or adduct information are dropped entirely regardless of peak-list quality

## Evidence
