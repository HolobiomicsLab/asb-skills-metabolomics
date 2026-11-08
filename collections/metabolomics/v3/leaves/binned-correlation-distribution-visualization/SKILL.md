---
name: binned-correlation-distribution-visualization
description: Use when a dilution or mixing-series dataset with known composition fractions
  is available and you must compare quantification fidelity of LC-MS feature tables
  from two processing tools (e.g., asari vs XCMS).
when_to_use_negative:
- The dataset has no known mixing ratios or dilution design — correlation against
  composition is then undefined.
- Input is already a feature table from only one tool; cross-tool matching by m/z
  and retention time requires both tables.
- Low-resolution data or non-Orbitrap platforms where asari's 5 ppm m/z precision
  settings are not appropriate.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: asari
  role: Processes the BM21 mzML files (positive mode, default 5 ppm m/z precision)
    to produce preferred_Feature_table.tsv peak areas
  repo: https://github.com/shuzhao-li/asari
- name: XCMS
  role: Produces the comparison peak-area feature table via its default peak-picking
    and correspondence workflow on the same mzML files
- name: Python
  role: Used to match features by m/z and retention time, compute Pearson correlations,
    bin r values, and plot the distributions (requires Python 3.8+)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:d2475cfd3c054232c3532008c900884b7a953fdaf0a6a4cf7ba57b03669ced29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - Python
  - XCMS
  - asari
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# binned-correlation-distribution-visualization

## Summary

A quantification-quality evaluation that computes Pearson correlations between feature peak areas and known sample mixing ratios, then bins the r values (e.g., from -1 to 1) to compare how many features each tool tracks faithfully, as in Fig. 7d of the asari benchmark.

## When to use

Use when a dilution or mixing-series dataset with known composition fractions is available and you must compare quantification fidelity of LC-MS feature tables from two processing tools (e.g., asari vs XCMS).

## When NOT to use

- The dataset has no known mixing ratios or dilution design — correlation against composition is then undefined.
- Input is already a feature table from only one tool; cross-tool matching by m/z and retention time requires both tables.
- Low-resolution data or non-Orbitrap platforms where asari's 5 ppm m/z precision settings are not appropriate.

## Inputs

- mzML files of a dilution/mixing series (e.g., BM21 plasma-vegetable-juice ratios)
- asari preferred_Feature_table.tsv of peak areas per sample
- XCMS feature table of peak areas per sample
- vector of known mixing ratios per sample

## Outputs

- table of matched features with Pearson r to mixing ratios per tool
- binned correlation distributions (r from -1 to 1) per tool
- side-by-side plot of binned distributions (Fig. 7d analogue)

## How to apply

Process the dilution-series mzML files (e.g., the BM21 human plasma/vegetable juice mixing experiment) with each tool to obtain per-sample peak-area feature tables — asari's preferred_Feature_table.tsv and XCMS's correspondence output. Match features between the tools by m/z and retention time to isolate common features (5,581 in the BM21 dataset). For each matched feature, compute the Pearson correlation between peak areas across samples and the known mixing ratios. Bin the r values (spanning -1 to 1) separately per tool and plot the binned distributions side by side as in Fig. 7d. The rationale: a tool whose quantification tracks true composition will show a heavier tail of features in the highest-correlation bin (r > 0.9), which is how asari was judged more useful than XCMS.

## Related tools

- **asari** (Processes the BM21 mzML files (positive mode, default 5 ppm m/z precision) to produce preferred_Feature_table.tsv peak areas) — https://github.com/shuzhao-li/asari
- **XCMS** (Produces the comparison peak-area feature table via its default peak-picking and correspondence workflow on the same mzML files)
- **Python** (Used to match features by m/z and retention time, compute Pearson correlations, bin r values, and plot the distributions (requires Python 3.8+))

## Examples

```
asari process --mode pos --input BM21_mzml_dir -o BM21_asari
```

## Evaluation signals

- Feature matching by m/z and retention time recovers approximately 5,581 common features in the BM21 dataset
- The binned correlation distribution shows more asari features in the r > 0.9 bin than XCMS, reproducing Fig. 7d
- All Pearson correlations fall within the binned range -1 to 1 and each feature appears in exactly one bin
- Correlations are computed only across samples with known mixing ratios, using peak areas (not peak heights) from each tool's table

## Limitations

- Asari settings are mostly tested on Orbitrap platforms and need optimization for others; the software does not work with low-resolution data
- Asari's borderline feature detection is a tradeoff between sensitivity and true discovery rate, which affects which features enter the correlation analysis
- Comparison depends on cross-tool feature matching by m/z and retention time, so errors in correspondence grouping propagate into the binned counts

## Evidence

- [results] Of the 5,581 features matched between XCMS and asari in the BM21 dataset, asari shows more features with Pearson correlation coefficient >0.9 to the mixing ratios, indicating its quantification is more useful than XCMS's.: [evidence span withheld — anchor status: not_found]
- [abstract] Asari compares favorably to other tools in feature detection and quantification: "Asari compares favorably to other tools in feature detection and quantification"
- [readme] The recommended feature table is preferred_Feature_table.tsv.: "The recommended feature table is `preferred_Feature_table.tsv`."
- [discussion] Asari has a short history and has been mostly tested on the Orbitrap platforms.: "Asari has a short history and has been mostly tested on the Orbitrap platforms."
