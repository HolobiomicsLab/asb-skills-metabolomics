---
name: lcms-feature-quantification-peak-area-extraction
description: Use when LC-MS data contain a dilution/mixing series with known composition
  ratios (e.g. BM21 plasma–vegetable juice dilutions) and you need to compare peak-area
  quantification quality across processing tools like asari vs XCMS.
when_to_use_negative:
- Input is already a feature table from a single tool with no competing quantification
  to benchmark, and no dilution/mixing series exists to define expected abundance
  gradients
- Data are low-resolution LC-MS or non-Orbitrap platforms, since asari does not work
  with low-resolution data and its settings are mostly tested on Orbitrap platforms
- MS/MS-centric analyses, because MS/MS spectra are ignored by asari's default LC-MS
  workflow
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: asari
  role: Processes BM21 mzML files in positive mode with default 5 ppm m/z precision
    to produce preferred_Feature_table.tsv of peak areas
  repo: github.com/shuzhao-li-lab/asari
- name: XCMS
  role: Processes the same BM21 mzML files with default peak-picking and correspondence
    workflow to generate a comparison feature table of peak areas
- name: Python
  role: Used for feature matching by m/z and retention time, Pearson correlation computation,
    binning of r values, and plotting the Fig. 7d-style distributions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:37b1d6ae11145fa89da3525ae7227c76dab87c367009d7a6e059062bb9e8df27
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - XCMS
  - asari
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lcms-feature-quantification-peak-area-extraction

## Summary

Extract peak-area feature tables from LC-MS metabolomics data using asari and XCMS, then evaluate quantification quality by correlating feature peak areas against known sample mixing ratios in a dilution series (BM21).

## When to use

Use when LC-MS data contain a dilution/mixing series with known composition ratios (e.g. BM21 plasma–vegetable juice dilutions) and you need to compare peak-area quantification quality across processing tools like asari vs XCMS.

## When NOT to use

- Input is already a feature table from a single tool with no competing quantification to benchmark, and no dilution/mixing series exists to define expected abundance gradients
- Data are low-resolution LC-MS or non-Orbitrap platforms, since asari does not work with low-resolution data and its settings are mostly tested on Orbitrap platforms
- MS/MS-centric analyses, because MS/MS spectra are ignored by asari's default LC-MS workflow

## Inputs

- BM21 dilution-series mzML files (human plasma and vegetable juice mixed at varying ratios)
- Known mixing ratios for each sample in the dilution series
- asari preferred_Feature_table.tsv (peak areas per sample)
- XCMS-generated feature table of peak areas per sample

## Outputs

- Matched feature set (~5,581 features common to asari and XCMS) keyed by m/z and retention time
- Per-feature Pearson correlation coefficients between peak areas and mixing ratios
- Binned correlation distributions (r from -1 to 1) for asari and XCMS, plotted side by side as in Fig. 7d

## How to apply

Process the BM21 mzML files with XCMS using its default peak-picking and correspondence workflow to obtain a feature table of peak areas per sample, and process the same files with asari in positive ionization mode with default 5 ppm m/z precision to obtain its preferred_Feature_table.tsv of peak areas. Match features between the two tools by m/z and retention time to recover the ~5,581 common features. For each matched feature, compute the Pearson correlation between peak areas across samples and the known mixing ratios of the dilution series. Bin the correlation values (r from -1 to 1) separately for each tool and compare the binned distributions side by side as in Fig. 7d. The rationale is that a faithful quantification should track the known composition gradient, so features whose peak areas correlate strongly with mixing ratios (r > 0.9) indicate more useful quantification.

## Related tools

- **asari** (Processes BM21 mzML files in positive mode with default 5 ppm m/z precision to produce preferred_Feature_table.tsv of peak areas) — github.com/shuzhao-li-lab/asari
- **XCMS** (Processes the same BM21 mzML files with default peak-picking and correspondence workflow to generate a comparison feature table of peak areas)
- **Python** (Used for feature matching by m/z and retention time, Pearson correlation computation, binning of r values, and plotting the Fig. 7d-style distributions)

## Examples

```
asari process --mode pos --input mydir/BM21_dir
```

## Evaluation signals

- After matching by m/z and RT, the recovered common feature count should be ~5,581 features between XCMS and asari in the BM21 dataset
- The binned correlation distribution should show more asari features in the r > 0.9 bin than XCMS, consistent with published Fig. 7d
- Each matched feature's Pearson r is computed against the known mixing ratios; values should span -1 to 1 across the binned histogram
- asari's preferred_Feature_table.tsv exists in the output project directory and contains peak areas per sample
- Correlation bins cover the full r range (-1 to 1) for both tools so the distributions are comparable

## Limitations

- Asari has a short history and has been mostly tested on Orbitrap platforms; settings need further optimization for other platforms
- The software does not work with low resolution data, restricting the quantification benchmark to high-resolution mzML input
- MS/MS spectra are ignored by asari in the default LC-MS workflow
- Borderline quantification cases are a tradeoff between sensitivity and true discovery rate

## Evidence

- [results] asari shows more features with Pearson correlation coefficient >0.9 to the mixing ratios: [evidence span withheld — anchor status: not_found]
- [abstract] Asari compares favorably to other tools in feature detection and quantification: "Asari compares favorably to other tools in feature detection and quantification"
- [readme] The recommended feature table is preferred_Feature_table.tsv.: "The recommended feature table is `preferred_Feature_table.tsv`."
- [readme] asari process --mode pos: "To process all mzML files under directory mydir/projectx_dir:

`asari process --mode pos --input mydir/projectx_dir`"
