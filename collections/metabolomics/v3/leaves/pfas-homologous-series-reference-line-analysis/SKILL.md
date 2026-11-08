---
name: pfas-homologous-series-reference-line-analysis
description: Use when you have HRMS feature lists (mzML-derived or external_feature_list.xlsx)
  and need to visually verify separation of fluorinated features from matrix components
  in the MD/C vs. m/C space before applying data-reduction cutoffs.
when_to_use_negative:
- Input is raw vendor data (.d, Agilent) not yet converted to mzML with MSConvert
  — feature detection must precede MD/C-m/C plotting
- The goal is homologous-series detection across (CF2)n or CF2O repeating units —
  use the KMD analysis plot (KMD vs. m/z with linked m/z vs. RT) instead
- Features have not been blank-corrected (features within 2 mDa and RT <0.1 min of
  the extraction blank, unless ≥5-fold more abundant, must first be removed), which
  would distort the density histogram
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PFΔScreen
  role: Computes per-feature carbon number, m/C and MD/C and renders the interactive
    MD/C-m/C density plot with reference lines and cutoff guides
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Performs feature detection from vendor-independent mzML raw data prior to
    MD/C-m/C analysis (or ingests custom feature lists)
- name: Python
  role: Runtime environment (Python 3.9 required) in which PFΔScreen and the plotting
    routines execute
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:e4bab678371953e825d7465b9f9f4d84b71cfe7119ec620b125e79369f7e6535
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pfas-homologous-series-reference-line-analysis

## Summary

Renders and interprets a 2D compound-density histogram of mass defect per carbon (MD/C) vs. mass per carbon (m/C), overlaying CHxF2−x and CFx reference lines, to separate potential PFAS features from hydrocarbon matrix components in HRMS non-target screening via PFΔScreen.

## When to use

Use when you have HRMS feature lists (mzML-derived or external_feature_list.xlsx) and need to visually verify separation of fluorinated features from matrix components in the MD/C vs. m/C space before applying data-reduction cutoffs.

## When NOT to use

- Input is raw vendor data (.d, Agilent) not yet converted to mzML with MSConvert — feature detection must precede MD/C-m/C plotting
- The goal is homologous-series detection across (CF2)n or CF2O repeating units — use the KMD analysis plot (KMD vs. m/z with linked m/z vs. RT) instead
- Features have not been blank-corrected (features within 2 mDa and RT <0.1 min of the extraction blank, unless ≥5-fold more abundant, must first be removed), which would distort the density histogram

## Inputs

- external_feature_list.xlsx with the corresponding mzML file (or PFΔScreen FeatureFinding output)
- mzML raw data file (data-dependent acquisition, centroided spectra) for MS2 context

## Outputs

- Interactive 2D MD/C vs. m/C compound-density histogram (mdc_mc_density_plot.html) with log-scaled color axis, CHxF2−x and CFx reference lines, and m/C=30 / MD/C=+0.003 cutoff overlays
- PFΔScreen results table (Excel format) with per-feature carbon number, m/C, and MD/C values

## How to apply

Load the feature list in PFΔScreen (Python 3.9 with pyOpenMS) and, for each feature, compute the carbon number, mass per carbon (m/C), and mass defect per carbon (MD/C) within the CH2-scaled Kendrick mass defect framework. Render a 2D compound-density histogram of MD/C vs. m/C with a log-scaled color axis, overlaying the CHxF2−x and CFx reference lines for fluorinated compounds. Add the paper's cutoff guides: m/C = 30 (fluorine-dominated PFAS usually have a higher m/C) and MD/C < +0.003. Correct application reproduces the separation pattern of Fig. 1: hydrocarbon matrix components cluster at m/C < ~25 with positive MD/C, while potentially fluorinated features fall near m/C ~ 40 at MD/C ≈ −0.002. The plot (mdc_mc_density_plot.html) is saved alongside the PFΔScreen results table in Excel format.

## Related tools

- **PFΔScreen** (Computes per-feature carbon number, m/C and MD/C and renders the interactive MD/C-m/C density plot with reference lines and cutoff guides) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Performs feature detection from vendor-independent mzML raw data prior to MD/C-m/C analysis (or ingests custom feature lists))
- **Python** (Runtime environment (Python 3.9 required) in which PFΔScreen and the plotting routines execute)

## Examples

```
Run_PFAScreen.bat  # load external_feature_list.xlsx + mzML via 'Browse SampleFeatures.xlsx', then 'Run ExternalFeatureTable' and 'Run PFASPrioritization' to produce mdc_mc_density_plot.html
```

## Evaluation signals

- After applying the m/C = 30 and MD/C < +0.003 cutoffs, retained features show the reported separation: hydrocarbons at m/C < ~25 with positive MD/C vs. fluorinated features near m/C ~ 40 and MD/C = −0.002 (matching Fig. 1)
- Data reduction is superior to MD alone: the MD range (−0.25 to +0.1 Da) alone left 17% of features, while combining m/C and MD/C cutoffs left only 7.4%
- The interactive HTML plot has a log-scaled color axis and displays the CHxF2−x and CFx reference lines for fluorinated compounds
- The plot file (mdc_mc_density_plot.html) and Excel results table are saved in the sample-named output folder
- Per-feature MD/C and m/C values are consistent with the CH2-scaled Kendrick mass defect framework (mass defect and mass per carbon)

## Limitations

- Cutoffs (m/C = 30, MD/C < +0.003) are sample- and instrument-dependent; PFΔScreen input parameters can be adjusted depending on the tested sample, instrument, and end-user needs
- The MD/C-m/C approach reduces features drastically but does not confirm PFAS identity; further steps (KMD analysis, fragment matching, suspect screening with 4 mDa tolerance) are needed
- A paper-wide gap: the number of unknown PFAS in complex environmental and technical samples is still unknown, so the plot may miss uncharacterized fluorinated chemistry

## Evidence

- [results] the m/C and MD/C dimensions were used for data reduction: "the m/C and MD/C dimensions were used for data reduction"
- [results] m/C cutoff at 30: "Therefore, here a cutoff at m/C = 30 was chosen since PFAS that are dominated by fluorine usually have a higher m/C"
- [results] MD/C cutoff of < +0.003: "Attempting to remove further features, an MD/C cutoff of < + 0.003 was set"
- [discussion] MD/C-m/C reduction power: "the MD/C-m/C approach is a powerful tool to drastically decrease the number of features and thus reduce false-positive assignments"
- [readme] HTML plots saved including MD/C-m/C plot: "including a MD/C-m/C plot, a m/z vs. RT plot (with and without MS2 raw data), a KMD vs. m/z with linked m/z vs. RT plot (to verify systematic RT-shifts), and a m/C histogram"
- [results] MD range alone vs combined cutoffs: [evidence span withheld — anchor status: not_found]
