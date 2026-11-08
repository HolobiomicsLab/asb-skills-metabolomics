---
name: compound-density-histogram-visualization
description: Use when an HRMS feature list (e.g., external_feature_list.xlsx from
  PFΔScreen FeatureFinding) must be inspected as MD/C vs. m/C density to see whether
  potential PFAS separate from hydrocarbon matrix components before applying m/C and
  MD/C cutoffs.
when_to_use_negative:
- Input is raw, unfound mzML spectra — m/C and MD/C are per-feature quantities, so
  run PFΔScreen FeatureFinding (pyOpenMS) or supply an external feature list first.
- The goal is detecting homologous series rather than global feature separation —
  use KMD analysis with (CF2)n or CF2O repeating units instead.
- The goal is confirming PFAS identities from MS2 — diagnostic fragments, fragment
  mass differences, and suspect screening (4 mDa tolerance) are the relevant PFΔScreen
  techniques, not this density plot.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: PFΔScreen
  role: Generates the MD/C-m/C interactive HTML plot and the Excel results table during
    PFASPrioritization; the skill reproduces and inspects this MD/C vs. m/C density
    output
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Performs feature detection on mzML (or preprocesses external feature lists)
    to yield the per-feature m/z from which carbon number, m/C, and MD/C are computed
- name: MSConvert
  role: Converts vendor raw data (e.g., Agilent .d) into the mzML files consumed alongside
    the feature list for the plot
- name: Python
  role: Python 3.9 runtime required by the PFΔScreen installation; environment in
    which the histogram is rendered with a log-scaled color axis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:6e40990200954e3f535c9ce3498053aedf6c22b42fc26b1719be0a346a956156
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - MSConvert
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# compound-density-histogram-visualization

## Summary

Renders the PFΔScreen MD/C-m/C compound-density plot: a 2D histogram of mass defect per carbon (MD/C) versus mass per carbon (m/C) for HRMS features, with a log-scaled color axis and CHxF2-x/CFx reference lines. It visualizes why the combined m/C = 30 and MD/C < +0.003 cutoffs separate potential PFAS from hydrocarbon matrix components more effectively than mass defect filtering alone.

## When to use

Use when an HRMS feature list (e.g., external_feature_list.xlsx from PFΔScreen FeatureFinding) must be inspected as MD/C vs. m/C density to see whether potential PFAS separate from hydrocarbon matrix components before applying m/C and MD/C cutoffs.

## When NOT to use

- Input is raw, unfound mzML spectra — m/C and MD/C are per-feature quantities, so run PFΔScreen FeatureFinding (pyOpenMS) or supply an external feature list first.
- The goal is detecting homologous series rather than global feature separation — use KMD analysis with (CF2)n or CF2O repeating units instead.
- The goal is confirming PFAS identities from MS2 — diagnostic fragments, fragment mass differences, and suspect screening (4 mDa tolerance) are the relevant PFΔScreen techniques, not this density plot.

## Inputs

- external_feature_list.xlsx — PFΔScreen custom feature list (per-feature m/z, RT, intensity) from FeatureFinding or vendor software
- Corresponding sample mzML file (ddMS2, centroided) so MS2 spectra remain linkable
- Optional blank feature list (BlankFeatures.xlsx) plus blank mzML for background removal before plotting

## Outputs

- mdc_mc_density_plot.html — interactive 2D MD/C vs. m/C compound-density histogram with log-scaled color axis, CHxF2-x/CFx reference lines, and m/C = 30 / MD/C < +0.003 cutoff guides
- PFΔScreen results table in Excel format
- Companion interactive HTML plots written to the same sample-named folder (m/z vs. RT, KMD vs. m/z, m/C histogram)

## How to apply

