---
name: csa-parameter-spreadsheet-configuration
description: Use when MS1-level mzXML/mzML/netCDF files have already been processed
  by IDSL.IPA into 'peaklists' and 'peak_alignment' directories and a CSA composite
  spectra (.msp) library must be generated.
when_to_use_negative:
- No IDSL.IPA peaklists/peak_alignment directories exist yet — run IDSL.IPA first,
  as the CSA workflow consumes its chromatographic m/z-RT outputs
- The goal is DDA or DIA (MSE/AIF/SWATH-MS) spectral deconvolution rather than MS1-only
  composite spectra analysis — select the corresponding workflow mode instead of PARAM0001
  = YES
- Input is already an .msp/.mgf spectra file to be annotated — that is the IDSL.FSA
  search workflow, not CSA library creation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: IDSL.CSA
  role: R package whose parameter spreadsheet (39 parameters, 5 sections) is configured
    and passed to IDSL.CSA_workflow to run CSA deconvolution
  repo: https://github.com/idslme/IDSL.CSA
- name: IDSL.IPA
  role: Upstream R workflow that generates the 'peaklists' and 'peak_alignment' directories
    referenced by CSA0008 and CSA0009
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.FSA
  role: Companion R package used downstream to annotate the .msp library produced
    at CSA0011 via mass spectral similarity searches
  repo: https://github.com/idslme/IDSL.FSA
- name: R
  role: Runtime environment; IDSL.CSA is installed from CRAN and invoked via IDSL.CSA_workflow()
  repo: https://cran.r-project.org/package=IDSL.CSA
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:2860b827edb07698cd43caf3e61c2e99520b46efcee4b06f6a90c312e9c16132
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.CSA
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# csa-parameter-spreadsheet-configuration

## Summary

Configure and drive the IDSL.CSA workflow through its 39-parameter spreadsheet (5 sections), pointing CSA0005/CSA0008/CSA0009 at IDSL.IPA outputs and setting PARAM0001=YES for CSA-only composite spectra analysis of MS1-only LC/HRMS data.

## When to use

Use when MS1-level mzXML/mzML/netCDF files have already been processed by IDSL.IPA into 'peaklists' and 'peak_alignment' directories and a CSA composite spectra (.msp) library must be generated.

## When NOT to use

- No IDSL.IPA peaklists/peak_alignment directories exist yet — run IDSL.IPA first, as the CSA workflow consumes its chromatographic m/z-RT outputs
- The goal is DDA or DIA (MSE/AIF/SWATH-MS) spectral deconvolution rather than MS1-only composite spectra analysis — select the corresponding workflow mode instead of PARAM0001 = YES
- Input is already an .msp/.mgf spectra file to be annotated — that is the IDSL.FSA search workflow, not CSA library creation

## Inputs

- Raw LC/HRMS data files (mzXML, mzML, netCDF), MS1 level
- IDSL.IPA 'peaklists' directory (per-file chromatographic peak information)
- IDSL.IPA 'peak_alignment' directory (aligned m/z-RT peak table)
- IDSL.CSA parameter spreadsheet (CSA_parameters.xlsx), with optional reference annotation library for targeted mode

## Outputs

- CSA_MSP directory containing the composite spectra .msp library
- CSA_adduct_annotation peaklists with potential adduct information
- peak_alignment_subset tables (aligned peak subsets for major ions in each CSA cluster)
- aligned_spectra_table describing CSA aggregation on the aligned table
- Batch untargeted aligned extracted ion chromatogram (EIC) figures

## How to apply

First run IDSL.IPA on the raw LC/HRMS files (mzXML, mzML, netCDF) so that per-file peaklists and the aligned peak table (m/z-RT information) exist. Download the IDSL.CSA parameter spreadsheet (CSA_parameters.xlsx) — a full-scale CSA analysis requires 39 parameters distributed across 5 sections — and set PARAM0001 = YES in the 'Start' tab to run only the CSA workflow. Fill CSA0005 with the HRMS (MS1-level) data location, CSA0008 with the IDSL.IPA 'peaklists' directory, CSA0009 with the 'peak_alignment' directory, and CSA0011 with the output location for .msp files and EIC figures; scale CSA0004 to the available processing threads since the package parallelizes on Windows and Linux. In targeted mode, supply the published annotations (e.g., 177 compounds for ST000923, or 359 authentic standards) as the reference library. Execute 'IDSL.CSA_workflow("<spreadsheet address>")' in R; results appear under CSA0011 in the CSA_MSP, CSA_adduct_annotation, peak_alignment_subset, and aligned_spectra_table outputs.

## Related tools

- **IDSL.CSA** (R package whose parameter spreadsheet (39 parameters, 5 sections) is configured and passed to IDSL.CSA_workflow to run CSA deconvolution) — https://github.com/idslme/IDSL.CSA
- **IDSL.IPA** (Upstream R workflow that generates the 'peaklists' and 'peak_alignment' directories referenced by CSA0008 and CSA0009) — https://github.com/idslme/IDSL.IPA
- **IDSL.FSA** (Companion R package used downstream to annotate the .msp library produced at CSA0011 via mass spectral similarity searches) — https://github.com/idslme/IDSL.FSA
- **R** (Runtime environment; IDSL.CSA is installed from CRAN and invoked via IDSL.CSA_workflow()) — https://cran.r-project.org/package=IDSL.CSA

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("Address of the CSA parameter spreadsheet")
```

## Evaluation signals

- CSA_MSP directory at the CSA0011 output address contains a populated .msp composite spectra library file
- CSA_adduct_annotation, peak_alignment_subset, and aligned_spectra_table outputs are present and consistent with the aligned peak table from IDSL.IPA
- For ST000923 targeted reproduction, counts match published benchmarks: 105/177 compounds with CSA spectra, 25,351 total CSA spectra from 600 samples, ~2.5 variants/compound
- For the 359-standard test (MSV000088661), ~214/359 standards yield CSA spectra and up to 42 show in-source fragments matching MS/MS libraries
- Downstream IDSL.FSA search of the exported .msp against NIST 2020 recovers expected top hits (e.g., Purine, L-Kynurenine, N-Acetyl histidine, Homoserine, N-Acetylneuraminic acid for the pool_P1_A1-A12 test file)

## Limitations

- Only 5 path/thread parameters plus PARAM0001 are study-specific; the remaining of the 39 parameters across 5 sections govern deconvolution behavior and should be reviewed, not left blindly at defaults for unusual datasets
- CSA spectra yield is partial even for known compounds (59.3% of 177 targeted ST000923 compounds; 59.6% of 359 standards), so absence of a CSA spectrum does not imply absence of the compound
- Only a minority of CSA spectra contain in-source fragments resembling MS/MS reference spectra (up to 42/214, 19.6%), limiting library-search annotation rates

## Evidence

- [readme] CSA0005, CSA0008, CSA0009 and CSA0011 define data, peaklists, peak_alignment and output locations.: [evidence span withheld — anchor status: not_found]
- [methods] Workflow invocation of IDSL.CSA via the parameter spreadsheet address.: [evidence span withheld — anchor status: not_found]
