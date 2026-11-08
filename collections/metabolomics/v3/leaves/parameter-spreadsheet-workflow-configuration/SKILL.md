---
name: parameter-spreadsheet-workflow-configuration
description: Use when processing raw untargeted LC/HRMS files (mzXML, mzML, netCDF)
  with IDSL.IPA and you must set data/output locations, peak-detection and recursive
  mass correction options (PARAM0019-PARAM0020) before calling IPA_workflow.
when_to_use_negative:
- Input is already an aligned peak table or feature matrix — the spreadsheet-driven
  IPA workflow is for raw MS1 files, and further processing (adduct/fragment identification,
  statistics filtering) is handled downstream
- Structural annotation requires MS2 spectra — IDSL.IPA/UFA workflows operate on MS1
  only; use MS2-based annotation tools instead
- You need interactive, non-scripted processing with no reproducible parameter record
  — the spreadsheet interface is specifically designed for scripted, population-scale
  (n > 500) batch runs
tools:
- name: IDSL.IPA
  role: R package whose IPA_workflow function consumes the filled parameter spreadsheet
    to run EIC candidate generation, peak detection, peak property evaluation, recursive
    mass correction and peak annotation
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.UFA
  role: R package whose UFA_workflow function consumes its own parameter spreadsheet
    to annotate IPA peaklists with molecular formulas via isotopic profile matching
  repo: https://github.com/idslme/IDSL.UFA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:98462790d7b6559a1ee86c123c349da482f7c02dd3c15b86d832c68e7f3ca634
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# parameter-spreadsheet-workflow-configuration

## Summary

Configure and run IDSL-family (IDSL.IPA/UFA) R workflows by filling a downloadable parameter spreadsheet (e.g. PARAM0007 for raw LC/HRMS data location, PARAM0010 for output, PARAM0019-PARAM0020 for recursive mass correction) and passing its address to the workflow function (IPA_workflow / UFA_workflow).

## When to use

Use when processing raw untargeted LC/HRMS files (mzXML, mzML, netCDF) with IDSL.IPA and you must set data/output locations, peak-detection and recursive mass correction options (PARAM0019-PARAM0020) before calling IPA_workflow.

## When NOT to use

- Input is already an aligned peak table or feature matrix — the spreadsheet-driven IPA workflow is for raw MS1 files, and further processing (adduct/fragment identification, statistics filtering) is handled downstream
- Structural annotation requires MS2 spectra — IDSL.IPA/UFA workflows operate on MS1 only; use MS2-based annotation tools instead
- You need interactive, non-scripted processing with no reproducible parameter record — the spreadsheet interface is specifically designed for scripted, population-scale (n > 500) batch runs

## Inputs

- Raw MS1-level LC/HRMS data files (mzXML, mzML, netCDF), e.g. 003.d from study MTBLS1684
- Downloaded parameter spreadsheet (IPA_parameters.xlsx / UFA_parameters.xlsx) to be filled with PARAM fields
- For IDSL.UFA: pre-calculated IPDB (.Rdata) isotopic profile database and IDSL.IPA 'peaklists' and 'peak_alignment' output directories

## Outputs

- Individual peaklists directory containing per-file peak tables (m/z-RT-intensity)
- peak_alignment directory with aligned peak table across samples
- MS1 processed data output location (for IDSL.UFA: molecular formula annotations with candidate ranking)
- Before/after peak counts for evaluating recursive mass correction (e.g. recursive_mass_correction_003d.csv)

## How to apply

Download the tool's parameter spreadsheet (e.g. IPA_parameters.xlsx or UFA_parameters.xlsx); for IDSL.IPA set PARAM0007 to the location of the raw LC/HRMS file (e.g. 003.d from MTBLS1684) and PARAM0010 to the output location, then optionally enable recursive mass correction via PARAM0019-PARAM0020 so peaks are re-checked against the chromatogram trace of the intensity-weighted average mass (m/z-bar). Pass the spreadsheet address to IPA_workflow in R (install.packages("IDSL.IPA")) to generate individual peaklists in a 'peaklists' directory and aligned tables in a 'peak_alignment' directory. IDSL.UFA similarly requires 30 parameters in 4 sections (e.g. PARAM0004 for the IPDB .Rdata address, PARAM0011/PARAM0012 for the IPA peaklists and peak_alignment directories, PARAM0014 for output, PARAM0006 for aggregating annotated formulas on the aligned table). The rationale is that the spreadsheet is the sole, versioned interface to pipeline options, making runs reproducible and letting custom R-scripts be generated via https://ipa.idsl.me.

## Related tools

- **IDSL.IPA** (R package whose IPA_workflow function consumes the filled parameter spreadsheet to run EIC candidate generation, peak detection, peak property evaluation, recursive mass correction and peak annotation) — https://github.com/idslme/IDSL.IPA
- **IDSL.UFA** (R package whose UFA_workflow function consumes its own parameter spreadsheet to annotate IPA peaklists with molecular formulas via isotopic profile matching) — https://github.com/idslme/IDSL.UFA

## Examples

```
install.packages("IDSL.IPA"); IDSL.IPA::IPA_workflow("path/to/IPA_parameters.xlsx")
```

## Evaluation signals

- The 'peaklists' directory is created and populated with one peaklist per input raw file, and a 'peak_alignment' directory appears when multiple samples are processed
- For recursive mass correction validation on file 003.d (MTBLS1684): removed-peak records must total 2,679 of 23,740 pre-recursion peaks (~11%), matching the reported m/z-change distribution for the intensity-weighted average mass
- All required PARAM fields are non-empty and point to existing paths (e.g. PARAM0007 data location, PARAM0010 output location; UFA's PARAM0004 IPDB address, PARAM0011/PARAM0012 IPA output directories)
- Peak counts are consistent with the article's reference run (IDSL.IPA detected 20,253 total peaks for 003.d under the reported settings; the 23,740 figure corresponds to the pre-recursion set)
- Re-running IPA_workflow with the same spreadsheet reproduces identical peak counts, confirming the spreadsheet fully parameterizes the run

## Limitations

- Parameter numbering and semantics differ per package and version (IDSL.IPA uses PARAM0007/PARAM0010/PARAM0019-PARAM0020; IDSL.UFA uses 30 parameters across 4 sections such as PARAM0004, PARAM0009, PARAM0011, PARAM0012, PARAM0014), so spreadsheets are not interchangeable
- The article does not state the IDSL.IPA software version used, so exact default parameter values may differ across CRAN releases
- IDSL.IPA aligned matrices still require additional downstream processing (adducts/fragments, statistics filtering, molecular formula annotation), which the spreadsheet workflow does not cover

## Evidence
