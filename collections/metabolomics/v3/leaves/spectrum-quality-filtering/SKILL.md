---
name: spectrum-quality-filtering
description: Use when an integrated OMSL (e.g. .MSP or .MGF files from GNPS, MoNA,
  MassBank) must be cleaned before export, and spectra with too few peaks or weak
  high-intensity peaks should be removed.
when_to_use_negative:
- Input is already a curated spectral library where every spectrum is known to satisfy
  the minimum peak-count and high-intensity criteria — reapplying the filters would
  only add overhead
- Peak lists are required in full, unfiltered form (e.g. for raw reprocessing or fragmentation-detail
  mining) rather than for spectral matching, since these filters permanently drop
  low-peak spectra
- Spectra lack SMILES, InChI, precursor m/z, or adduct metadata; FragHub removes these
  inconsistent spectra during processing anyway, so quality filtering alone will not
  salvage them
tools:
- name: FragHub
  role: Hosts the check_minimum_peak_requiered and check_minimum_of_high_peaks_requiered
    generic filters, selectable and parameterized via its GUI filter tab
  repo: github.com/eMetaboHUB/FragHub
- name: Python
  role: Implementation language of the FragHub filtering functions; filter logic can
    be adapted in Python scripts
- name: MatchMS
  role: Ecosystem package cited alongside FragHub's spectral processing; its satellites
    (e.g. MSMetaEnhancer) complement metadata harmonization, though the quality filters
    themselves are FragHub-specific
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:8e69417023691d558be74736f9e8d97bc99f671f64a27ce1afaaf122e058c3a9
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectrum-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies FragHub's generic spectral-quality filters to mass spectra in an integrated Open Mass Spectra Library (OMSL), dropping spectra that lack a minimum number of MS/MS peaks or sufficient high-intensity peaks so the resulting library contains only usable reference spectra.

## When to use

Use when an integrated OMSL (e.g. .MSP or .MGF files from GNPS, MoNA, MassBank) must be cleaned before export, and spectra with too few peaks or weak high-intensity peaks should be removed.

## When NOT to use

- Input is already a curated spectral library where every spectrum is known to satisfy the minimum peak-count and high-intensity criteria — reapplying the filters would only add overhead
- Peak lists are required in full, unfiltered form (e.g. for raw reprocessing or fragmentation-detail mining) rather than for spectral matching, since these filters permanently drop low-peak spectra
- Spectra lack SMILES, InChI, precursor m/z, or adduct metadata; FragHub removes these inconsistent spectra during processing anyway, so quality filtering alone will not salvage them

## Inputs

- Integrated or single-source mass spectral library in .MSP, .MGF, .JSON, .CSV, or .XML format with a peaks field (CSV must use ';' or tab separators with quotechar '"' and a 'peaks' column)

## Outputs

- Filtered spectral library retaining only spectra with ≥3 MS/MS peaks and ≥2 peaks at ≥5% of maximum intensity, in FragHub-compatible output formats (.MSP, .MGF, .JSON) for tools such as MS-Dial, MZmine, or Flash Entropy Search
- Count of spectra dropped versus retained for QC reporting

## How to apply

Load the integrated spectral library and apply FragHub's check_minimum_peak_requiered filter with a minimum-peak threshold of 3, dropping any spectrum whose peak array contains fewer than three MS/MS signals. Then apply check_minimum_of_high_peaks_requiered with intensity_percent=5 and no_peaks=2: a 'high peak' is one whose intensity is at least 5% of the maximum intensity, and spectra lacking at least two such peaks are ignored. These filters are selected and parameterized through the FragHub GUI filter tab, alongside optional filters such as remove_peak_above_precursormz, reduce_peak_list, normalize_intensity, keep_mz_in_range, and remove_spectrum_under_entropy_score. The rationale is to retain only spectra informative for spectral matching (e.g. dot-product or Flash Entropy Search scoring), since low-peak or near-single-peak spectra contribute noise and inconsistent scoring. Write surviving spectra to a filtered output file and record how many spectra were removed versus retained for QC.

## Related tools

- **FragHub** (Hosts the check_minimum_peak_requiered and check_minimum_of_high_peaks_requiered generic filters, selectable and parameterized via its GUI filter tab) — github.com/eMetaboHUB/FragHub
- **Python** (Implementation language of the FragHub filtering functions; filter logic can be adapted in Python scripts)
- **MatchMS** (Ecosystem package cited alongside FragHub's spectral processing; its satellites (e.g. MSMetaEnhancer) complement metadata harmonization, though the quality filters themselves are FragHub-specific)

## Examples

```
# In the FragHub GUI, filter tab: enable 'check_minimum_peak_requiered' (n_peaks=3) and 'check_minimum_of_high_peaks_requiered' (intensity_percent=5, no_peaks=2), then click START on the loaded .MSP library.
```

## Evaluation signals

- After filtering, every remaining spectrum's peak list contains at least 3 MS/MS peaks
- For each remaining spectrum, count of peaks with intensity ≥5% of the maximum intensity is at least 2
- No dropped spectra reappear in the output file; the difference between input and output spectrum counts equals the number of removed spectra
- Output file loads cleanly in target analysis software (e.g. MS-Dial, MZmine, Flash Entropy Search) without format errors

## Limitations

- Filters permanently remove spectra rather than flagging them, so low-peak spectra cannot be recovered without re-running from the unfiltered source
- The three-peak / two-high-peak-above-5% thresholds are user-adjustable; defaults are not guaranteed optimal for every library, and overly strict settings may discard valid spectra of low fragmentation intensity
- Quality filtering alone does not address redundancy across OMSLs, which the article notes can cause inconsistencies with dot-product and reverse dot-product scoring sensitive to fragment number and intensity
- CSV input must follow FragHub's strict format (semicolon or tab separator, double-quote quotechar, peaks column named 'peaks') or the filter cannot parse the peak data

## Evidence

- [results] FragHub applies a second, more stringent filter that excludes spectra unless they meet a minimum threshold of three MS/MS signals and two MS/MS peaks with intensities above 5% of the maximum intensity.: [evidence span withheld — anchor status: not_found]
