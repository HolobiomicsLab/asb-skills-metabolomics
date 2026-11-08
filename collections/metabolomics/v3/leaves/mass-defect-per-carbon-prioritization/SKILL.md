---
name: mass-defect-per-carbon-prioritization
description: Use when a post-FeatureFinding feature list (e.g., external_feature_list.xlsx
  from mzML data) must be reduced from thousands of matrix components to a small set
  of potential PFAS candidates in non-target screening.
when_to_use_negative:
- Input is a single suspect list of known PFAS (e.g., PFASMASTER/PFASOECDNA hits)
  — suspect screening, not MD/C-m/C reduction, is the right step
- Data are not LC- or GC-HRMS features (no m/z, RT, isotopic traces) — the CH2-scaled
  Kendrick framework does not apply
- Raw spectra have not been converted to mzML or passed feature finding/blank correction
  — the skill requires a per-feature carbon-number calculation, not raw profile data
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PFΔScreen
  role: Executes the MD/C-m/C prioritization calculations and saves the MD/C-m/C plot
    and results table
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Feature detection from mzML raw data producing the feature list that feeds
    the MD/C-m/C reduction
- name: MSConvert
  role: Converts vendor raw MS data (e.g., Agilent .d) into mzML required for pyOpenMS
    feature finding
- name: Python
  role: Runtime environment (Python 3.9 required) for PFΔScreen and pyOpenMS-based
    per-feature m/C and MD/C computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:795e3c59c516047321d8ce5b0eb5a6416de4c66abc705b987d1f95e5958b2fea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - MSConvert
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# mass-defect-per-carbon-prioritization

## Summary

Data-reduction skill that prioritizes potential PFAS features in HRMS feature lists using the CH2-scaled Kendrick-based MD/C-m/C approach (mass defect and mass per carbon), applied via PFΔScreen on mzML-derived features from LC/GC-HRMS data.

## When to use

Use when a post-FeatureFinding feature list (e.g., external_feature_list.xlsx from mzML data) must be reduced from thousands of matrix components to a small set of potential PFAS candidates in non-target screening.

## When NOT to use

- Input is a single suspect list of known PFAS (e.g., PFASMASTER/PFASOECDNA hits) — suspect screening, not MD/C-m/C reduction, is the right step
- Data are not LC- or GC-HRMS features (no m/z, RT, isotopic traces) — the CH2-scaled Kendrick framework does not apply
- Raw spectra have not been converted to mzML or passed feature finding/blank correction — the skill requires a per-feature carbon-number calculation, not raw profile data

## Inputs

- external_feature_list.xlsx with the corresponding mzML file (vendor-independent DDA, centroided spectra)
- pyOpenMS FeatureFinding output or custom vendor feature lists
- PFASPrioritization input parameters (mass error 10 ppm, MS1 intensity threshold 2000 counts, OpenMS isotope model)

## Outputs

- mdc_mc_density_plot.html — interactive 2D MD/C vs. m/C compound-density plot with CHxF2-x and CFx reference lines
- PFΔScreen results table (Excel format) with features retained after m/C and MD/C cutoffs
- Reduced candidate feature set for downstream KMD, fragment, and suspect-screening steps

## How to apply

Load the feature list (external_feature_list.xlsx with the corresponding mzML file) into PFΔScreen in a Python 3.9 environment with pyOpenMS, and for each feature compute carbon number, mass per carbon (m/C) and mass defect per carbon (MD/C) in the CH2-scaled Kendrick framework. Render a 2D MD/C vs. m/C density histogram (log-scaled color axis) overlaid with CHxF2-x and CFx reference lines. Apply the paper's cutoffs as data-reduction filters: m/C = 30 (fluorine-dominated PFAS have higher m/C) and MD/C < +0.003. Expect hydrocarbon matrix components to cluster at m/C < ~25 with positive MD/C, while potentially fluorinated features fall near m/C ~ 40 at MD/C ≈ -0.002, reproducing Fig. 1. Save the interactive MD/C-m/C HTML plot (mdc_mc_density_plot.html) together with the PFΔScreen results table in Excel format.

## Related tools

- **PFΔScreen** (Executes the MD/C-m/C prioritization calculations and saves the MD/C-m/C plot and results table) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Feature detection from mzML raw data producing the feature list that feeds the MD/C-m/C reduction)
- **MSConvert** (Converts vendor raw MS data (e.g., Agilent .d) into mzML required for pyOpenMS feature finding)
- **Python** (Runtime environment (Python 3.9 required) for PFΔScreen and pyOpenMS-based per-feature m/C and MD/C computation)

## Evaluation signals

- Applying the MD range (−0.25 to +0.1 Da) alone should leave ~17% of features, while combining m/C and MD/C cutoffs should leave only ~7.4% — MD/C-m/C must outperform MD alone
- Retained features after m/C = 30 and MD/C < +0.003 cutoffs must show hydrocarbons clustered at m/C < ~25 with positive MD/C versus fluorinated features near m/C ~ 40 at MD/C ≈ -0.002 (Fig. 1 pattern)
- The CHxF2-x and CFx reference lines must be overlaid on the MD/C vs. m/C histogram with a log-scaled color axis
- Output folder must contain mdc_mc_density_plot.html and the results table in Excel format alongside other interactive HTML plots
- Retention is biased toward higher m/C — check that no feature with m/C well below 30 and strongly positive MD/C survives the filter

## Limitations

- The m/C = 30 and MD/C < +0.003 cutoffs were chosen for PFAS-dominated by fluorine and may discard short-chain or oxygen-rich PFAS with lower m/C
- MD/C-m/C reduction drastically decreases features but is a prioritization, not identification — retained features still require KMD, fragment matching, and suspect screening
- No quantitative benchmark against alternative NTS data-reduction tools was reported; the number of unknown PFAS in complex samples remains unknown

## Evidence

- [results] Therefore, here a cutoff at m/C = 30 was chosen since PFAS that are dominated by fluorine usually have a higher m/C: "Therefore, here a cutoff at m/C = 30 was chosen since PFAS that are dominated by fluorine usually have a higher m/C"
- [results] Attempting to remove further features, an MD/C cutoff of < +0.003 was set: "Attempting to remove further features, an MD/C cutoff of < + 0.003 was set"
- [results] the m/C and MD/C dimensions were used for data reduction: "the m/C and MD/C dimensions were used for data reduction"
- [readme] PFΔScreen uses several techniques for prioritization such as the MD/C-m/C approach, Kendrick mass defect (KMD) analysis and fragment mass differences and diagnostic fragments in the MS2 data.: "PFΔScreen uses several techniques for prioritization such as the MD/C-m/C approach, Kendrick mass defect (KMD) analysis and fragment mass differences and diagnostic fragments in the MS2 data."
- [readme] several interactive HTML plots are saved in a folder named after the sample that can be easily inspected, including a MD/C-m/C plot: "the PFΔScreen results table (Excel format) and several interactive HTML plots are saved in a folder named after the sample that can be easily inspected, including a MD/C-m/C plot"
