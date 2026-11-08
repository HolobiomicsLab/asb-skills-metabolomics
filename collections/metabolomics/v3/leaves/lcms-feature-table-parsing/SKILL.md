---
name: lcms-feature-table-parsing
description: Use when feature tables from multiple LC-MS processing tools must be
  parsed and unified for cross-tool comparison or per-feature rescoring, e.g. recomputing
  peak-shape fitness scores on the Yeast2021 dataset.
when_to_use_negative:
- The analysis goal is annotation or pathway analysis on an already-curated feature
  table rather than cross-tool quality rescoring of raw tool outputs
- Input is raw (profile-mode or centroided-only) spectra without any tool-produced
  feature table to parse
- Data are low-resolution LC-MS, for which the asari-based scoring approach is not
  designed
tools:
- name: asari
  role: Source of the Yeast2021 feature table (preferred_Feature_table.tsv / full_Feature_table.tsv)
    and of the Gaussian peak-shape scoring function (peaks.evaluate_gaussian_peak_on_intensity_list)
    applied to the parsed features
  repo: https://github.com/shuzhao-li-lab/asari
- name: XCMS
  role: Source of a comparison feature table whose reported features are parsed and
    rescored for peak-shape fitness
- name: MZmine
  role: Source of two comparison feature tables (wavelet and local minimum peak detection)
    parsed and rescored alongside asari and XCMS outputs
- name: metDataModel
  role: Shared data models for peaks and features that can structure the unified per-feature
    representation across tools
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:cf4e865717a12743e1a451101082742decf1645f603591e1c7a7acc804a32dd6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/asari
  tools:
  - MZmine
  - XCMS
  - asari
  - metDataModel
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# lcms-feature-table-parsing

## Summary

Loads and normalizes LC-MS feature tables produced by different processing tools (asari, XCMS, MZmine wavelet and MZmine local minimum) into a common per-feature structure so downstream per-feature quality metrics, such as Gaussian peak-shape fitness on Yeast2021, can be recomputed and compared across tools.

## When to use

Use when feature tables from multiple LC-MS processing tools must be parsed and unified for cross-tool comparison or per-feature rescoring, e.g. recomputing peak-shape fitness scores on the Yeast2021 dataset.

## When NOT to use

- The analysis goal is annotation or pathway analysis on an already-curated feature table rather than cross-tool quality rescoring of raw tool outputs
- Input is raw (profile-mode or centroided-only) spectra without any tool-produced feature table to parse
- Data are low-resolution LC-MS, for which the asari-based scoring approach is not designed

## Inputs

- asari preferred_Feature_table.tsv (or export/full_Feature_table.tsv) from a Yeast2021 asari project directory
- XCMS feature table on Yeast2021
- MZmine feature tables (wavelet peak detection and local minimum peak detection) on Yeast2021
- Centroided mzML files underlying the feature tables, for EIC extraction over padded peak regions

## Outputs

- Unified per-tool feature tables with m/z, retention time, peak boundaries and per-sample intensities
- Per-feature EIC intensity lists (3 s padding each side of reported peak boundaries)
- Per-feature Gaussian peak-shape fitness scores and per-tool bad-peak counts (fitness < 0.5) vs good-peak counts (fitness > 0.5)

## How to apply

Load each tool's exported feature table separately — for asari use preferred_Feature_table.tsv (or export/full_Feature_table.tsv when all snr/shape-passing peaks are needed), and the equivalent XCMS and MZmine (wavelet and local minimum peak detection) outputs on Yeast2021. Map the tool-specific columns into a common schema of m/z, retention time, peak boundaries and per-sample intensities, since each tool reports feature correspondence and consensus m/z differently. For each reported feature, extract the EIC intensity list over the peak region padded by 3 s on both left and right of the reported peak boundaries from the mzML-derived data, then fit a Gaussian model (e.g. via asari's peaks.evaluate_gaussian_peak_on_intensity_list) to obtain the peak-shape fitness score per feature. Flag features with fitness score < 0.5 as bad peaks and count them per tool, so the inflated feature counts of some tools can be attributed to low-quality, non-reproducible peaks rather than better coverage. Preserve tool provenance per row so bad/good peak counts can be tabulated per tool and compared against published counts.

## Related tools

- **asari** (Source of the Yeast2021 feature table (preferred_Feature_table.tsv / full_Feature_table.tsv) and of the Gaussian peak-shape scoring function (peaks.evaluate_gaussian_peak_on_intensity_list) applied to the parsed features) — https://github.com/shuzhao-li-lab/asari
- **XCMS** (Source of a comparison feature table whose reported features are parsed and rescored for peak-shape fitness)
- **MZmine** (Source of two comparison feature tables (wavelet and local minimum peak detection) parsed and rescored alongside asari and XCMS outputs)
- **metDataModel** (Shared data models for peaks and features that can structure the unified per-feature representation across tools) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
python3 -c "import pandas as pd; t = pd.read_csv('preferred_Feature_table.tsv', sep='\t'); print(t.shape)"
```

## Evaluation signals

- Recomputed bad-peak counts (fitness < 0.5, 3 s padding) must match 504 (asari), 1243 (XCMS), 4035 (MZmine wavelets) and 9436 (MZmine local minimum) on Yeast2021
- Good-peak counts (fitness > 0.5) should be stable at approximately 5000 across all four tools
- Parsed row counts per table must equal each tool's reported feature count on Yeast2021
- Extracted EIC length must correspond to the reported peak boundaries plus 3 s padding on both left and right
- All fitness scores fall in a valid [0, 1] goodness-of-fit range and every feature carries its tool-of-origin label

## Limitations

- Tool-specific output schemas differ (asari TSV vs XCMS and MZmine exports), so column mapping is required and mis-mapped peak-boundary columns will corrupt the 3 s padded EIC extraction
- MS/MS spectra are ignored by asari's default LC-MS workflow, so only MS1 feature information is available for parsing
- The widely used wavelet algorithm shows no advantage in this study and adds unnecessary complexity, so MZmine wavelet feature counts carry many low-quality peaks (4035 bad) that must not be interpreted as better coverage

## Evidence

- [readme] All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards: "All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards"
- [methods] peaks.evaluate_gaussian_peak_on_intensity_list: "[peaks.evaluate_gaussian_peak_on_intensity_list](peaks.evaluate_gaussian_peak_on_intensity_list)"
- [discussion] Wavelet algorithm shows no advantage: "The widely used wavelet algorithm shows no advantage in this study, but costs unnecessary complexity."
- [readme] preferred_Feature_table.tsv recommended: "The recommended feature table is `preferred_Feature_table.tsv`."
- [abstract] Inconsistency attributed to mass alignment and feature quality: "Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality"
