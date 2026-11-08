---
name: feature-table-mz-rounding
description: Use when comparing LC-MS feature tables from different processing tools
  (XCMS, MZmine, asari) and you need to quantify whether reported m/z values remain
  distinguishable at the 5 ppm instrument resolution after rounding to 3 decimal places,
  e.
when_to_use_negative:
- Input is raw centroided mzML data rather than a processed feature table — the metric
  operates on reported per-feature m/z values
- Data are from a low-resolution instrument — the paper notes 'The software does not
  work with low resolution data', and the 5 ppm distinctness criterion is meaningless
  there
- The feature table was produced by asari itself and the goal is not benchmarking
  — asari features should cluster near mSelectivity 1 by design, so this skill adds
  no information
tools:
- name: asari
  role: Produces the reference feature table (preferred_Feature_table.tsv) whose mSelectivity
    scores should cluster near 1; its mass-track design underlies the 3-decimal-place
    rounding model
  repo: https://github.com/shuzhao-li/asari
- name: XCMS
  role: Source of the benchmark feature table whose mSelectivity distribution is expected
    to be poor
- name: MZmine
  role: Source of the benchmark feature table whose mSelectivity distribution is expected
    to be poor
- name: Python
  role: Implementation language for rounding m/z values, computing 5 ppm nearest-neighbor
    mSelectivity, and plotting the distributions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:050fe986bb455fe0408bac66b501d019b93ccb7900232387f75687d57effc0a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - MZmine
  - Python
  - XCMS
  - asari
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# feature-table-mz-rounding

## Summary

A quality-metric workflow that rounds feature-table m/z values to the 3rd decimal place (mass-track granularity) and computes an mSelectivity score measuring how distinguishable each m/z is from its nearest neighbors within the 5 ppm instrument resolution, exposing poor mass selectivity in XCMS/MZmine feature tables relative to asari.

## When to use

Use when comparing LC-MS feature tables from different processing tools (XCMS, MZmine, asari) and you need to quantify whether reported m/z values remain distinguishable at the 5 ppm instrument resolution after rounding to 3 decimal places, e.g. to reproduce a Fig. 1d-style mSelectivity distribution contrast.

## When NOT to use

- Input is raw centroided mzML data rather than a processed feature table — the metric operates on reported per-feature m/z values
- Data are from a low-resolution instrument — the paper notes 'The software does not work with low resolution data', and the 5 ppm distinctness criterion is meaningless there
- The feature table was produced by asari itself and the goal is not benchmarking — asari features should cluster near mSelectivity 1 by design, so this skill adds no information

## Inputs

- XCMS feature table for the HZV029 dataset (m/z values per feature)
- MZmine feature table for the HZV029 dataset (m/z values per feature)
- asari preferred_Feature_table.tsv (reference features from export/full_Feature_table.tsv directory structure)
- Instrument mass-resolution tolerance (5 ppm)

## Outputs

- Per-feature mSelectivity scores for each tool's feature table
- Side-by-side mSelectivity distributions (histogram/violin) reproducing the Fig. 1d pattern
- Count of features with poor mSelectivity per tool

## How to apply

Load the XCMS and MZmine feature tables for the dataset (e.g. HZV029) and the asari preferred_Feature_table.tsv as the reference. Round every feature's m/z value to the 3rd decimal place to mimic the mass-track m/z granularity of the asari preprocessing approach. For each feature, compute mSelectivity as the distinctness of its m/z measurement relative to the closest neighboring m/z values within the 5 ppm mass-resolution tolerance (i.e. convert nearest-neighbor m/z gaps into ppm differences and flag features whose neighbors fall within tolerance). Plot the mSelectivity distributions of the three tools side by side as histograms or violin plots. The rationale is that, at modern high-resolution (e.g. Orbitrap) data, rounding to the millithousandth bin should still leave every feature distinguishable; low scores indicate that the tool's mass alignment and feature correspondence produced indistinguishable m/z values.

## Related tools

- **asari** (Produces the reference feature table (preferred_Feature_table.tsv) whose mSelectivity scores should cluster near 1; its mass-track design underlies the 3-decimal-place rounding model) — https://github.com/shuzhao-li/asari
- **XCMS** (Source of the benchmark feature table whose mSelectivity distribution is expected to be poor)
- **MZmine** (Source of the benchmark feature table whose mSelectivity distribution is expected to be poor)
- **Python** (Implementation language for rounding m/z values, computing 5 ppm nearest-neighbor mSelectivity, and plotting the distributions)

## Examples

```
mz_r = [round(m, 3) for m in feature_mzs]; sel = [min(abs(b-a)/a*1e6 for b in set(mz_r) if b != a) <= 5 for a in mz_r]
```

## Evaluation signals

- XCMS and MZmine mSelectivity distributions show poor (low) distinctness values across a very large number of features
- asari-derived feature distribution clusters near 1 (perfect compliance of mass resolution), matching the Fig. 1d contrast
- Rounded m/z values collide with neighbors within the 5 ppm tolerance only in the XCMS/MZmine tables, not in the asari table
- The m/z rounding step uses exactly 3 decimal places, consistent with the mass-track m/z granularity described in the paper

## Limitations

- Interpretation depends on the 5 ppm instrument resolution assumption; a different instrument resolution changes the tolerance used for mSelectivity
- Inconsistency among tools is attributed by the paper to deficiencies of mass alignment and feature quality controls, so this metric reflects alignment quality rather than feature-detection sensitivity
- ppm is only a practical approximation; the paper notes a 2nd order polynomial model may be required to cover the full m/z range more accurately

## Evidence

- [results] m/z values not distinguishable at the 5 ppm instrument resolution: [evidence span withheld — anchor status: not_found]
- [readme] The recommended feature table is `preferred_Feature_table.tsv`.: "The recommended feature table is `preferred_Feature_table.tsv`."
- [readme] Tracking peak quality, selectiviy metrics on m/z, chromatography and annotation databases: "Tracking peak quality, selectiviy metrics on m/z, chromatography and annotation databases"
- [abstract] Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality: "Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality"
- [results] all features have mSelectivity close to 1: [evidence span withheld — anchor status: not_found]
