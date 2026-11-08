---
name: regression-based-ion-filtering
description: Use when you have an MZmine full-scan MS1 feature table (.
when_to_use_negative:
- Input is already an MS/MS-only feature table or spectra list rather than a full-scan
  MS1 feature table exported from MZmine - RegFilter filters full-scan-mode feature
  tables.
- You need unsupervised statistical filtering of multiple sample groups/conditions
  rather than ion selection for targeted re-injection; RegFilter is a semi-automated
  linear-regression filter over XIC points, not a general feature-differential-analysis
  tool.
- Data have not been preprocessed by MZmine (or an equivalent aligner exporting the
  expected CSV columns), so no aligned feature table with row ID/m/z/RT/peak area
  exists to filter.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: RegFilter
  role: Performs the linear regression filtering (R2 >= 0.9, negative slope) over
    MZmine full-scan feature tables and exports the scheduled precursor list .csv
  repo: https://github.com/computational-chemical-biology/regression_filter
- name: MZmine 2.53
  role: Preprocesses .mzXML full-scan data (mass detection, ADAP chromatogram building,
    baseline cut-off deconvolution, isotopic grouping, Join aligner, Peak finder)
    and exports the feature table that RegFilter consumes
- name: MSConvert
  role: Converts raw MS files to .mzXML format prior to MZmine preprocessing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:b4b5010bed44eb50f4a69a6b6e21ce6e3a8e6b0760e833fe50b69a2aab363944
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/computational-chemical-biology/regression_filter
  repo_url: https://github.com/computational-chemical-biology/regression_filter
  tools:
  - MSConvert
  - MZmine
  - MZmine 2.53
  - RegFilter
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# regression-based-ion-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

RegFilter is a semi-automated feature selection package that runs a linear regression filter over MZmine-derived full-scan MS1 feature tables, selecting ions with R2 >= 0.9 and a negative slope so that biological signals missed in DDA fragmentation can be rationally targeted.

## When to use

Use when you have an MZmine full-scan MS1 feature table (.csv with row ID, m/z, RT, peak area) from an untargeted metabolomics run in DDA mode and need to rationally move to a targeted approach by selecting which ions to schedule for fragmentation, because not all detected ions are fragmented in DDA.

## When NOT to use

- Input is already an MS/MS-only feature table or spectra list rather than a full-scan MS1 feature table exported from MZmine - RegFilter filters full-scan-mode feature tables.
- You need unsupervised statistical filtering of multiple sample groups/conditions rather than ion selection for targeted re-injection; RegFilter is a semi-automated linear-regression filter over XIC points, not a general feature-differential-analysis tool.
- Data have not been preprocessed by MZmine (or an equivalent aligner exporting the expected CSV columns), so no aligned feature table with row ID/m/z/RT/peak area exists to filter.

## Inputs

- MZmine-preprocessed full-scan MS1 feature table in .csv format (columns: row ID, row m/z, row retention time, peak area) exported from MZmine 2.53 (e.g. after Join aligner and Peak finder)
- raw LC-MS files (.mzXML, converted from vendor format by MSConvert) if preprocessing must be redone first

## Outputs

- New .csv file exported to the notebooks directory containing only features that meet the filtering criteria (R2 >= 0.9, negative slope), with retention times included
- Report of number of total features and number of filtered (selected) features for each input table
- Scheduled precursor list (SPL) for targeted MS/MS re-acquisition

## How to apply

First preprocess raw MS files with MSConvert to .mzXML, then run MZmine 2.53 preprocessing in full-scan mode separately from MS/MS mode, using Centroid mass detection (noise level 1E3 for MS1, 1E2 for MS2), ADAP chromatogram building (min group size 4, m/z tolerance 0.001 m/z or 15 ppm), baseline cut-off deconvolution (min peak height 3E3), isotopic peak grouping and Join aligner (RT tolerance 0.2), and export the aligned feature table to CSV with row ID, row m/z, and row retention time. Open the notebook building_spl_and_assessing_XIC_for_all_ions.ipynb, point it to the input .csv, and run the RegFilter filtering algorithm, setting the number of points via the npts= argument for XIC-based assessment of each feature. The linear regression filter keeps features satisfying R2 >= 0.9 with a negative slope and exports a new .csv containing only the selected features plus their retention times to the notebooks directory, which serves as a scheduled precursor list (SPL) for targeted re-injection. The rationale is to cover the entire chromatographic run and select low-abundance ions that DDA's stochastic nature may skip, then produce a report of total vs. filtered feature counts.

## Related tools

- **RegFilter** (Performs the linear regression filtering (R2 >= 0.9, negative slope) over MZmine full-scan feature tables and exports the scheduled precursor list .csv) — https://github.com/computational-chemical-biology/regression_filter
- **MZmine 2.53** (Preprocesses .mzXML full-scan data (mass detection, ADAP chromatogram building, baseline cut-off deconvolution, isotopic grouping, Join aligner, Peak finder) and exports the feature table that RegFilter consumes)
- **MSConvert** (Converts raw MS files to .mzXML format prior to MZmine preprocessing)

## Examples

```
conda activate regfilter && python notebooks/building_spl_and_assessing_XIC_for_all_ions.ipynb --input path/to/features.csv
```

## Evaluation signals

- The exported .csv contains the filtered features with their respective retention times and lives in the notebooks directory of the installed package.
- Reported selected-feature counts match the article's reference values (135 for BRA006, 219 for BRA010, 150 for BRA177) when the same MZmine parameters and npts are used.
- The number of filtered features is much smaller than the total features, with biological-signal recovery percentages on the order of 9.3%, 9.9%, and 9.0% reported for BRA006, BRA010, and BRA177.
- The SPL selected by RegFilter overlaps with DDA-fragmented precursors (76, 135, and 77 features for BRA006, BRA010, and BRA177, respectively) - substantially more selected ions should be present in DDA MS1 than were actually fragmented.
- The filtering retained ions with low m/z and low relative abundance, confirming the rationale of covering the entire chromatographic run instead of DDA's stochastic ion selection.

## Limitations

- RegFilter is semi-automated and requires a notebook-driven workflow (npts= argument must be set manually); no fully scripted one-liner is provided.
- Filtering depends on MZmine preprocessing parameters (noise levels, m/z tolerance 0.001 m/z or 15 ppm, RT tolerance 0.2); changing these changes the input feature table and hence the selected set.
- The article does not provide a DOI or Zenodo identifier for the RegFilter package itself beyond the Zenodo-archived batch mode file, limiting citability of the tool version.
- Negative-slope criterion and R2 >= 0.9 threshold were validated on three actinomycete extracts; extrapolation to other sample types or chromatographic setups is not established in this paper.

## Evidence

- [readme] README: access the notebook building_spl_and_assessing_XIC_for_all_ions.ipynb: [evidence span withheld — anchor status: not_found]
