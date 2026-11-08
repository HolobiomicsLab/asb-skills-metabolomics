---
name: amino-acid-biocrates-p180-kit-analysis
description: Use when raw SRM/MRM files from a Biocrates AbsoluteIDQ p180 plate (e.g.,
  Waters Xevo TQ-S serum runs) must be reanalyzed in Skyline to reproduce or audit
  vendor-reported amino acid concentrations.
when_to_use_negative:
- The data require nontargeted feature-finding, alignment, and identification workflows
  (XCMS, MZmine, MS-DIAL) rather than extraction of known targets
- The instrument/workflow is a latest-generation or vendor-specific acquisition Skyline
  does not yet support, since Skyline may lag behind commercial software until support
  is implemented
- The sample data are GC-MS or ion mobility small molecule runs, whose Skyline support
  is still ongoing
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Skyline
  role: Import Waters vendor SRM raw files, define the 19 amino acid targets from
    the p180 transition list, apply the TargetLynx-matching linear calibration with
    1/x weighting, and export calculated concentrations
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: TargetLynx
  role: Instrument vendor software whose reported concentrations serve as the benchmark
    values for the Skyline-vs-TargetLynx correlation analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:45b36377d0a56d1dbdfb8b9b24af2f7242f339b19d5eb9330e5d9ca0b86b3d32
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - Skyline
  - TargetLynx
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# amino-acid-biocrates-p180-kit-analysis

## Summary

Reprocess raw targeted SRM data from the Biocrates AbsoluteIDQ p180 kit (19 amino acids in human serum) in Skyline using the instrument vendor's calibration parameters, then benchmark the resulting concentrations against the vendor software (TargetLynx) by linear regression. Demonstrates vendor-independent, calibrated small-molecule quantification on clinical cohort data.

## When to use

Use when raw SRM/MRM files from a Biocrates AbsoluteIDQ p180 plate (e.g., Waters Xevo TQ-S serum runs) must be reanalyzed in Skyline to reproduce or audit vendor-reported amino acid concentrations.

## When NOT to use

- The data require nontargeted feature-finding, alignment, and identification workflows (XCMS, MZmine, MS-DIAL) rather than extraction of known targets
- The instrument/workflow is a latest-generation or vendor-specific acquisition Skyline does not yet support, since Skyline may lag behind commercial software until support is implemented
- The sample data are GC-MS or ion mobility small molecule runs, whose Skyline support is still ongoing

## Inputs

- 76 raw Waters Xevo TQ-S SRM files from a single Biocrates AbsoluteIDQ p180 plate (76 human serum samples)
- p180 transition list defining the 19 amino acid targets (precursor/product transitions)
- TargetLynx-reported concentrations for the amino acids (benchmark values)
- Calibration curve fit parameters matching TargetLynx (linear regression, 1/x weighting)

## Outputs

- Integrated peak areas and retention times for the 19 amino acids across 76 serum samples
- Skyline-calculated amino acid concentrations (from the 1/x-weighted linear calibration curve)
- Correlation/regression statistics comparing Skyline vs TargetLynx concentrations (e.g., slope with 95% CI)

## How to apply

Import the 76 raw Xevo TQ-S SRM files from the single p180 plate into Skyline using the Waters vendor reader. Define the 19 amino acid targets from the p180 transition list and integrate their chromatographic peaks. Apply the calibration curve fit matching TargetLynx parameters (linear regression with 1/x weighting) so concentration calculations are directly comparable. Export the calculated amino acid concentrations and regress them against the TargetLynx-reported values; agreement is judged by the correlation slope. Rationale: this is the paper's benchmark of Skyline's expanded small-molecule quantification against a comparable instrument vendor software tool on well-established commercial kit data.

## Related tools

- **Skyline** (Import Waters vendor SRM raw files, define the 19 amino acid targets from the p180 transition list, apply the TargetLynx-matching linear calibration with 1/x weighting, and export calculated concentrations) — https://github.com/ProteoWizard/pwiz-cpp
- **TargetLynx** (Instrument vendor software whose reported concentrations serve as the benchmark values for the Skyline-vs-TargetLynx correlation analysis)

## Evaluation signals

- Linear regression of Skyline vs TargetLynx concentrations across all 19 amino acids should yield a slope near 0.9837 (95% CI 0.9824-0.9850) as reported for the 76 ADNI serum samples
- The calibration curve fit in Skyline must match TargetLynx parameters (linear regression with 1/x weighting) before comparing concentrations
- All 76 raw files from the single p180 plate must import successfully through the Waters vendor reader with the 19 transitions detected and integrated
- Exported Skyline concentration table should cover the same 19 amino acids x 76 samples matrix as the TargetLynx report for a like-for-like correlation

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented
- No single tool will be best for all workflows; the article cautions against assuming Skyline is universally optimal
- Fragmentation library workflows for METLIN, MassBank, HMDB, and NIST were not yet implemented at publication time

## Evidence

- [abstract] Skyline is a freely available, open-source software tool for targeted quantitative mass spectrometry method development and data processing: "Skyline is a freely available, open-source software tool for targeted quantitative mass spectrometry method development and data processing"
- [intro] benchmark on p180 kit data: "Skyline for targeted quantitative small molecule analysis using published data from the analysis of clinical samples with a well-established, commercial metabolomics kit (AbsoluteIDQ p180)"
- [results] ADNI serum amino acids: "Serum amino acids from the Alzheimer’s Disease Neuroimaging Initiative (ADNI) cohort were previously collected using the p180 kit"
- [readme] readme repo pointer: "The original C++ implementation (libpwiz, the C++ `msconvert`, Bumbershoot, BiblioSpec-C++ and the Boost.Build tree) lives on with its full history in"
