---
name: instrument-vendor-software-concentration-comparison
description: Use when raw targeted SRM/MRM data (e.g., Waters Xevo TQ-S .raw files
  from a Biocrates AbsoluteIDQ p180 plate) already have vendor-software quantification
  results and you need to verify that an open-source pipeline reproduces those concentrations.
when_to_use_negative:
- The analysis is non-targeted (feature finding in MS1, alignment, identification)
  — use XCMS, MZmine, or MS-DIAL instead of a targeted extraction/comparison workflow.
- A concentration/feature table already exists and no raw-file reprocessing is needed
  — re-importing raw SRM files into Skyline adds no value.
- The data are GC-MS or ion mobility small-molecule acquisitions, where Skyline support
  was still ongoing at the time of the paper.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Skyline
  role: Imports raw Waters SRM files, defines the 19 amino acid targets from the p180
    transition list, integrates peaks, applies the 1/x-weighted linear calibration
    fit, and exports concentrations for comparison
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: TargetLynx
  role: Instrument vendor software providing the reference concentrations that Skyline-derived
    concentrations are benchmarked against
- name: ProteoWizard/pwiz
  role: Underlying library providing vendor raw-format readers (e.g., Waters) that
    enable Skyline's vendor-independent data import
  repo: https://github.com/ProteoWizard/pwiz-cpp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:9a51f7bb8205136849d465c5d9972834e58bdf2e011a858fc1ca6234d5aeae04
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - ProteoWizard/pwiz
  - Skyline
  - TargetLynx
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# instrument-vendor-software-concentration-comparison

## Summary

Benchmarks an open-source targeted quantification tool (Skyline) against instrument vendor software (TargetLynx) by reprocessing the same raw SRM files with matched curve-fit parameters and comparing calculated concentrations via linear regression. Demonstrates vendor-independent reproducibility of calibrated small-molecule quantification.

## When to use

Use when raw targeted SRM/MRM data (e.g., Waters Xevo TQ-S .raw files from a Biocrates AbsoluteIDQ p180 plate) already have vendor-software quantification results and you need to verify that an open-source pipeline reproduces those concentrations.

## When NOT to use

- The analysis is non-targeted (feature finding in MS1, alignment, identification) — use XCMS, MZmine, or MS-DIAL instead of a targeted extraction/comparison workflow.
- A concentration/feature table already exists and no raw-file reprocessing is needed — re-importing raw SRM files into Skyline adds no value.
- The data are GC-MS or ion mobility small-molecule acquisitions, where Skyline support was still ongoing at the time of the paper.

## Inputs

- 76 raw Waters Xevo TQ-S SRM acquisition files from a single Biocrates AbsoluteIDQ p180 plate
- p180 transition list defining 19 amino acid targets (precursor/product transitions, retention times)
- TargetLynx-reported concentration values for the same samples and analytes

## Outputs

- Skyline-calculated concentrations for 19 amino acids across 76 serum samples
- Linear regression statistics (slope, 95% CI, correlation) comparing Skyline vs TargetLynx concentrations

## How to apply

Import the raw vendor files (here, 76 Xevo TQ-S SRM files from a single Biocrates p180 plate) into Skyline using the Waters vendor reader. Define the target analytes (19 amino acids) from the p180 transition list and integrate their chromatographic peaks. Reproduce the vendor's quantification settings — a linear regression calibration curve with 1/x weighting — in Skyline's calibrated-quantification settings so the comparison is apples-to-apples. Export the calculated concentrations and fit a linear regression of Skyline concentrations against the vendor-reported TargetLynx values across all analytes and samples; agreement is judged by the regression slope and confidence interval (paper benchmark: slope 0.9837, 95% CI 0.9824–0.9850).

## Related tools

- **Skyline** (Imports raw Waters SRM files, defines the 19 amino acid targets from the p180 transition list, integrates peaks, applies the 1/x-weighted linear calibration fit, and exports concentrations for comparison) — https://github.com/ProteoWizard/pwiz-cpp
- **TargetLynx** (Instrument vendor software providing the reference concentrations that Skyline-derived concentrations are benchmarked against)
- **ProteoWizard/pwiz** (Underlying library providing vendor raw-format readers (e.g., Waters) that enable Skyline's vendor-independent data import) — https://github.com/ProteoWizard/pwiz-cpp

## Evaluation signals

- Linear regression of Skyline vs TargetLynx concentrations across all 19 amino acids yields a slope ≈ 0.98 (paper benchmark: 0.9837, 95% CI 0.9824–0.9850).
- Calibration curve fit type and weighting in Skyline (linear regression with 1/x weighting) exactly match the TargetLynx parameters used for the reference values.
- Concentration export contains a complete matrix: all 19 amino acids × all 76 serum samples with no missing quantitation entries.
- Peak integration of each amino acid transition is visually inspected in Skyline's extracted ion chromatogram views before concentrations are accepted.
- Per-analyte correlation is verified at the level reported in the paper's results rather than relying on the pooled slope alone.

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented.
- No single tool will be best for all workflows — the benchmark validates one targeted SRM/small-molecule use case, not universal equivalence with vendor software.
- Vendor raw-format reading depends on vendor SDKs: direct reading of many vendor raw formats is supported on Windows, and licensing must be accepted (i-agree-to-the-vendor-licenses.bat) when building from source.

## Evidence

- [abstract] Skyline performance was benchmarked against a comparable instrument vendor software tool.: [evidence span withheld — anchor status: not_found]
- [discussion] Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support can be implemented.: [evidence span withheld — anchor status: not_found]
