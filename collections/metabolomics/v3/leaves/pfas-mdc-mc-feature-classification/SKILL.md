---
name: pfas-mdc-mc-feature-classification
description: Use when a feature list from non-target LC- or GC-HRMS data (mzML, DDA)
  must be reduced to plausible PFAS candidates, separating them from hydrocarbon matrix
  components on the MD/C vs m/C plane.
when_to_use_negative:
- Data are not HRMS feature-level measurements (e.g., already confirmed PFAS identities
  or raw spectra without feature finding) — use suspect screening or MS2 fragment
  matching instead
- The goal is to assign PFAS classes or confirm structures; MD/C-m/C is a prioritization/data-reduction
  step, not an identification method
- Sample matrix is not dominated by hydrocarbon components whose m/C/MD/C separation
  (m/C < ~25, positive MD/C) underlies the cutoff rationale
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PFΔScreen
  role: Executes the MD/C-m/C calculation, renders the 2D density histogram, applies
    the m/C=30 and MD/C<+0.003 cutoffs, and exports results table and HTML plots
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Provides feature detection from mzML raw data (or accepts custom feature lists)
    feeding the MD/C-m/C classification
- name: MSConvert
  role: Converts vendor raw files (e.g., Agilent .d) into mzML required as PFΔScreen
    input
- name: Python
  role: Runtime environment (Python 3.9 required) in which PFΔScreen and the MD/C-m/C
    computations run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:52e023b8e544fbb553279a9214640f8934ad936b7b6d4d822005eaebe01b2d4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - MSConvert
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# pfas-mdc-mc-feature-classification

## Summary

Classifies HRMS features as potentially fluorinated (PFAS) vs hydrocarbon matrix components using the CH2-scaled Kendrick mass defect framework: per-feature carbon number, mass per carbon (m/C), and mass defect per carbon (MD/C), visualized as a 2D density histogram and applied as a data-reduction filter.

## When to use

Use when a feature list from non-target LC- or GC-HRMS data (mzML, DDA) must be reduced to plausible PFAS candidates, separating them from hydrocarbon matrix components on the MD/C vs m/C plane.

## When NOT to use

- Data are not HRMS feature-level measurements (e.g., already confirmed PFAS identities or raw spectra without feature finding) — use suspect screening or MS2 fragment matching instead
- The goal is to assign PFAS classes or confirm structures; MD/C-m/C is a prioritization/data-reduction step, not an identification method
- Sample matrix is not dominated by hydrocarbon components whose m/C/MD/C separation (m/C < ~25, positive MD/C) underlies the cutoff rationale

## Inputs

- Feature list from feature finding (external_feature_list.xlsx) with corresponding mzML file
- mzML raw MS data (DDA, centroided spectra), vendor-independently converted via MSConvert
- Blank mzML (optional, e.g., extraction blank) for prior blank correction

## Outputs

- 2D compound-density histogram of MD/C vs m/C with log-scaled color axis and CHxF2-x / CFx reference lines (interactive HTML plot, mdc_mc_density_plot.html)
- PFΔScreen results table (Excel) with per-feature carbon number, m/C, MD/C and retained/reduced status
- Reduced feature subset retained after m/C = 30 and MD/C < +0.003 cutoffs

## How to apply

Load the feature list (external_feature_list.xlsx with its corresponding mzML file) via PFΔScreen in a Python 3.9 environment with pyOpenMS, or generate features directly with pyOpenMS feature finding. For each feature, compute carbon number, m/C, and MD/C within the CH2-scaled Kendrick mass defect framework. Render a 2D compound-density histogram of MD/C vs m/C with a log-scaled color axis, overlaying the CHxF2-x and CFx reference lines for fluorinated compounds. Apply the paper's data-reduction cutoffs as visual guides: m/C cutoff at 30 (PFAS dominated by fluorine usually have a higher m/C) and MD/C cutoff of < +0.003. Validate the separation pattern of Fig. 1: hydrocarbon matrix components cluster at m/C < ~25 with positive MD/C, while potentially fluorinated features fall near m/C ~ 40 at MD/C ≈ -0.002. Save the interactive HTML plot (mdc_mc_density_plot.html) alongside the PFΔScreen results table in Excel format.

## Related tools

- **PFΔScreen** (Executes the MD/C-m/C calculation, renders the 2D density histogram, applies the m/C=30 and MD/C<+0.003 cutoffs, and exports results table and HTML plots) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Provides feature detection from mzML raw data (or accepts custom feature lists) feeding the MD/C-m/C classification)
- **MSConvert** (Converts vendor raw files (e.g., Agilent .d) into mzML required as PFΔScreen input)
- **Python** (Runtime environment (Python 3.9 required) in which PFΔScreen and the MD/C-m/C computations run)

## Examples

```
double click Run_PFAScreen.bat; load external_feature_list.xlsx + sample mzML via 'Browse SampleFeatures.xlsx', then press 'Run ExternalFeatureTable' followed by 'Run PFASPrioritization'
```

## Evaluation signals

- Features retained after the m/C = 30 and MD/C < +0.003 cutoffs reproduce the Fig. 1 separation: hydrocarbons at m/C < ~25 with positive MD/C vs fluorinated features near m/C ~ 40 and MD/C ≈ -0.002
- Data-reduction ratio matches the paper's finding: MD range (-0.25 to +0.1 Da) alone leaves 17% of features, while combined m/C and MD/C cutoffs leave only 7.4%
- The mdc_mc_density_plot.html renders with log-scaled color axis and the CHxF2-x and CFx reference lines overlaid
- The Excel results table contains per-feature carbon number, m/C and MD/C values consistent with the CH2-scaled Kendrick mass defect framework
- Upstream filters were correctly applied first: FWHM 1 s–1 min with ≥2 isotopic traces, and blank correction (2 mDa / <0.1 min RT / ≥5-fold sample abundance)

## Limitations

- The MD/C-m/C approach is a prioritization/data-reduction tool to reduce false-positive assignments; it does not confirm PFAS identity
- Cutoffs (m/C = 30, MD/C < +0.003) were chosen for PFAS-contaminated soil extracts; input parameters must be adjusted depending on the tested sample, instrument used, and end-user needs
- Requires DDA mzML data with centroided spectra for MS2-based verification; data evaluation fails when the corresponding mzML files are missing

## Evidence

- [results] the m/C and MD/C dimensions were used for data reduction: "the m/C and MD/C dimensions were used for data reduction"
- [results] m/C cutoff at 30: "Therefore, here a cutoff at m/C = 30 was chosen since PFAS that are dominated by fluorine usually have a higher m/C"
- [results] MD/C cutoff of < +0.003: "Attempting to remove further features, an MD/C cutoff of < + 0.003 was set"
- [discussion] MD/C-m/C as powerful reduction tool: "the MD/C-m/C approach is a powerful tool to drastically decrease the number of features and thus reduce false-positive assignments"
- [readme] MD/C-m/C plot among PFΔScreen outputs: "several interactive HTML plots are saved in a folder named after the sample that can be easily inspected, including a MD/C-m/C plot"
- [readme] custom feature lists require corresponding mzML: "Note that data evaluation only works when the corresponding mzML files are also given; otherwise MS2 data would be missing."
