---
name: calibration-curve-regression-1x-weighting
description: Use when targeted SRM/PRM data (e.g., Xevo TQ-S raw files from a Biocrates
  p180 plate) must be converted from integrated peak areas to concentrations, and
  the results must be benchmarked against or reproduce a vendor quantification package
  such as TargetLynx.
when_to_use_negative:
- The workflow uses uncalibrated relative quantification or stable-isotope dilution
  without external calibration standards, where a concentration curve is not fit at
  all
- The original quantification used a non-linear or differently weighted fit — replicating
  those results requires matching that specific curve model instead of 1/x-weighted
  linear regression
- The analysis is non-targeted feature finding (e.g., XCMS/MZmine-style untargeted
  metabolomics), which does not use calibration-curve quantification
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Skyline
  role: Applies the 1/x-weighted linear regression calibration curve to SRM peak areas
    and exports calculated amino acid concentrations, benchmarked against TargetLynx
  repo: https://github.com/ProteoWizard/pwiz-cpp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:5b1761b7f6db6f9442f19f2dab2f0a32f9e542c2349be4fce3789b2b73a26921
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - Skyline
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# calibration-curve-regression-1x-weighting

## Summary

Applies a linear regression calibration curve with 1/x weighting in Skyline to convert SRM peak areas into analyte concentrations, matching the curve-fit parameters of Waters TargetLynx for vendor-independent reanalysis of Biocrates AbsoluteIDQ p180 kit data.

## When to use

Use when targeted SRM/PRM data (e.g., Xevo TQ-S raw files from a Biocrates p180 plate) must be converted from integrated peak areas to concentrations, and the results must be benchmarked against or reproduce a vendor quantification package such as TargetLynx.

## When NOT to use

- The workflow uses uncalibrated relative quantification or stable-isotope dilution without external calibration standards, where a concentration curve is not fit at all
- The original quantification used a non-linear or differently weighted fit — replicating those results requires matching that specific curve model instead of 1/x-weighted linear regression
- The analysis is non-targeted feature finding (e.g., XCMS/MZmine-style untargeted metabolomics), which does not use calibration-curve quantification

## Inputs

- Raw Xevo TQ-S SRM data files (76 samples from one Biocrates AbsoluteIDQ p180 plate)
- Transition list defining the 19 amino acid targets from the p180 kit
- Calibration standard concentrations from the p180 kit plate layout
- Vendor-reported concentrations from TargetLynx (for benchmarking)

## Outputs

- Fitted linear calibration curve with 1/x weighting per analyte in Skyline
- Calculated analyte concentrations exported from Skyline (e.g., via the Report/Export grid)
- Slope, intercept, and regression statistics comparing Skyline vs. TargetLynx concentrations

## How to apply

In Skyline, after importing the raw files with the Waters vendor reader and defining the 19 amino acid targets from the p180 transition list, open the calibration curve settings and select linear regression with 1/x weighting so the fit matches the TargetLynx quantification parameters. Assign calibration standards and heavy-isotope internal standards from the p180 kit layout, then integrate the chromatographic peaks and let Skyline calculate concentrations from the fitted curve. Export the calculated concentrations for all analytes and compare them against the vendor-reported values (e.g., via linear regression of Skyline vs. TargetLynx concentrations). The rationale is that weighting residuals by 1/x gives low-concentration calibrants proportionally more influence, which is how TargetLynx fits the p180 kit curves; matching this weighting is what makes the cross-software comparison meaningful. Validation: the paper reports slope 0.9837 (95% CI 0.9824-0.9850) for the Skyline vs. TargetLynx amino acid concentration correlation across the 76 ADNI serum samples.

## Related tools

- **Skyline** (Applies the 1/x-weighted linear regression calibration curve to SRM peak areas and exports calculated amino acid concentrations, benchmarked against TargetLynx) — https://github.com/ProteoWizard/pwiz-cpp

## Evaluation signals

- Regression of Skyline-calculated concentrations against TargetLynx-reported values for all 19 amino acids should reproduce the reported slope of 0.9837 (95% CI 0.9824-0.9850)
- The calibration curve settings in Skyline must be verifiably set to linear regression with 1/x weighting, matching the TargetLynx parameters
- Exported concentration tables must cover all 19 amino acids across all 76 samples from the single p180 plate
- Calibration points with high concentration residuals should not dominate the fit — low-level calibrants should retain influence under 1/x weighting

## Limitations

- Skyline may lag behind commercial software for vendor-specific workflows (e.g., TargetLynx) until support is implemented, so curve-fit defaults must be manually matched
- No single tool is best for all workflows; Skyline's agreement with vendor software was demonstrated for this p180 amino acid assay and may differ for other kits or analyte classes
- Accurate reproduction requires the same transition list, integration, and curve-fit parameters as the vendor run — any mismatch in weighting or fit model degrades the correlation

## Evidence

- [results] Apply the calibration curve fit matching TargetLynx parameters: [evidence span withheld — anchor status: not_found]
- [abstract] Benchmarked against vendor software: "demonstrate Skyline performance benchmarked against a comparable instrument vendor software tool"
- [intro] External calibration quantification: "often accompanied by approaches utilizing stable-isotope dilution or external calibration to quantitatively calculate concentrations of compounds of interest"
