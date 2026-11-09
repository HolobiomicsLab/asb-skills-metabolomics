---
name: lcms-feature-correspondence-matching-across-tools
description: Use when benchmarking feature detection/quantification of LC-MS processing
  tools on a dilution or mixing-ratio dataset, where feature tables from multiple
  tools must be matched and ranked by correlation of peak areas to known abundances.
when_to_use_negative:
- The data lack known ground-truth abundances (no mixing ratios, spike-ins, or dilution
  series), so correlation-to-truth cannot be computed
- The input is already a single tool's feature table with no second tool to compare
  against
- The data are low-resolution or non-Orbitrap platforms where asari is not yet optimized
  — tool comparison conclusions may not transfer
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: asari
  role: Processes the mzML dilution-series files (positive mode, default 5 ppm m/z
    precision) to generate preferred_Feature_table.tsv peak areas for one side of
    the comparison
  repo: https://github.com/shuzhao-li/asari
- name: XCMS
  role: Processes the same mzML files with default peak-picking and correspondence
    to generate the reference feature table of peak areas
- name: Python
  role: Used to match features by m/z and retention time, compute Pearson correlations
    against mixing ratios, bin r values, and plot the comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:f8ce8db55aeb5950e621d643bd1ebb089787994d023c93c9870ae7c2865d9584
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - Python
  - XCMS
  - asari
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# lcms-feature-correspondence-matching-across-tools

## Summary

Matches LC-MS features detected by two processing tools (e.g., asari and XCMS) on the same dataset by m/z and retention time, then compares their quantification quality by correlating per-feature peak areas against known sample mixing ratios (Pearson r).

## When to use

Use when benchmarking feature detection/quantification of LC-MS processing tools on a dilution or mixing-ratio dataset, where feature tables from multiple tools must be matched and ranked by correlation of peak areas to known abundances.

## When NOT to use

- The data lack known ground-truth abundances (no mixing ratios, spike-ins, or dilution series), so correlation-to-truth cannot be computed
- The input is already a single tool's feature table with no second tool to compare against
- The data are low-resolution or non-Orbitrap platforms where asari is not yet optimized — tool comparison conclusions may not transfer

## Inputs

- Centroided mzML files from an LC-MS dilution/mixing series with known mixing ratios per sample (e.g., BM21 human plasma and vegetable juice dilution dataset)
- XCMS feature table of peak areas per sample (default peak-picking and correspondence workflow)
- asari preferred_Feature_table.tsv of peak areas per sample
- Vector of known mixing ratios for each sample in the dilution series

## Outputs

- Matched feature list keyed by m/z and retention time (e.g., 5,581 common features between XCMS and asari)
- Per-feature Pearson correlation coefficients between peak areas and mixing ratios, for each tool
- Binned correlation distributions (bins spanning r = -1 to 1) and a side-by-side comparison plot analogous to Fig. 7d

## How to apply

Process the same centroided mzML files independently with each tool: XCMS with its default peak-picking and correspondence workflow to get per-sample peak areas, and asari in the relevant mode (e.g., positive, default 5 ppm m/z precision) to produce its preferred_Feature_table.tsv. Match features between the two tables by m/z and retention time to find common features (in the BM21 plasma/vegetable-juice dilution study this recovered 5,581 matched features). For each matched feature, compute the Pearson correlation between peak areas across the dilution-series samples and the known mixing ratios, then bin the r values (e.g., bins spanning -1 to 1) per tool and plot the binned distributions side by side as in Fig. 7d. The rationale is that a tool whose peak areas track ground-truth mixing ratios more tightly provides more useful quantification, independently of feature-count differences.

## Related tools

- **asari** (Processes the mzML dilution-series files (positive mode, default 5 ppm m/z precision) to generate preferred_Feature_table.tsv peak areas for one side of the comparison) — https://github.com/shuzhao-li/asari
- **XCMS** (Processes the same mzML files with default peak-picking and correspondence to generate the reference feature table of peak areas)
- **Python** (Used to match features by m/z and retention time, compute Pearson correlations against mixing ratios, bin r values, and plot the comparison)

## Examples

```
asari process --mode pos --input BM21_mzml_dir; # then in Python, match XCMS and asari feature tables by m/z and rtime, correlate peak areas vs mixing ratios, bin r in [-1, 1]
```

## Evaluation signals

- The number of features matched by m/z and retention time between the two tools is consistent with the expected count (5,581 for the BM21 dataset)
- The binned Pearson-correlation distribution shows more asari features in the r > 0.9 bin than XCMS, reproducing the Fig. 7d pattern
- Every matched feature has a defined Pearson r against the mixing-ratio vector; unmatched or NaN correlations are excluded and counted
- asari output contains preferred_Feature_table.tsv with per-sample peak-area columns matching the dilution-series sample names

## Limitations

- asari has mostly been tested on Orbitrap platforms; settings need optimization elsewhere, so tool-comparison results may not generalize to other instruments
- The software does not work with low-resolution data
- Asari ignores MS/MS spectra in the default LC-MS workflow, so the comparison is restricted to MS1 peak-area quantification
- The article attributes inconsistency among processing tools to deficiencies in mass alignment and feature quality controls, so matched-feature counts depend on each tool's alignment quality

## Evidence

- [results] Of the 5,581 features matched between XCMS and asari in the BM21 dataset, asari shows more features with Pearson correlation coefficient >0.9 to the mixing ratios, indicating its quantification is more useful than XCMS's.: [evidence span withheld — anchor status: not_found]
- [abstract] Asari compares favorably to other tools in feature detection and quantification: "Asari compares favorably to other tools in feature detection and quantification"
- [readme] Input data are centroid mzML files from LC, GC or DI-MS metabolomics.: "Input data are centroid mzML files from LC, GC or DI-MS metabolomics. "
- [readme] The recommended feature table is preferred_Feature_table.tsv.: "The recommended feature table is `preferred_Feature_table.tsv`. "
- [discussion] Asari has a short history and has been mostly tested on the Orbitrap platforms. The settings will need further optimization for other platforms.: "Asari has a short history and has been mostly tested on the Orbitrap platforms. The settings will need further optimization for other platforms."
- [abstract] Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality: "Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality"
