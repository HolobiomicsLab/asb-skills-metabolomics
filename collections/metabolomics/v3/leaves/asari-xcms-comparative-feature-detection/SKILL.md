---
name: asari-xcms-comparative-feature-detection
description: Use when you have centroided mzML files from an LC-MS experiment with
  a known sample ordering (dilution series, mixing ratios, spike-ins) and need to
  compare how well two processing tools (asari vs XCMS) detect features and whose
  peak areas track the expected signal.
when_to_use_negative:
- Input data are low-resolution mass spectrometry data — the article states asari
  'does not work with low resolution data' and is mostly tested on Orbitrap platforms
- You already have a single tool's feature table and only need annotation or statistical
  analysis — this skill is specifically a two-tool detection/quantification comparison
  on raw mzML
- The experiment has no known ground-truth ordering (no dilution series, mixing ratios,
  or spike-ins), so there is nothing to correlate peak areas against
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: asari
  role: Processes the mzML files (e.g. positive ionization mode, 5 ppm m/z precision)
    to produce preferred_Feature_table.tsv of peak areas
  repo: https://github.com/shuzhao-li/asari
- name: XCMS
  role: Baseline processing of the same mzML files using default peak-picking and
    correspondence workflow to produce the comparison feature table
- name: Python
  role: Feature matching by m/z and retention time, Pearson correlation computation,
    and binned correlation distribution plotting (requires Python 3.8+)
- name: ThermoRawFileParser
  role: Converts Thermo .RAW files to centroided .mzML before processing by either
    tool
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:68a13e4f11e4992fcd69dcd7af3bfc376e1d4b2df149fa68b662af62dc679cf1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - Python
  - ThermoRawFileParser
  - XCMS
  - asari
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# asari-xcms-comparative-feature-detection

## Summary

Benchmark LC-MS feature detection and quantification by processing the same centroided mzML dataset with both asari and XCMS, matching features across tools by m/z and retention time, and scoring peak areas against a known experimental variable (e.g. mixing ratios in a dilution series) via Pearson correlation.

## When to use

Use when you have centroided mzML files from an LC-MS experiment with a known sample ordering (dilution series, mixing ratios, spike-ins) and need to compare how well two processing tools (asari vs XCMS) detect features and whose peak areas track the expected signal.

## When NOT to use

- Input data are low-resolution mass spectrometry data — the article states asari 'does not work with low resolution data' and is mostly tested on Orbitrap platforms
- You already have a single tool's feature table and only need annotation or statistical analysis — this skill is specifically a two-tool detection/quantification comparison on raw mzML
- The experiment has no known ground-truth ordering (no dilution series, mixing ratios, or spike-ins), so there is nothing to correlate peak areas against

## Inputs

- Centroided mzML files from an LC-MS dilution/mixing-ratio series (e.g. BM21 human plasma and vegetable juice mixed at varying ratios)
- Known sample-level mixing ratios or expected concentration vector for each sample
- XCMS feature table of peak areas per sample (default peak-picking and correspondence)
- asari preferred_Feature_table.tsv of peak areas per sample

## Outputs

- Matched feature list keyed by m/z and retention time (e.g. 5,581 common XCMS/asari features)
- Per-feature Pearson correlation coefficients against the mixing ratios, for each tool
- Binned correlation distributions (bins spanning r = -1 to 1) for XCMS and asari, plottable side by side (Fig. 7d style)

## How to apply

Process the mzML files with XCMS using its default peak-picking and correspondence workflow to get a per-sample peak-area feature table, then process the identical files with asari in the matching ionization mode (e.g. `--mode pos`, default 5 ppm m/z precision) to generate its preferred_Feature_table.tsv. Match features between the two tools by m/z and retention time to recover the common feature set (in the BM21 plasma/vegetable-juice dilution study this yielded 5,581 matched features). For each matched feature, compute the Pearson correlation between peak areas across samples and the known mixing ratios. Bin the correlation values (bins spanning r from -1 to 1) separately for XCMS and asari features and plot the binned distributions side by side. The rationale: asari prioritizes mass separation and alignment on high-resolution data and performs peak detection on a composite map, so its quantification should track true concentration gradients more reliably; the r > 0.9 bin is the decision point for declaring which tool's quantification is more useful.

## Related tools

- **asari** (Processes the mzML files (e.g. positive ionization mode, 5 ppm m/z precision) to produce preferred_Feature_table.tsv of peak areas) — https://github.com/shuzhao-li/asari
- **XCMS** (Baseline processing of the same mzML files using default peak-picking and correspondence workflow to produce the comparison feature table)
- **Python** (Feature matching by m/z and retention time, Pearson correlation computation, and binned correlation distribution plotting (requires Python 3.8+))
- **ThermoRawFileParser** (Converts Thermo .RAW files to centroided .mzML before processing by either tool) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
asari process --mode pos --input mydir/projectx_dir
```

## Evaluation signals

- The m/z + retention-time matching recovers a stable common feature set (in the published BM21 dataset, 5,581 features matched between XCMS and asari)
- The binned correlation distribution for asari shows more features in the r > 0.9 bin than XCMS, consistent with the published Fig. 7d pattern
- The asari run produces the expected output structure including preferred_Feature_table.tsv, export/full_Feature_table.tsv and project.json
- Pearson correlations are computed per matched feature across all samples against the known mixing ratios, with bins spanning r from -1 to 1
- Both tools are run on the identical mzML input files so any difference in feature quality is attributable to the processing algorithms

## Limitations

- Asari has mostly been tested on Orbitrap platforms; settings need further optimization for other instruments
- The software does not work with low-resolution data, and efficiency is not optimal for data from longer chromatography
- MS/MS spectra are ignored in the default LC-MS workflow, so the comparison covers MS1 feature detection and quantification only
- Borderline feature cases are a tradeoff between sensitivity and true discovery rate; feature matching across tools depends on tolerance choices for m/z and retention time

## Evidence

- [results] Of the 5,581 features matched between XCMS and asari in the BM21 dataset, asari shows more features with Pearson correlation coefficient >0.9 to the mixing ratios: [evidence span withheld — anchor status: not_found]
- [abstract] Asari compares favorably to other tools in feature detection and quantification: "Asari compares favorably to other tools in feature detection and quantification"
- [intro] The most widely used are XCMS and MZmine: "Multiple software tools have been developed for LC-MS metabolomics data processing over the years"
- [readme] The recommended feature table is preferred_Feature_table.tsv: "The recommended feature table is `preferred_Feature_table.tsv`."
- [discussion] Asari has a short history and has been mostly tested on the Orbitrap platforms: "Asari has a short history and has been mostly tested on the Orbitrap platforms. The settings will need further optimization for other platforms."
