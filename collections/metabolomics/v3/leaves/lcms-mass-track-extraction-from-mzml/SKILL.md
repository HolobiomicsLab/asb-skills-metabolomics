---
name: lcms-mass-track-extraction-from-mzml
description: Use when centroided mzML LC-MS files must be converted into per-sample
  mass tracks (full-RT intensity vectors per unique m/z) and aligned into a MassGrid
  before peak picking or annotation.
when_to_use_negative:
- Input data are low-resolution mass spectra — the paper states 'The software does
  not work with low resolution data' since 0.001 amu binning requires high mass resolution
- You already have a feature table or composite mass tracks — this skill covers the
  upstream track-extraction/alignment stage only
- Data are MS/MS spectra — 'MS/MS spectra are ignored by asari' in the default LC-MS
  workflow
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: asari
  role: Implements the binning (get_thousandth_bins), NN-clustering by m/z seeds,
    full-RT track extraction, and MassGrid alignment
  repo: https://github.com/shuzhao-li/asari
- name: ThermoRawFileParser
  role: Converts Thermo .RAW files to centroided .mzML, the required input format
  repo: https://github.com/compomics/ThermoRawFileParser
- name: msConvert
  role: ProteoWizard converter handling most vendor data formats and .mzXML files
    into mzML
  repo: https://proteowizard.sourceforge.io/tools.shtml
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:f14dba8566fa3b85d07e2f095ccd80bbf641e760694f31521b0a5bc974953f47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - Python
  - ThermoRawFileParser
  - asari
  - msConvert
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# lcms-mass-track-extraction-from-mzml

## Summary

Extracts per-sample mass tracks (EICs of unique m/z values over the full RT range) from centroided mzML files by binning m/z into 0.001 amu bins and NN-clustering bins with seeds, then aligns the tracks into a MassGrid. This mass-first alignment is asari's core strategy for trackable feature detection.

## When to use

Use when centroided mzML LC-MS files must be converted into per-sample mass tracks (full-RT intensity vectors per unique m/z) and aligned into a MassGrid before peak picking or annotation.

## When NOT to use

- Input data are low-resolution mass spectra — the paper states 'The software does not work with low resolution data' since 0.001 amu binning requires high mass resolution
- You already have a feature table or composite mass tracks — this skill covers the upstream track-extraction/alignment stage only
- Data are MS/MS spectra — 'MS/MS spectra are ignored by asari' in the default LC-MS workflow

## Inputs

- Centroided mzML files (one per sample) from LC-MS, GC-MS, or DI-MS metabolomics acquisitions
- Thermo .RAW files (converted to mzML with ThermoRawFileParser) or vendor formats converted via msConvert

## Outputs

- Per-sample mass tracks: intensity vectors per unique m/z value spanning the full RT range
- Aligned MassGrid mapping export (_mass_grid_mapping.csv)
- Per-sample pickle intermediates (e.g., Blank_20210803_003.pickle) used by later composite-track and peak-detection steps

## How to apply

Start from centroided mzML files (convert Thermo .RAW with ThermoRawFileParser or use msConvert in ProteoWizard for other vendor formats). In asari, parse each mzML's MS1 spectra and bin m/z values into 0.001 amu bins via get_thousandth_bins, then NN-cluster bins by m/z using seeds (nn_cluster_by_mz_seeds). Extract each mass track over the full RT range as an intensity vector (extract_single_track_fullrt_length), prioritizing 13C/12C isotopic and Na/H adduct pairs as anchor mass tracks because they are ubiquitous and well-resolved. Align per-sample mass tracks into a MassGrid by mapping anchor mass tracks first and recalibrating m/z when a systematic difference above 1 ppm is detected (NN-clustering for larger studies); select the reference sample as the one with the most anchor mass tracks. The resulting tracks and MassGrid are exported as export/_mass_grid_mapping.csv and per-sample pickle files, and later summarized into composite mass tracks in the CompositeMap.

## Related tools

- **asari** (Implements the binning (get_thousandth_bins), NN-clustering by m/z seeds, full-RT track extraction, and MassGrid alignment) — https://github.com/shuzhao-li/asari
- **ThermoRawFileParser** (Converts Thermo .RAW files to centroided .mzML, the required input format) — https://github.com/compomics/ThermoRawFileParser
- **msConvert** (ProteoWizard converter handling most vendor data formats and .mzXML files into mzML) — https://proteowizard.sourceforge.io/tools.shtml

## Examples

```
asari process --mode pos --input mydir/projectx_dir
```

## Evaluation signals

- Every extracted mass track is an intensity vector spanning the full RT range for a unique m/z, binned at 0.001 amu granularity
- Anchor mass tracks (13C/12C isotopic pairs, Na/H adduct pairs) are identified per sample and used to seed the MassGrid alignment
- Systematic m/z differences above 1 ppm between a sample and the reference trigger recalibration; the reference sample has the highest number of anchor mass tracks
- Output project directory contains export/_mass_grid_mapping.csv and per-sample pickle files, and later composite tracks remain traceable back to individual sample tracks

## Limitations

- Asari has mostly been tested on Orbitrap platforms; settings need optimization for others
- Does not work with low-resolution data
- ppm-based m/z mapping is only a practical approximation; a 2nd order polynomial model may be required to cover the full m/z range more accurately

## Evidence

- [methods] From each mzML data file, mass tracks of unique m/z values are extracted, then aligned into a MassGrid.: "From each mzML data file, mass tracks of unique m/z values are extracted, then aligned into a MassGrid."
- [readme] We use ThermoRawFileParser (https://github.com/compomics/ThermoRawFileParser) to convert Thermo .RAW files to .mzML: "We use ThermoRawFileParser (https://github.com/compomics/ThermoRawFileParser) to convert Thermo .RAW files to .mzML"
- [readme] Taking advantage of high mass resolution to prioritize mass separation and alignment: "Taking advantage of high mass resolution to prioritize mass separation and alignment"
- [intro] To take advantage of the mass resolution in current data, mass alignment is performed first, represented by mass tracks within and composite mass tracks across acquisitions: "To take advantage of the mass resolution in current data, mass alignment is performed first, represented by mass tracks within and composite mass tracks across acquisitions"
