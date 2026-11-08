---
name: pearson-correlation-of-peak-areas-to-mixing-ratios
description: Use when evaluating metabolomics peak-area quantification accuracy on
  a dilution/mixing series with known expected ratios, especially comparing two processing
  tools' feature tables.
when_to_use_negative:
- Input is already an annotated feature table without sample-level peak areas, or
  there is no known mixing/dilution series to correlate against.
- Data is low-resolution or from non-Orbitrap platforms without optimized asari settings.
- The goal is peak detection quality (feature counts) rather than quantification accuracy.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: asari
  role: Processes the BM21 mzML files (positive ionization mode, default 5 ppm m/z
    precision) to generate the preferred_Feature_table.tsv of peak areas being evaluated.
  repo: https://github.com/shuzhao-li-lab/asari
- name: XCMS
  role: Processes the same BM21 mzML files with its default peak-picking and correspondence
    workflow to produce the baseline feature table of peak areas.
- name: Python
  role: Used for matching features by m/z and retention time, computing Pearson correlations,
    binning r values, and plotting the binned distributions (Requires Python 3.8+
    for asari).
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:52e947e310b70e6c45359f0d76dc84939aba4aaa330bf4d732104f8736e65882
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/asari
  tools:
  - Python
  - XCMS
  - asari
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# pearson-correlation-of-peak-areas-to-mixing-ratios

## Summary

Quantification benchmark that computes Pearson correlation between LC-MS feature peak areas and known mixing ratios in a dilution series (BM21 plasma/vegetable juice), then bins the r values to compare asari vs XCMS feature tables.

## When to use

Use when evaluating metabolomics peak-area quantification accuracy on a dilution/mixing series with known expected ratios, especially comparing two processing tools' feature tables.

## When NOT to use

- Input is already an annotated feature table without sample-level peak areas, or there is no known mixing/dilution series to correlate against.
- Data is low-resolution or from non-Orbitrap platforms without optimized asari settings.
- The goal is peak detection quality (feature counts) rather than quantification accuracy.

## Inputs

- BM21 dilution-series mzML files (centroided; human plasma mixed with vegetable juice at varying ratios)
- Known per-sample mixing ratios of the BM21 dilution series
- XCMS feature table of peak areas per sample
- asari preferred_Feature_table.tsv of peak areas

## Outputs

- Per-feature Pearson correlation coefficients (r) to the mixing ratios for each tool
- Matched feature set between asari and XCMS (m/z and retention time matching)
- Binned correlation distributions (bins spanning r from -1 to 1) plotted side by side per tool (Fig. 7d-style)

## How to apply

Process the BM21 mzML files with XCMS (default peak-picking and correspondence) and with asari in positive ionization mode at default 5 ppm m/z precision, yielding peak-area feature tables (asari's preferred_Feature_table.tsv). Match features between tools by m/z and retention time to recover the common set (5,581 features in the BM21 dataset). For each matched feature, compute the Pearson correlation between per-sample peak areas and the known mixing ratios across the dilution series. Bin the correlation values (e.g. bins spanning r from -1 to 1) separately for each tool and plot the binned distributions side by side as in Fig. 7d; correctness is judged by asari having more features in the r > 0.9 bin than XCMS.

## Related tools

- **asari** (Processes the BM21 mzML files (positive ionization mode, default 5 ppm m/z precision) to generate the preferred_Feature_table.tsv of peak areas being evaluated.) — https://github.com/shuzhao-li-lab/asari
- **XCMS** (Processes the same BM21 mzML files with its default peak-picking and correspondence workflow to produce the baseline feature table of peak areas.)
- **Python** (Used for matching features by m/z and retention time, computing Pearson correlations, binning r values, and plotting the binned distributions (Requires Python 3.8+ for asari).)

## Examples

```
asari process --mode pos --input mydir/BM21_dir
```

## Evaluation signals

- Feature matching between asari and XCMS recovers 5,581 common features in the BM21 dataset.
- The binned correlation distribution shows more asari features in the r > 0.9 bin than XCMS, consistent with published Fig. 7d.
- Correlation values are binned over the full r range (-1 to 1) for both tools and plotted side by side.
- Each matched feature has a Pearson r computed across all samples of the dilution series against the known mixing ratios.
- asari output folder contains preferred_Feature_table.tsv (the recommended feature table).

## Limitations

- MS/MS spectra are ignored by asari's default LC-MS workflow.
- Asari has been mostly tested on Orbitrap platforms; settings need optimization for other platforms and it does not work with low-resolution data.
- Efficiency is not optimal on data from longer chromatography, and very high noise data may need customized processing.

## Evidence

- [abstract] Asari compares favorably to other tools in feature detection and quantification: "Asari compares favorably to other tools in feature detection and quantification"
- [readme] readme input format: [evidence span withheld — anchor status: external_doc]
- [readme] readme recommended table: "The recommended feature table is `preferred_Feature_table.tsv`."
- [other] task card finding: [evidence span withheld — anchor status: not_found]
- [other] asari ppm precision: [evidence span withheld — anchor status: not_found]
