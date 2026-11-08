---
name: feature-count-reproducibility-assessment
description: Use when the same raw LC-MS dataset (e.g., pooled human plasma mzML files)
  must be processed by two or more tools to compare feature counts and cross-tool
  feature correspondence, or to benchmark a new processing tool against XCMS/MZmine.
when_to_use_negative:
- Input is already a processed feature table rather than raw centroided mzML files
  — re-processing to compare tool behavior is not possible
- The goal is metabolite annotation or pathway analysis rather than comparing feature
  detection consistency across tools
- Data are low-resolution or from non-Orbitrap platforms without re-optimized settings
  (asari, the benchmarked tool, was mostly tested on Orbitrap platforms and does not
  work with low-resolution data)
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: XCMS
  role: Reference processing tool whose peak-picking and correspondence parameters
    generate one feature table for the count comparison
- name: MZmine
  role: Second reference processing tool whose peak detection and deconvolution modules
    generate the comparison feature table
- name: asari
  role: Trackable Python processing tool used to produce an alternative feature table
    (preferred_Feature_table.tsv) for benchmarking feature detection against XCMS
    and MZmine
  repo: https://github.com/shuzhao-li/asari
- name: ThermoRawFileParser
  role: Converts Thermo .RAW files to .mzML so both tools process identical centroided
    input
  repo: https://github.com/compomics/ThermoRawFileParser
- name: msConvert
  role: ProteoWizard converter handling most vendor data formats and .mzXML files
    into centroided mzML
  repo: https://proteowizard.sourceforge.io/tools.shtml
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:563116fbe7633b6253bdc5d49ff0a5195bd741b74ee3c3b67671204b1e5298b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - MZmine
  - ThermoRawFileParser
  - XCMS
  - asari
  - msConvert
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# feature-count-reproducibility-assessment

## Summary

Quantifies how consistently different LC-MS data processing tools (e.g., XCMS, MZmine, asari) detect and correspond features on the same dataset, by producing parallel feature tables and comparing feature counts and match rates.

## When to use

Use when the same raw LC-MS dataset (e.g., pooled human plasma mzML files) must be processed by two or more tools to compare feature counts and cross-tool feature correspondence, or to benchmark a new processing tool against XCMS/MZmine.

## When NOT to use

- Input is already a processed feature table rather than raw centroided mzML files — re-processing to compare tool behavior is not possible
- The goal is metabolite annotation or pathway analysis rather than comparing feature detection consistency across tools
- Data are low-resolution or from non-Orbitrap platforms without re-optimized settings (asari, the benchmarked tool, was mostly tested on Orbitrap platforms and does not work with low-resolution data)

## Inputs

- 184 pooled human plasma mzML files from the HZV029 study (or any shared centroided mzML dataset)
- Thermo .RAW or other vendor raw files requiring conversion to mzML
- Tool-specific processing parameters (peak-picking/correspondence settings for XCMS; peak detection/deconvolution modules for MZmine; asari mode and workflow flags)

## Outputs

- One feature table per processing tool (e.g., XCMS feature rows, MZmine feature rows, asari preferred_Feature_table.tsv)
- Summary table of feature counts per tool
- Cross-tool feature correspondence statistics (fraction of features unambiguously matched)

## How to apply

Obtain the identical raw dataset (e.g., 184 pooled human plasma samples from the HZV029 study) and convert vendor files to centroided mzML using ThermoRawFileParser (for Thermo .RAW) or msConvert (for other vendor formats and .mzXML). Process each file set independently with each tool under comparable peak-picking/correspondence parameters — e.g., XCMS peak-picking and correspondence parameters, MZmine peak detection and deconvolution modules, or `asari process` — to generate one feature table per tool. Count feature rows in each table and write the counts to a summary table; additionally, match features across tools by m/z and retention time to compute the fraction unambiguously matched (in the reference study only ~60% of XCMS features matched MZmine). Validate against published reference values (10,901 XCMS features vs 42,099 MZmine features on HZV029), accepting minor deviation from software version drift. The rationale is that inconsistency among tools is attributed to deficiencies in mass alignment and feature quality controls, so count discrepancies are themselves diagnostic signal.

## Related tools

- **XCMS** (Reference processing tool whose peak-picking and correspondence parameters generate one feature table for the count comparison)
- **MZmine** (Second reference processing tool whose peak detection and deconvolution modules generate the comparison feature table)
- **asari** (Trackable Python processing tool used to produce an alternative feature table (preferred_Feature_table.tsv) for benchmarking feature detection against XCMS and MZmine) — https://github.com/shuzhao-li/asari
- **ThermoRawFileParser** (Converts Thermo .RAW files to .mzML so both tools process identical centroided input) — https://github.com/compomics/ThermoRawFileParser
- **msConvert** (ProteoWizard converter handling most vendor data formats and .mzXML files into centroided mzML) — https://proteowizard.sourceforge.io/tools.shtml

## Examples

```
asari process --mode pos --input mydir/projectx_dir
```

## Evaluation signals

- Feature counts match the published reference values (10,901 XCMS features and 42,099 MZmine features on HZV029, Fig. 1a / Supplementary Table 1) within tolerance for software version drift
- Each tool's feature table has consistent columns (m/z, retention time, per-sample intensities) and one row per feature
- Cross-tool match rate is computed with an explicit m/z and retention-time tolerance and reported (reference: ~60% of XCMS features unambiguously matched to MZmine)
- The same mzML file set (same file count, e.g., 184 samples) was fed to every tool, with conversion provenance recorded
- Counts are recorded in a summary table rather than reported ad hoc, enabling direct comparison across reruns

## Limitations

- Software version drift can cause counts to deviate from published reference values; minor deviation is acceptable
- Inconsistency among tools is attributed to deficiencies in mass alignment and controls of feature quality, so counts alone do not identify which tool is 'correct'
- Borderline feature cases are a tradeoff between sensitivity and true discovery rate, affecting whether low-quality features are counted
- Asari settings need optimization for non-Orbitrap platforms, the software does not work with low-resolution data, and efficiency is not optimal on longer chromatography — limiting cross-platform benchmarking

## Evidence

- [methods] Counts match paper reference values: [evidence span withheld — anchor status: not_found]
- [methods] Published reference counts: [evidence span withheld — anchor status: not_found]
- [results] Correspondence discrepancy finding: [evidence span withheld — anchor status: not_found]
- [abstract] Root cause of inconsistency: "Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality"
- [readme] Format conversion requirement: "We use ThermoRawFileParser (https://github.com/compomics/ThermoRawFileParser) to convert Thermo .RAW files to .mzML. Msconvert in ProteoWizard"
- [readme] Asari output feature table: "The recommended feature table is `preferred_Feature_table.tsv`."