Load the feature list produced by PFΔScreen FeatureFinding (external_feature_list.xlsx together with the corresponding mzML file) in a Python 3.9 environment with pyOpenMS, since m/C and MD/C are per-feature quantities. For each feature, compute carbon number, mass per carbon (m/C), and mass defect per carbon (MD/C) within the CH2-scaled Kendrick mass defect framework. Render a 2D compound-density histogram of MD/C vs. m/C with a log-scaled color axis, overlay the CHxF2-x and CFx reference lines for fluorinated compounds, and draw the paper's cutoffs as visual guides: m/C = 30 (PFAS dominated by fluorine usually have a higher m/C) and MD/C < +0.003. Validate against Fig. 1: hydrocarbon matrix components should cluster at m/C < ~25 with positive MD/C, while potentially fluorinated features fall near m/C ~ 40 at MD/C ≈ -0.002. Save the interactive HTML plot (mdc_mc_density_plot.html) alongside the PFΔScreen results table in Excel format, which PFΔScreen writes into a sample-named folder after PFASPrioritization.

## Related tools

- **PFΔScreen** (Generates the MD/C-m/C interactive HTML plot and the Excel results table during PFASPrioritization; the skill reproduces and inspects this MD/C vs. m/C density output) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Performs feature detection on mzML (or preprocesses external feature lists) to yield the per-feature m/z from which carbon number, m/C, and MD/C are computed)
- **MSConvert** (Converts vendor raw data (e.g., Agilent .d) into the mzML files consumed alongside the feature list for the plot)
- **Python** (Python 3.9 runtime required by the PFΔScreen installation; environment in which the histogram is rendered with a log-scaled color axis)

## Examples

```
Run_PFAScreen.bat  # GUI: Browse SampleFeatures.xlsx + mzML → Run ExternalFeatureTable → Run PFASPrioritization → open mdc_mc_density_plot.html
```

## Evaluation signals

- Hydrocarbon matrix components cluster at m/C < ~25 with positive MD/C, reproducing the separation pattern of Fig. 1.
- Potentially fluorinated features plot near m/C ~ 40 at MD/C ≈ -0.002, i.e., below the +0.003 MD/C cutoff.
- The plot has a log-scaled color axis and both CHxF2-x and CFx reference lines overlaid on the MD/C vs. m/C plane.
- Output is an interactive HTML file (mdc_mc_density_plot.html) written to a folder named after the sample, next to the Excel results table.
- Combined cutoffs retain far fewer features than MD-range filtering alone — the study reports 7.4% of features retained with combined m/C + MD/C cutoffs vs. 17% with MD (-0.25 to +0.1 Da) alone.

## Limitations

- Cutoffs (m/C = 30, MD/C < +0.003) are sample- and instrument-dependent; the authors note input parameters can be adjusted 'depending on the tested sample, instrument used, and end-user needs'.
- The plot prioritizes but does not identify — retained features still require MS2 confirmation (diagnostic fragments, KMD homologous series, suspect screening) before PFAS assignment.
- No quantitative benchmark against alternative NTS data-reduction tools is reported, and the documented setup is Python 3.9 with ddMS2 centroided mzML input.

## Evidence

- [results] the m/C and MD/C dimensions were used for data reduction: "the m/C and MD/C dimensions were used for data reduction"
- [results] Therefore, here a cutoff at m/C = 30 was chosen since PFAS that are dominated by fluorine usually have a higher m/C: "Therefore, here a cutoff at m/C = 30 was chosen since PFAS that are dominated by fluorine usually have a higher m/C"
- [results] Attempting to remove further features, an MD/C cutoff of < + 0.003 was set: "Attempting to remove further features, an MD/C cutoff of < + 0.003 was set"
- [discussion] the MD/C-m/C approach is a powerful tool to drastically decrease the number of features and thus reduce false-positive assignments: "the MD/C-m/C approach is a powerful tool to drastically decrease the number of features and thus reduce false-positive assignments"
- [readme] the PFΔScreen results table (Excel format) and several interactive HTML plots are saved in a folder named after the sample that can be easily inspected, including a MD/C-m/C plot: "the PFΔScreen results table (Excel format) and several interactive HTML plots are saved in a folder named after the sample that can be easily inspected, including a MD/C-m/C plot"
- [readme] PFΔScreen uses several techniques for prioritization such as the MD/C-m/C approach, Kendrick mass defect (KMD) analysis and fragment mass differences and diagnostic fragments in the MS2 data.: "PFΔScreen uses several techniques for prioritization such as the MD/C-m/C approach, Kendrick mass defect (KMD) analysis and fragment mass differences and diagnostic fragments in the MS2 data."
