---
name: peak-intensity-thresholding
description: Use when curating a mass spectral library (.MSP, .MGF, .JSON) in FragHub
  and you need to drop sparse or low-intensity spectra before writing the final unified
  library.
when_to_use_negative:
- The input is raw untargeted MS/MS data from an instrument rather than an already
  integrated open mass spectral library, since FragHub's filtering stage assumes parsed,
  standardized spectra.
- The library is already split by ionization mode and chromatography method and requires
  metadata harmonization (SMILES/InChI/InChIKey recalculation) rather than peak-list
  quality filtering.
- The goal is to filter by precursor m/z, peak count beyond three, or entropy score,
  where other FragHub filters like remove_peak_above_precursormz, reduce_peak_list,
  or remove_spectrum_under_entropy_score are the appropriate choice.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FragHub
  role: Provides the GUI-based generic spectrum filters 'check_minimum_peak_requiered'
    and 'check_minimum_of_high_peaks_requiered' used to enforce the three-signal /
    two-high-peak-above-5% threshold.
  repo: https://github.com/eMetaboHUB/FragHub
- name: Python
  role: Underlying language in which FragHub's filter functions are implemented and
    exposed via its executable and GUI.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:cc79d4d8eb07d3ff3c12f1b271f29d61ea07acf56c64bc645d31389cae72a0db
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
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# peak-intensity-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies FragHub's two-part spectral quality gate: retain only mass spectra containing at least three peaks and at least two peaks whose intensity is at least 5% of the maximum intensity, removing low-information spectra from integrated OMSL libraries.

## When to use

Use when curating a mass spectral library (.MSP, .MGF, .JSON) in FragHub and you need to drop sparse or low-intensity spectra before writing the final unified library.

## When NOT to use

- The input is raw untargeted MS/MS data from an instrument rather than an already integrated open mass spectral library, since FragHub's filtering stage assumes parsed, standardized spectra.
- The library is already split by ionization mode and chromatography method and requires metadata harmonization (SMILES/InChI/InChIKey recalculation) rather than peak-list quality filtering.
- The goal is to filter by precursor m/z, peak count beyond three, or entropy score, where other FragHub filters like remove_peak_above_precursormz, reduce_peak_list, or remove_spectrum_under_entropy_score are the appropriate choice.

## Inputs

- Integrated mass spectral library file (.MSP, .MGF, .JSON, or .CSV) loaded into FragHub
- User-configured filter parameters: minimum number of peaks (e.g. 3) and intensity_percent threshold for high peaks (e.g. 5%)

## Outputs

- Filtered_spectra.msp containing only spectra meeting the three-peak minimum and the two-high-peak (≥5% relative intensity) criteria

## How to apply

Load the integrated spectra file into FragHub, then in the GUI filter tab apply 'check_minimum_peak_requiered' with a minimum of three peaks so that spectra with fewer peaks are ignored. Next apply 'check_minimum_of_high_peaks_requiered' defining a 'high peak' as one whose intensity is above a given percentage (intensity_percent, e.g. 5%) of the maximum intensity, and require at least two such peaks; spectra lacking enough high peaks are dropped. Rationale: these user-defined filters streamline peak lists to only essential data, reducing data size without compromising analytical relevance, and are part of FragHub's generic spectrum filtering via a graphical user interface. The two filters act sequentially, so only spectra passing both the count and relative-intensity criteria survive to the filtered_spectra output.

## Related tools

- **FragHub** (Provides the GUI-based generic spectrum filters 'check_minimum_peak_requiered' and 'check_minimum_of_high_peaks_requiered' used to enforce the three-signal / two-high-peak-above-5% threshold.) — https://github.com/eMetaboHUB/FragHub
- **Python** (Underlying language in which FragHub's filter functions are implemented and exposed via its executable and GUI.)

## Examples

```
FragHub GUI → Filter tab → enable 'check_minimum_peak_requiered' (n_peaks=3) and 'check_minimum_of_high_peaks_requiered' (intensity_percent=5, no_peaks=2) → START
```

## Evaluation signals

- The filtered output should contain only spectra with at least three peaks; spectra with fewer peaks should be absent from filtered_spectra.msp.
- Every surviving spectrum should have at least two peaks whose intensity is at least 5% of the maximum intensity; verify by recomputing relative intensities from the output peak arrays.
- The FragHub filter tab should show both check_minimum_peak_requiered and check_minimum_of_high_peaks_requiered selected with the configured parameters (e.g. n_peaks=3, intensity_percent=5) before START is pressed.
- The resulting library size should be smaller than the pre-filter integrated set (e.g. the article's 794,985 input spectra refined to 602,744 across all steps), confirming that sparse/low-intensity spectra were removed.
- Spectra removed by these filters should not appear in downstream analysis tools like MS-Dial, MZmine, or Flash Entropy Search consuming the filtered library.

## Limitations

- The filters operate on peak arrays only and do not validate chemical identity, so they cannot replace FragHub's stricter removal rule that drops spectra lacking SMILES, InChI, precursor m/z, and adduct information.
- The 'high peak' percentage is user-configurable (intensity_percent), so different parameter choices will yield different retained-spectra counts; the article's specific criterion is a minimum of two peaks above 5% of the maximum intensity.
- These filters are applied after integration and standardization, so they cannot rescue spectra that were already discarded for missing metadata during earlier FragHub processing steps.

## Evidence

- [results] FragHub applies a second, more stringent filter that excludes spectra unless they meet a minimum threshold of three MS/MS signals and two MS/MS peaks with intensities above 5% of the maximum intensity.: [evidence span withheld — anchor status: not_found]
