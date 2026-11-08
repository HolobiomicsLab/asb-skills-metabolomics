---
name: hrms-feature-table-processing
description: Use when a non-target LC/GC-HRMS feature list (e.g. external_feature_list.xlsx
  from FeatureFinding with its mzML file) must be prioritized for PFAS by reducing
  matrix features using the MD/C-m/C data-reduction dimensions.
when_to_use_negative:
- Input is raw MS data rather than a feature list — use PFΔScreen's FeatureFinding
  tab first (or vendor feature finding) before external-feature-table MD/C-m/C processing
- Analysis goal is homologous-series detection rather than per-feature MD/C-m/C reduction
  — KMD analysis with (CF2)n and CF2O repeating units (±2 mDa, ≥3 homologues) is the
  appropriate technique
- Sample matrix is dominated by features with m/C < 30 and negative MD/C that are
  not PFAS — the m/C = 30 / MD/C < +0.003 cutoffs may wrongly discard or retain features;
  parameters must be adjusted per sample
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: PFΔScreen
  role: Runs the MD/C-m/C prioritization on the feature list, produces the MD/C-m/C
    plot and Excel results table
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Feature detection from mzML raw data feeding the MD/C-m/C reduction (or the
    basis for loading custom feature lists in the Python 3.9 environment)
  repo: https://github.com/JonZwe/PFAScreen
- name: MSConvert
  role: Converts vendor raw data (e.g. Agilent .d) into the mzML format consumed before
    MD/C-m/C processing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:163884bc561fb2e31b80c2cbf0d28f16dc5135140ba0e8d73f87d762a4b1f6bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - MSConvert
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hrms-feature-table-processing

## Summary

Processes HRMS feature tables through the MD/C-m/C framework of PFΔScreen: per-feature carbon number, mass-per-carbon (m/C) and mass-defect-per-carbon (MD/C) are computed under the CH2-scaled Kendrick framework and used with cutoffs (m/C = 30, MD/C < +0.003) to reduce non-target features to potential PFAS far more efficiently than a mass-defect window alone.

## When to use

Use when a non-target LC/GC-HRMS feature list (e.g. external_feature_list.xlsx from FeatureFinding with its mzML file) must be prioritized for PFAS by reducing matrix features using the MD/C-m/C data-reduction dimensions.

## When NOT to use

- Input is raw MS data rather than a feature list — use PFΔScreen's FeatureFinding tab first (or vendor feature finding) before external-feature-table MD/C-m/C processing
- Analysis goal is homologous-series detection rather than per-feature MD/C-m/C reduction — KMD analysis with (CF2)n and CF2O repeating units (±2 mDa, ≥3 homologues) is the appropriate technique
- Sample matrix is dominated by features with m/C < 30 and negative MD/C that are not PFAS — the m/C = 30 / MD/C < +0.003 cutoffs may wrongly discard or retain features; parameters must be adjusted per sample

## Inputs

- external_feature_list.xlsx (custom feature list from FeatureFinding or vendor software)
- Corresponding mzML sample file (DDA, centroided; required so MS2 data are available)
- Optional external blank feature list (BlankFeatures.xlsx) with its mzML file

## Outputs

- PFΔScreen results table in Excel format with retained (potential PFAS) features
- mdc_mc_density_plot.html — interactive 2D MD/C vs. m/C compound-density histogram with log-scaled color axis and CHxF2-x/CFx reference lines
- Features passing m/C = 30 and MD/C < +0.003 cutoffs (reported ~7.4% of original features)

## How to apply

Load the feature list (external_feature_list.xlsx with the corresponding mzML file) via PFΔScreen's ExternalFeatureTable path in a Python 3.9 environment with pyOpenMS, or run FeatureFinding directly (10 ppm mass error, MS1 intensity threshold 2000 counts, OpenMS metabolites (5% RMS) isotope model). For each feature compute carbon number, m/C and MD/C in the CH2-scaled Kendrick mass defect framework, then render a 2D compound-density histogram of MD/C vs. m/C with a log-scaled color axis, overlaying CHxF2-x and CFx reference lines for fluorinated compounds. Apply the paper's reduction cutoffs as visual guides: m/C = 30 (fluorine-dominated PFAS usually have higher m/C) and MD/C < +0.003; for reference, an MD window of −0.25 to +0.1 Da alone left 17% of features, whereas the combined m/C and MD/C cutoffs left only 7.4%. Decision point: features near m/C ~ 40 at MD/C ≈ −0.002 are candidate PFAS; hydrocarbon matrix components cluster at m/C < ~25 with positive MD/C, matching Fig. 1 of the paper. Save the interactive HTML plot (mdc_mc_density_plot.html) alongside the PFΔScreen results table in Excel format.

## Related tools

- **PFΔScreen** (Runs the MD/C-m/C prioritization on the feature list, produces the MD/C-m/C plot and Excel results table) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Feature detection from mzML raw data feeding the MD/C-m/C reduction (or the basis for loading custom feature lists in the Python 3.9 environment)) — https://github.com/JonZwe/PFAScreen
- **MSConvert** (Converts vendor raw data (e.g. Agilent .d) into the mzML format consumed before MD/C-m/C processing)

## Examples

```
pip install -r requirements.txt && Run_PFAScreen.bat  (then load external_feature_list.xlsx with its mzML and press Run ExternalFeatureTable / Run PFASPrioritization)
```

## Evaluation signals

- Hydrocarbon matrix components cluster at m/C < ~25 with positive MD/C, while potentially fluorinated features fall near m/C ~ 40 at MD/C ≈ −0.002 (matches Fig. 1)
- Retained-feature fraction after combined cutoffs is ~7.4%, versus ~17% for an MD-only window of −0.25 to +0.1 Da — demonstrating superior data reduction
- Features surviving the m/C = 30 and MD/C < +0.003 filters dominate the fluorine-dominated region above the m/C cutoff
- Output artifacts exist and are consistent: mdc_mc_density_plot.html renders a log-scaled color axis with CHxF2-x and CFx reference lines, and the results table is written in Excel format in the sample-named folder

## Limitations

- Cutoffs are sample- and instrument-dependent; the article notes input parameters can be adjusted depending on the tested sample, instrument used, and end-user needs
- Processing via the external feature table only works when the corresponding mzML files are also given, otherwise MS2 data would be missing
- Python 3.9 is required for automated PFΔScreen installation; features must be reduced by blank correction first (e.g. 4209 → 3750 features for soil M1 in ESI−) before MD/C-m/C reduction
- The MD/C-m/C approach drastically decreases features but does not itself confirm PFAS identity — suspect screening (4 mDa tolerance) and fragment evidence are complementary steps

## Evidence

- [results] the m/C and MD/C dimensions were used for data reduction: "the m/C and MD/C dimensions were used for data reduction"
- [results] m/C cutoff at 30: "Therefore, here a cutoff at m/C = 30 was chosen since PFAS that are dominated by fluorine usually have a higher m/C"
- [results] MD/C cutoff of < +0.003: "Attempting to remove further features, an MD/C cutoff of < + 0.003 was set"
- [discussion] MD/C-m/C is a powerful feature-reduction tool: "the MD/C-m/C approach is a powerful tool to drastically decrease the number of features and thus reduce false-positive assignments"
- [readme] external feature lists with mzML files: "custom feature lists (see external_feature_list.xlsx) together with the respective mzML files can instead be included in PFΔScreen"
