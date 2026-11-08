---
name: user-defined-spectrum-filtering
description: Use when converting .MSP/.MGF/.JSON/.CSV spectral libraries (e.g., GNPS,
  MoNA, MassBank, MS-Dial, or MZmine in-house exports) into a cleaned, standardized
  FragHub output and you need to drop spectra with too few peaks, excessive entropy-reduction,
  or peaks above the precursor m/z.
when_to_use_negative:
- The input is already a feature table, annotated peak list, or post-dereplication
  result rather than a raw spectral library file (.MSP/.MGF/.JSON/.CSV) needing standardization
- You require harmonization of collision-energy values across libraries — FragHub
  explicitly does not address collision-energy harmonization due to ~70 nonstandardized
  formats
- Your workflow needs the input library converted without any peak-level filtering,
  e.g., a strict lossless catalog build where every spectrum must be preserved regardless
  of peak count, entropy, or precursor-m/z consistency
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FragHub
  role: Provides the GUI filter tab and the named spectral-filter functions (check_minimum_peak_requiered,
    remove_peak_above_precursormz, reduce_peak_list, normalize_intensity, keep_mz_in_range,
    check_minimum_of_high_peaks_requiered, remove_spectrum_under_entropy_score) applied
    to the parsed library
  repo: https://github.com/eMetaboHUB/FragHub
- name: MZmine
  role: Source of in-house spectral libraries whose .MSP/.JSON/.MGF/.CSV exports are
    accepted as filter input, following the MZmine tutorial (Supporting Information
    S6)
- name: RDKit
  role: Used internally by FragHub to standardize chemical identifiers (SMILES, InChI,
    InChIKey) and compute exact/average masses during the same filtering pass
- name: MS-Dial
  role: Downstream analysis software that consumes the filtered, standardized FragHub
    output spectra
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:f12e2539091eae405c28af0c5c1904dd388383ca034702b9aa1de9a8e3faa8de
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
  - MS-Dial
  - MZmine
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# user-defined-spectrum-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies FragHub's optional user-configurable spectral filters (via its GUI filter tab) to a parsed mass spectral library, removing low-information or inconsistent spectra (e.g., those lacking SMILES/InChI, precursor m/z, or adduct) and trimming peak lists before export in a unified format.

## When to use

Use when converting .MSP/.MGF/.JSON/.CSV spectral libraries (e.g., GNPS, MoNA, MassBank, MS-Dial, or MZmine in-house exports) into a cleaned, standardized FragHub output and you need to drop spectra with too few peaks, excessive entropy-reduction, or peaks above the precursor m/z.

## When NOT to use

- The input is already a feature table, annotated peak list, or post-dereplication result rather than a raw spectral library file (.MSP/.MGF/.JSON/.CSV) needing standardization
- You require harmonization of collision-energy values across libraries — FragHub explicitly does not address collision-energy harmonization due to ~70 nonstandardized formats
- Your workflow needs the input library converted without any peak-level filtering, e.g., a strict lossless catalog build where every spectrum must be preserved regardless of peak count, entropy, or precursor-m/z consistency

## Inputs

- Mass spectral library files (.MSP, .MGF, .JSON, .CSV, .XML) from open mass spectral libraries such as GNPS, MoNA, MassBank, MS-Dial, HMDB, RIKEN, or in-house MZmine3 spectral library generation exports
- User-selected filter configuration (which filters to enable and their parameters) supplied via the FragHub GUI filter tab
- An output directory path, either empty (new project) or containing a prior FragHub run's 'update.json' SPLASH-key repository for incremental updates

## Outputs

- Filtered, standardized spectra in unified formats (.MSP and .JSON among others), normalized metadata (SMILES/InChI/InChIKey verified via RDKit, exact and average masses, HUPO PSI-MS instrument/ionization terms), and SPLASH keys for each spectrum
- Spectra segregated by ionization mode (positive/negative), separation technique (LC/GC), and acquisition type (experimental vs in-silico), with an 'update.json' SPLASH-key repository enabling incremental processing on library updates

## How to apply

