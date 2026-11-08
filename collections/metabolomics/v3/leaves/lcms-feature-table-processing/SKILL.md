---
name: lcms-feature-table-processing
description: Use when raw LC-MS(/MS) files from a serially diluted sample set must
  be preprocessed in MZmine into aligned full-scan feature tables (.csv with row ID,
  row m/z, row retention time, peak areas) suitable for downstream regression-based
  feature selection.
when_to_use_negative:
- The input is already an aligned MZmine .csv feature table — skip preprocessing and
  proceed directly to the RegFilter filtering step
- The data are targeted MS/MS acquisitions with no serial dilution series, so linear-regression
  filtering across dilution points is not meaningful
- Only DDA-mode data without a full-scan feature table are available, since the filtering
  pool requires the full-scan features detected in full scan mode
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: MSConvert
  role: Converts raw MS files to .mzXML format as the first step before MZmine preprocessing
- name: MZmine 2.53
  role: 'Performs full-scan and MS/MS preprocessing: mass detection, ADAP chromatogram
    building, chromatogram deconvolution, isotopic grouping, join alignment, peak
    finding, and CSV export'
- name: RegFilter
  role: Consumes the MZmine .csv feature table and filters features by linear regression
    (R2 >= 0.9) over the dilution series via the building_spl_and_assessing_XIC_for_all_ions.ipynb
    notebook
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:7c53ed2bfd17da566664dbbe7feb67ae22f7b4d579a915c1baffc7ece29ceb7b
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# lcms-feature-table-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts raw LC-MS files to .mzXML, preprocesses full-scan and MS/MS data in MZmine 2.53 (mass detection, ADAP chromatogram building, deconvolution, isotopic grouping, join alignment, gap filling), and exports aligned feature tables (.csv) that feed the RegFilter dilution-series filter.

## When to use

Use when raw LC-MS(/MS) files from a serially diluted sample set must be preprocessed in MZmine into aligned full-scan feature tables (.csv with row ID, row m/z, row retention time, peak areas) suitable for downstream regression-based feature selection.

## When NOT to use

- The input is already an aligned MZmine .csv feature table — skip preprocessing and proceed directly to the RegFilter filtering step
- The data are targeted MS/MS acquisitions with no serial dilution series, so linear-regression filtering across dilution points is not meaningful
- Only DDA-mode data without a full-scan feature table are available, since the filtering pool requires the full-scan features detected in full scan mode

## Inputs

- Raw LC-MS/MS files from serially diluted actinomycete organic extracts (BRA006, BRA010, BRA177) injected at 1.0, 0.5, 0.25, 0.125, and 0.0625 mg/mL
- .mzXML files converted from raw MS data with MSConvert
- MZmine-preprocessed full-scan feature table in .csv format (columns: row ID, row m/z, row retention time, peak area)

## Outputs

- MZmine-exported .csv full-scan feature table with row ID, row m/z, row retention time, and per-sample peak areas
- GNPS-FBMN-ready export containing only rows with MS2
- RegFilter-filtered .csv of features meeting the R2 >= 0.9 negative-slope criterion, with their retention times, for scheduled precursor list (SPL) construction

## How to apply

Convert raw MS files to .mzXML with MSConvert, then preprocess in MZmine 2.53, handling full-scan and MS/MS modes separately. For MS level 1 use the Centroid detector at noise level 1E3 (1E2 for MS2), build chromatograms with ADAP (min group size 4, group intensity threshold 1E3, min highest intensity 3E3, m/z tolerance 0.001 m/z or 15 ppm), deconvolute with the Baseline cut-off algorithm (min peak height 3E3, peak duration 0–15, baseline level 1E3), group isotopes (0.001 m/z or 15 ppm, RT tolerance 0.2, max charge 3, most intense as representative), align with Join aligner (m/z weight 75, RT weight 25, 0.001 m/z / 0.2 RT tolerances), and gap-fill with Peak finder (intensity tolerance 10). Export common elements (row ID, row m/z, row retention time) to CSV for RegFilter input, and separately export/submit to GNPS-FBMN filtering rows to only those with MS2. The rationale is that the full-scan feature table contains more features than the DDA run, providing the pool from which RegFilter selects ions whose peak area declines linearly (R2 ≥ 0.9, negative slope) across the 1.0, 0.5, 0.25, 0.125, and 0.0625 mg/mL dilution series.

## Related tools

- **MSConvert** (Converts raw MS files to .mzXML format as the first step before MZmine preprocessing)
- **MZmine 2.53** (Performs full-scan and MS/MS preprocessing: mass detection, ADAP chromatogram building, chromatogram deconvolution, isotopic grouping, join alignment, peak finding, and CSV export)
- **RegFilter** (Consumes the MZmine .csv feature table and filters features by linear regression (R2 >= 0.9) over the dilution series via the building_spl_and_assessing_XIC_for_all_ions.ipynb notebook) — https://github.com/computational-chemical-biology/regression_filter

## Examples

```
conda env create -f regfilter_env.yml && conda activate regfilter && pip install git+https://github.com/computational-chemical-biology/regression_filter.git
```

## Evaluation signals

- The exported full-scan .csv contains the required columns: row ID, row m/z, row retention time, and peak area columns mapped onto the five dilution levels (1.0–0.0625 mg/mL)
- The same m/z tolerance (0.001 m/z or 15 ppm) and RT tolerance (0.2 min) are consistently applied across ADAP, isotopic grouping, join aligner, and peak finder steps
- RegFilter applied to the full-scan tables yields 135 filtered features for BRA006, 219 for BRA010, and 150 for BRA177, matching the article's reported counts
- The biological-signal recovery percentages (~9.3%, 9.9%, 9.0% of total features) fall in the same range as reported, and the GNPS-FBMN export contains only rows with MS2

## Limitations

- Many detected ions are not fragmented in data-dependent acquisition (DDA) mode, so MS2 information is missing for most full-scan features and the export for FBMN must filter to rows with MS2
- The RegFilter filtering threshold R2 >= 0.9 is a default value adjustable by the user, and the number of points to filter must be set explicitly via the npts= argument
- No batch-mode processing beyond the Zenodo batch mode file is described; filtering runs interactively through the notebook, pointing it to the .csv path

## Evidence
