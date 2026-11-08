---
name: metabolomics-feature-count-validation
description: Use when you have an MZmine-exported full-scan feature table (.csv with
  row ID, row m/z, row retention time, peak area) and must verify that a regression-based
  feature filter reproduces published filtered-feature counts and MS1 recovery percentages.
when_to_use_negative:
- The input is already a filtered feature list or only contains DDA-acquired MS/MS
  data — RegFilter in this workflow is applied to full-scan mode feature tables, where
  total features are higher than in DDA mode.
- The goal is annotation or molecular networking (FBMN, SIRIUS, ChemWalker) rather
  than feature selection/count validation — those are downstream steps in the article's
  workflow, not this skill.
- The feature table was produced with different preprocessing parameters (e.g., different
  m/z or retention time tolerances), since the published counts were generated with
  MZmine 2.53 settings of 0.001 m/z or 15 ppm and 0.2 min RT tolerance.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RegFilter
  role: Custom Python package performing the linear-regression filtering (R2 >= 0.9,
    negative slope) on the full-scan feature table via the building_spl_and_assessing_XIC_for_all_ions.ipynb
    notebook, exporting the filtered .csv
  repo: https://github.com/computational-chemical-biology/regression_filter
- name: MZmine 2.53
  role: Preprocessing software producing the full-scan feature table (.csv with row
    ID, m/z, retention time, peak area) that is the input to RegFilter
- name: MSConvert
  role: Converts raw MS files to .mzXML format before MZmine preprocessing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:e671095f52ab5cba214ec9e6512c1039e23597094ee3206542f67860146eb98b
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolomics-feature-count-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce RegFilter linear-regression feature selection on MZmine-preprocessed full-scan MS1 feature tables and validate the filtered-feature counts (R2 >= 0.9, negative slope) against published values for actinomycete strains BRA006, BRA010, and BRA177.

## When to use

Use when you have an MZmine-exported full-scan feature table (.csv with row ID, row m/z, row retention time, peak area) and must verify that a regression-based feature filter reproduces published filtered-feature counts and MS1 recovery percentages.

## When NOT to use

- The input is already a filtered feature list or only contains DDA-acquired MS/MS data — RegFilter in this workflow is applied to full-scan mode feature tables, where total features are higher than in DDA mode.
- The goal is annotation or molecular networking (FBMN, SIRIUS, ChemWalker) rather than feature selection/count validation — those are downstream steps in the article's workflow, not this skill.
- The feature table was produced with different preprocessing parameters (e.g., different m/z or retention time tolerances), since the published counts were generated with MZmine 2.53 settings of 0.001 m/z or 15 ppm and 0.2 min RT tolerance.

## Inputs

- MZmine-preprocessed full-scan MS1 feature table in .csv format (columns: row ID, row m/z, row retention time, peak area) for each strain (BRA006, BRA010, BRA177)
- Raw MS files converted to .mzXML format using MSConvert (upstream input)

## Outputs

- Filtered .csv file containing only features meeting the RegFilter criteria (R2 >= 0.9, negative slope), with their respective retention times
- Report stating the number of total features and the number of filtered (selected) features per strain (expected: 135 BRA006, 219 BRA010, 150 BRA177)
- Biological-signal recovery percentages for comparison (expected: 9.3%, 9.9%, 9.0%)

## How to apply

Preprocess raw LC-MS data in MZmine 2.53 with the published full-scan parameters (m/z tolerance 0.001 m/z or 15 ppm, retention time tolerance 0.2 min) and export a .csv feature table with row ID, row m/z, row retention time, and peak area columns. Install the RegFilter package in a dedicated conda environment (regfilter_env.yml) and open the notebook building_spl_and_assessing_XIC_for_all_ions.ipynb, pointing it to each strain's input .csv. Run the RegFilter filtering algorithm (linear regression criteria R2 >= 0.9 with negative slope), setting the number of points for XIC-based feature assessment via the npts= argument, and let RegFilter export a new .csv containing only features that meet the criteria to the package's notebooks directory. Generate a report of total features and filtered (selected) features per strain, then compare against the reported values: 135 features for BRA006, 219 for BRA010, and 150 for BRA177, corresponding to biological-signal recovery percentages of 9.3%, 9.9%, and 9.0%. Validation is accepted when the exported .csv contains the filtered features with their retention times and the report matches the published counts.

## Related tools

- **RegFilter** (Custom Python package performing the linear-regression filtering (R2 >= 0.9, negative slope) on the full-scan feature table via the building_spl_and_assessing_XIC_for_all_ions.ipynb notebook, exporting the filtered .csv) — https://github.com/computational-chemical-biology/regression_filter
- **MZmine 2.53** (Preprocessing software producing the full-scan feature table (.csv with row ID, m/z, retention time, peak area) that is the input to RegFilter)
- **MSConvert** (Converts raw MS files to .mzXML format before MZmine preprocessing)

## Examples

```
conda activate regfilter; pip install git+https://github.com/computational-chemical-biology/regression_filter.git; # then in building_spl_and_assessing_XIC_for_all_ions.ipynb: run RegFilter with npts= on the strain .csv
```

## Evaluation signals

- Exported .csv contains exactly the features that meet the RegFilter criteria (R2 >= 0.9, negative slope) with their respective retention times.
- Reported filtered-feature counts match the article values: 135 for BRA006, 219 for BRA010, and 150 for BRA177.
- MS1 recovery percentages match the reported 9.3%, 9.9%, and 9.0% for BRA006, BRA010, and BRA177 respectively.
- Input feature table schema check: columns are row ID, row m/z, row retention time, and peak area, produced with MZmine 2.53 (m/z tolerance 0.001 m/z or 15 ppm, RT tolerance 0.2 min).
- The total number of features detected in full scan mode is higher than the total number detected in DDA mode, as reported in the article.

## Limitations

- Many detected ions are not fragmented in data-dependent acquisition (DDA) mode, and its stochastic nature means ions with similar intensities may or may not be fragmented; the skill validates feature selection, not fragmentation coverage.
- Most precursor ions selected in the SPL were present in the MS1 of the DDA method but not all were fragmented; counts are per-strain and depend on the exact MZmine preprocessing parameters.
- No code repository DOI is provided in the article discussion despite RegFilter being the newly introduced tool; reproducibility relies on the GitHub repository and Zenodo batch mode file.

## Evidence