On FragHub's filter tab, enable and parameterize the desired subset of filters: check_minimum_peak_requiered (drop spectra below a minimum peak count), remove_peak_above_precursormz (delete peaks with m/z greater than precursor m/z + 5 Da), reduce_peak_list (retain the top-N peaks by intensity), normalize_intensity (rescale peaks to max intensity = 1), keep_mz_in_range (restrict peaks to a [mz_from, mz_to] window), check_minimum_of_high_peaks_requiered (require a minimum count of peaks above intensity_percent of max), and remove_spectrum_under_entropy_score (delete spectra whose entropy score falls below a threshold). The rationale is to standardize heterogeneous library spectra without compromising analytical relevance; the article's integration of 794,985 spectra from GNPS (.MGF), MoNA (.MSP/.SDF/.JSON), MassBank, and MS-Dial-VS17 reduced them to 602,744 spectra for 32,193 unique chemicals using these filters plus metadata harmonization. FragHub also removes any spectrum lacking both SMILES and InChI, precursor m/z, or adduct information, ensuring downstream compatibility with MS-Dial, MZmine, and Flash Entropy Search.

## Related tools

- **FragHub** (Provides the GUI filter tab and the named spectral-filter functions (check_minimum_peak_requiered, remove_peak_above_precursormz, reduce_peak_list, normalize_intensity, keep_mz_in_range, check_minimum_of_high_peaks_requiered, remove_spectrum_under_entropy_score) applied to the parsed library) — https://github.com/eMetaboHUB/FragHub
- **MZmine** (Source of in-house spectral libraries whose .MSP/.JSON/.MGF/.CSV exports are accepted as filter input, following the MZmine tutorial (Supporting Information S6))
- **RDKit** (Used internally by FragHub to standardize chemical identifiers (SMILES, InChI, InChIKey) and compute exact/average masses during the same filtering pass)
- **MS-Dial** (Downstream analysis software that consumes the filtered, standardized FragHub output spectra)

## Examples

```
FragHub.exe (GUI): Tab 1 select input .msp/.mgf/.json/.csv files → Tab 3 enable 'check_minimum_peak_requiered', 'remove_peak_above_precursormz', 'normalize_intensity' → click START
```

## Evaluation signals

- Output spectrum count and unique-chemical count should be lower than input counts; the article's reference run went from 794,985 spectra (35,673 unique chemical identifiers) to 602,744 spectra (32,193 unique chemicals) after filtering and harmonization
- No output spectrum should lack both SMILES and InChI, or lack precursor m/z and adduct information, since FragHub removes such inconsistent spectra during processing
- Every output spectrum carries a SPLASH key generated per the Wohlgemuth et al. algorithm, recorded in 'update.json'; re-running on an unchanged library should process zero new spectra
- Peaks with m/z greater than precursor m/z + 5 Da must be absent from any spectrum where remove_peak_above_precursormz was enabled
- Peak intensities are normalized to the maximum intensity in each spectrum where normalize_intensity was enabled, so the maximum peak intensity equals 1 (or the library's defined maximum) in outputs
- Output spectra conform to GNPS naming conventions and are segregated into positive/negative, LC/GC, and experimental/in-silico subdirectories

## Limitations

- Filtering may discard structurally informative spectra — e.g., the article notes remove_peak_above_precursormz removes all peaks above precursor m/z + 5 Da, which can be too aggressive for spectra with diagnostic high-mass fragments
- The FragHub library coverage (32,193 unique chemicals) is low relative to the estimated millions of natural products, so filtered outputs cannot annotate the full diversity of natural product space
- Redundancy across OMSLs may remain post-filtering, causing inconsistencies when using dot-product and reverse dot-product scoring systems sensitive to fragment number and intensity
- FragHub does not harmonize collision energies (about 70 different formats exist), so filtered spectra may still differ in fragmentation conditions across source libraries
- In the 20_12_2024 release, MacOS and Linux support was removed and the license changed from MIT to CC-BY-NC 4.0, imposing non-commercial restrictions on use of the tool

## Evidence
