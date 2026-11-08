---
name: xic-based-feature-assessment
description: Use when you have an MZmine-preprocessed full-scan feature table (.csv
  with row ID, row m/z, row retention time, peak area) and need to statistically filter
  which features are real chromatographic signals worth targeting for MS/MS fragmentation.
when_to_use_negative:
- The input is MS/MS or DDA data rather than an MZmine full-scan MS1 feature table
  — RegFilter is applied to full-scan mode features to build SPLs for targeted fragmentation
- Features are already filtered or curated (e.g., coming from a scheduled precursor
  list) and no regression-based XIC assessment is needed
- The feature table lacks peak-area chromatographic information required for XIC extraction
  and linear regression fitting
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: RegFilter
  role: Python package performing the XIC-based linear regression filtering of full-scan
    features via building_spl_and_assessing_XIC_for_all_ions.ipynb
  repo: https://github.com/computational-chemical-biology/regression_filter
- name: MZmine 2.53
  role: Preprocessing of raw LC-MS full-scan data (mass detection, ADAP chromatogram
    building, deconvolution, isotopic grouping, join aligner, peak finder) before
    CSV export for RegFilter
- name: MSConvert
  role: Converts raw MS vendor files to .mzXML format prior to MZmine preprocessing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:a5467202ef6d7bb03519a4955282f9774787747358fd43bdaf1522426ce03368
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
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# xic-based-feature-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assesses MZmine full-scan MS1 features by fitting linear regressions to their extracted ion chromatograms (XICs) and retaining only features meeting objective statistical criteria (R2 >= 0.9 with negative slope) using the RegFilter Python package, to build a curated scheduled precursor list (SPL).

## When to use

Use when you have an MZmine-preprocessed full-scan feature table (.csv with row ID, row m/z, row retention time, peak area) and need to statistically filter which features are real chromatographic signals worth targeting for MS/MS fragmentation.

## When NOT to use

- The input is MS/MS or DDA data rather than an MZmine full-scan MS1 feature table — RegFilter is applied to full-scan mode features to build SPLs for targeted fragmentation
- Features are already filtered or curated (e.g., coming from a scheduled precursor list) and no regression-based XIC assessment is needed
- The feature table lacks peak-area chromatographic information required for XIC extraction and linear regression fitting

## Inputs

- MZmine 2.53 full-scan MS1 feature table in .csv format (columns: row ID, row m/z, row retention time, peak area) for each sample/strain

## Outputs

- Filtered .csv file containing only features meeting the regression criteria (R2 >= 0.9, negative slope), with their retention times, exported to the notebooks directory
- Report stating the number of total features and the number of filtered (selected) features per sample

## How to apply

Preprocess raw LC-MS data with MZmine 2.53 (m/z tolerance 0.001 m/z or 15 ppm, RT tolerance 0.2 min) and export the full-scan feature table to .csv. Install the RegFilter package from github.com/computational-chemical-biology/regression_filter in a dedicated conda environment (regfilter_env.yml), then open the notebook building_spl_and_assessing_XIC_for_all_ions.ipynb, point it to the input .csv path, and run the RegFilter algorithm setting the number of points via the npts= argument for the XIC-based assessment. RegFilter evaluates each feature by linear regression over its extracted ion chromatogram, keeping only features satisfying the desired criteria (R2 >= 0.9 with negative slope). The filtering run exports a new .csv with only the selected features (with their retention times) to the notebooks directory, and a report stating the number of total and filtered features.

## Related tools

- **RegFilter** (Python package performing the XIC-based linear regression filtering of full-scan features via building_spl_and_assessing_XIC_for_all_ions.ipynb) — https://github.com/computational-chemical-biology/regression_filter
- **MZmine 2.53** (Preprocessing of raw LC-MS full-scan data (mass detection, ADAP chromatogram building, deconvolution, isotopic grouping, join aligner, peak finder) before CSV export for RegFilter)
- **MSConvert** (Converts raw MS vendor files to .mzXML format prior to MZmine preprocessing)

## Examples

```
pip install git+https://github.com/computational-chemical-biology/regression_filter.git && jupyter notebook building_spl_and_assessing_XIC_for_all_ions.ipynb  # set csv path and npts=, run RegFilter cells
```

## Evaluation signals

- Exported .csv contains the filtered features with their respective retention times
- Report of filtered-feature counts matches article values: 135 features for BRA006, 219 for BRA010, and 150 for BRA177 (9.3%, 9.9%, and 9.0% recovery of full-scan features)
- All retained features satisfy the RegFilter linear regression criteria (R2 >= 0.9, negative slope)
- Input feature tables were produced with the specified MZmine 2.53 parameters (m/z tolerance 0.001 m/z or 15 ppm, RT tolerance 0.2 min), with columns row ID, row m/z, row retention time, peak area

## Limitations

- The number of retained features depends on the npts= (number of points) setting chosen for the XIC-based assessment; this parameter must be set by the user
- Filtering stringency (R2 >= 0.9, negative slope) may discard genuine features with noisy chromatographic shapes; in the article only ~9-10% of full-scan features passed
- RegFilter is semi-automated and requires manual execution of notebook cells and manual specification of the input .csv path

## Evidence
