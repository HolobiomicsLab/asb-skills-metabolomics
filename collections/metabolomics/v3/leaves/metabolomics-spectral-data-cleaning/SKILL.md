---
name: metabolomics-spectral-data-cleaning
description: Use when an integrated spectral library (.MSP/.MGF/.JSON/.CSV) contains
  spectra with sparse peak lists, and low-information spectra must be dropped via
  a minimum-peak-count filter before library harmonization or search.
when_to_use_negative:
- Input spectra are already fragment-rich (peak counts well above the threshold),
  so the filter adds no information — apply it only to enforce a minimum, not to trim
  rich spectra
- The task is metadata harmonization (SMILES/InChI/InChIKey recalculation, PubChem
  ontology completion) rather than peak-list cleaning — use FragHub's identifier-normalization
  workflow instead
- Spectra should be filtered on precursor-related or entropy criteria (e.g. remove_peak_above_precursormz,
  remove_spectrum_under_entropy_score) rather than raw peak count
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FragHub
  role: Provides the check_minimum_peak_requiered filter and the GUI filter-selection
    tab through which the minimum-peak threshold is configured and applied to peak
    lists
  repo: github.com/eMetaboHUB/FragHub
- name: Python
  role: Implementation language of the FragHub filter code (publicly available under
    an MIT license per the paper)
- name: MatchMS
  role: Python package ecosystem in which FragHub's filtering sits; MatchMS provides
    spectral-filtering primitives for MS/MS peak lists
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:d03d5e51d02e8c2af44a33fb8b7544ee9ebea44c9a94abcd62e2179a3faba203
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
  - MatchMS
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolomics-spectral-data-cleaning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies FragHub's user-defined peak-list filters (e.g. check_minimum_peak_requiered with a minimum peak threshold of 3) to drop spectra with too few MS/MS signals, as part of cleaning integrated open mass spectral libraries (.MSP/.MGF) prior to harmonized output.

## When to use

Use when an integrated spectral library (.MSP/.MGF/.JSON/.CSV) contains spectra with sparse peak lists, and low-information spectra must be dropped via a minimum-peak-count filter before library harmonization or search.

## When NOT to use

- Input spectra are already fragment-rich (peak counts well above the threshold), so the filter adds no information — apply it only to enforce a minimum, not to trim rich spectra
- The task is metadata harmonization (SMILES/InChI/InChIKey recalculation, PubChem ontology completion) rather than peak-list cleaning — use FragHub's identifier-normalization workflow instead
- Spectra should be filtered on precursor-related or entropy criteria (e.g. remove_peak_above_precursormz, remove_spectrum_under_entropy_score) rather than raw peak count

## Inputs

- Integrated spectral library file in .MSP, .MGF, .JSON, or .CSV format with a 'peaks' column/array per spectrum
- User-selected minimum-peak threshold (n_peaks, e.g. 3) set in the FragHub GUI filter tab

## Outputs

- Filtered spectral library containing only spectra whose peak lists meet the minimum peak count
- QC counts of spectra removed vs retained by the filter

## How to apply

Load the integrated FragHub spectral library (e.g. .MSP or .MGF input) and read each spectrum's peak list from the 'peaks' field. Apply FragHub's check_minimum_peak_requiered(peak_array, n_peaks) filter via the GUI's filter-selection tab, setting the minimum-peak threshold (e.g. n_peaks = 3); any spectrum whose peak array holds fewer than the minimum number of peaks is ignored (dropped). The rationale is that spectra with fewer than three MS/MS signals carry too little fragment information for reliable dereplication and consistency with dot-product/reverse-dot-product scoring. Optionally combine with the other FragHub filters exposed in the third GUI tab (remove_peak_above_precursormz, reduce_peak_list, normalize_intensity, keep_mz_in_range, check_minimum_of_high_peaks_requiered, remove_spectrum_under_entropy_score), whose parameters are user-adjustable. Write the retained spectra to the chosen output format(s) and record how many spectra were removed versus retained; note FragHub's whole-workflow refinement went from 794,985 to 602,744 spectra, so per-filter removal counts are the QC signal.

## Related tools

- **FragHub** (Provides the check_minimum_peak_requiered filter and the GUI filter-selection tab through which the minimum-peak threshold is configured and applied to peak lists) — github.com/eMetaboHUB/FragHub
- **Python** (Implementation language of the FragHub filter code (publicly available under an MIT license per the paper))
- **MatchMS** (Python package ecosystem in which FragHub's filtering sits; MatchMS provides spectral-filtering primitives for MS/MS peak lists)

## Examples

```
GUI: FragHub.exe → Tab 3 filters → enable check_minimum_peak_requiered with n_peaks = 3 → START
```

## Evaluation signals

- Output library exists and every remaining spectrum's peak array contains at least n_peaks (e.g. 3) MS/MS peaks
- Number of dropped spectra equals the count of input spectra with fewer than n_peaks peaks — recomputable by re-parsing the input 'peaks' field
- Retained spectra count matches expected input minus dropped; whole-workflow scale check: FragHub refined 794,985 input spectra to 602,744
- All retained spectra still carry consistent FragHub-standardized metadata (fields and values unchanged by this peak-count filter)
- Inconsistent spectra lacking SMILES, InChI, precursor m/z, and adduct information are additionally removed by FragHub's consistency rule

## Limitations

- Overly aggressive peak-count thresholds can discard otherwise usable low-fragment spectra; the paper notes redundancy and fragment-number sensitivity cause inconsistencies with dot product and reverse dot product scoring
- The filter only enforces a minimum; spectra passing it may still need intensity-based filtering (check_minimum_of_high_peaks_requiered) or entropy-based removal (remove_spectrum_under_entropy_score)
- FragHub removes all spectra lacking SMILES, InChI, precursor m/z, and adduct information regardless of peak count, so filtering results are conditional on metadata completeness

## Evidence

- [readme] README filter documentation: [evidence span withheld — anchor status: not_found]
