---
name: metabolomics-benchmark-count-reproduction
description: Use when validating or reproducing quantitative claims from a metabolomics
  methods paper — e.g., '105 of 177 compounds yielded CSA spectra, 25,351 total spectra'
  — using the authors' own public dataset and toolchain.
when_to_use_negative:
- The goal is novel discovery or annotation of a new dataset rather than verifying
  a published paper's quantitative claims
- Input is a pre-computed feature/peak table with no raw mzXML/mzML files, since IDSL.IPA
  peak detection must be re-run from raw data
- The study relies on MS/MS (DDA/DIA) fragmentation data rather than MS1-only composite
  spectra, so the CSA benchmark does not apply
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: IDSL.IPA
  role: 'Upstream peak picking: processes the 600 raw mzXML/mzML files to generate
    ''peaklists'' and ''peak_alignment'' directories with m/z-RT chromatographic information'
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.CSA
  role: Runs the targeted-mode CSA workflow from the 39-parameter spreadsheet to deconvolute
    composite spectra and write the .msp library and aligned tables
  repo: https://github.com/idslme/IDSL.CSA
- name: IDSL.FSA
  role: Searches the reproduced CSA .msp library against MS/MS mass spectral libraries
    (e.g., NIST 2020) to count library hits for the benchmark comparison
  repo: https://github.com/idslme/IDSL.FSA
- name: R
  role: Execution environment; all three packages are installed via install.packages()
    from CRAN
  repo: https://cran.r-project.org/package=IDSL.CSA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:805a741473cda2bc7b4dd2b307bdea4f37c66e2bff82fe9edfe0e3e9e9fa67e6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metabolomics-benchmark-count-reproduction

## Summary

Reproduces published benchmark counts (e.g., compounds with CSA spectra, total deconvoluted spectra, variants per compound, MS/MS library hits) from a metabolomics software paper by re-running its documented workflow on the same public dataset and comparing each metric against the reported values.

## When to use

Use when validating or reproducing quantitative claims from a metabolomics methods paper — e.g., '105 of 177 compounds yielded CSA spectra, 25,351 total spectra' — using the authors' own public dataset and toolchain.

## When NOT to use

- The goal is novel discovery or annotation of a new dataset rather than verifying a published paper's quantitative claims
- Input is a pre-computed feature/peak table with no raw mzXML/mzML files, since IDSL.IPA peak detection must be re-run from raw data
- The study relies on MS/MS (DDA/DIA) fragmentation data rather than MS1-only composite spectra, so the CSA benchmark does not apply

## Inputs

- 600 raw LC/HRMS MS1-only data files (mzXML/mzML) from public study ST000923 (stool samples)
- Published annotation list used as the targeted reference library (177 compounds)
- IDSL.CSA parameter spreadsheet (CSA_parameters.xlsx) with CSA0004/0005/0008/0009/0011 filled
- Reference benchmark values from the paper (e.g., 105/177 compounds, 25,351 CSA spectra, 46 MS/MS library hits)

## Outputs

- CSA_MSP folder containing the .msp composite spectra library
- CSA_adduct_annotation peaklists and peak_alignment_subset tables
- aligned_spectra_table from the CSA aggregation step
- Reproduced metric set: compound coverage (target 105/177, 59.3%), total CSA spectra (target 25,351), median detection frequency (204; min 1, max 589), variants/compound (2.5), MS/MS library hits (46 via IDSL.FSA)
- Validation verdict per metric (match/no-match within tolerance)

## How to apply

Obtain the published study's raw data (for ST000923: 600 LC/HRMS mzXML/mzML files, MS1-only) and the paper's reference annotations (177 targeted compounds). Re-run the authors' exact pipeline: process files with IDSL.IPA to generate 'peaklists' and 'peak_alignment' directories, fill the IDSL.CSA parameter spreadsheet (39 parameters in 5 sections; set CSA0005 = HRMS data location, CSA0008 = peaklists, CSA0009 = peak_alignment, CSA0011 = output for .msp/EICs, CSA0004 = thread count), then run IDSL.CSA_workflow in targeted mode against the 177 published annotations. Parse the CSA0011 output (CSA_MSP .msp library, CSA_adduct_annotation, peak_alignment_subset, aligned_spectra_table) and count: compounds with ≥1 CSA spectrum, total deconvoluted CSA spectra, spectra variants per compound, and — after searching the .msp with IDSL.FSA against MS/MS libraries such as NIST 2020 — library hits. Declare reproduction successful only when each metric matches the published benchmark (105/177 = 59.3%, 25,351 spectra, median detection frequency 204, 2.5 variants/compound, 46 FSA hits) within the study's reporting tolerance.

## Related tools

- **IDSL.IPA** (Upstream peak picking: processes the 600 raw mzXML/mzML files to generate 'peaklists' and 'peak_alignment' directories with m/z-RT chromatographic information) — https://github.com/idslme/IDSL.IPA
- **IDSL.CSA** (Runs the targeted-mode CSA workflow from the 39-parameter spreadsheet to deconvolute composite spectra and write the .msp library and aligned tables) — https://github.com/idslme/IDSL.CSA
- **IDSL.FSA** (Searches the reproduced CSA .msp library against MS/MS mass spectral libraries (e.g., NIST 2020) to count library hits for the benchmark comparison) — https://github.com/idslme/IDSL.FSA
- **R** (Execution environment; all three packages are installed via install.packages() from CRAN) — https://cran.r-project.org/package=IDSL.CSA

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("Address of the CSA parameter spreadsheet")
```

## Evaluation signals

- Compound coverage matches 105 of 177 targeted compounds (59.3%) with at least one CSA spectrum
- Total deconvoluted CSA spectra equals 25,351 across the 600 samples
- Median CSA detection frequency is 204 (min 1, max 589) and mean spectra variants per compound is 2.5
- IDSL.FSA search of the reproduced .msp against MS/MS libraries yields 46 library hits
- All expected output artifacts exist at the CSA0011 address: CSA_MSP (.msp file), CSA_adduct_annotation, peak_alignment_subset, and aligned_spectra_table

## Limitations

- Reproduction requires the exact public dataset and the authors' 177-compound reference annotations; different input data will not reproduce the benchmark counts
- Only 59.3% of targeted compounds yielded CSA spectra, so absence of a spectrum for a compound is an expected outcome, not necessarily a pipeline failure
- Compute-dependent parameters (CSA0004 thread count) must be set per available hardware, and the study used MS1-only data — MS2 availability is not assumed

## Evidence

- [other] For ST000923, 105 of the 177 targeted compounds (59.3%) had an associated CSA spectrum, with a median detection frequency of 204 (min 1, max 589), and a total of 25,351 CSA spectra were deconvoluted from the 600 samples.: [evidence span withheld — anchor status: not_found]
