---
name: untargeted-metabolomics-data-processing
description: Use when raw full-scan LC/HRMS data from population-scale untargeted
  metabolomics or exposomics studies (e.g. mzXML/mzML files from Orbitrap instruments)
  must be converted into a peak table without MS2 adduct/fragment handling.
when_to_use_negative:
- Input is already a processed/aligned feature table (the article notes IDSL.IPA aligned
  data matrices need to be further processed for adducts, fragments, statistics filtering
  and formula annotation by separate tools such as IDSL.UFA/UFAx/CSA).
- The analysis goal requires MS2 fragmentation spectra handling (e.g. DDA/DIA/SWATH
  deconvolution) — use IDSL.CSA for composite spectra analysis instead.
- Data are non-LC/HRMS (e.g. direct infusion MS or non-small-molecule proteomics data)
  since the pipeline is specifically built for organic small molecules in LC/HRMS
  untargeted studies.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: Core R package implementing the isotopologue pairing, EIC peak detection,
    mass/RT correction, and peak alignment workflow
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Runtime environment; IDSL.IPA is available in the R CRAN repository
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: enviPat
  role: Used for calculating theoretical isotopic profiles for molecular formulas
    to validate the R13C approach
- name: IDSL.UFA
  role: Downstream molecular formula annotation of the IDSL.IPA peaklists via isotopic
    profile matching (MS1 only)
  repo: https://github.com/idslme/IDSL.UFA
- name: IDSL.UFAx
  role: Downstream exhaustive elemental composition enumeration on IDSL.IPA peaks
  repo: https://github.com/idslme/IDSL.UFAx
- name: IDSL.CSA
  role: Downstream composite/fragmentation spectra (DDA/DIA/MSE/AIF/SWATH) deconvolution
    paired with IDSL.IPA peaklists
  repo: https://github.com/idslme/IDSL.CSA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:a6c9757c626ea35dccd94d730be1e6e71ca03fba7be330459b72d0e8c00b5565
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.CSA
  - IDSL.IPA
  - IDSL.UFA
  - IDSL.UFAx
  - MS-DIAL
  - MZmine
  - R
  - enviPat
  - xcms-CAMERA
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# untargeted-metabolomics-data-processing

## Summary

IDSL.IPA is an R package that processes raw untargeted LC/HRMS data (mzXML, mzML, netCDF) into comprehensive peak-intensity data matrices for carbon-containing organic small molecules, using 12C/13C isotopologue pairing in individual mass spectra followed by EIC peak detection, mass and retention-time correction, and peak quality filtering.

## When to use

Use when raw full-scan LC/HRMS data from population-scale untargeted metabolomics or exposomics studies (e.g. mzXML/mzML files from Orbitrap instruments) must be converted into a peak table without MS2 adduct/fragment handling.

## When NOT to use

- Input is already a processed/aligned feature table (the article notes IDSL.IPA aligned data matrices need to be further processed for adducts, fragments, statistics filtering and formula annotation by separate tools such as IDSL.UFA/UFAx/CSA).
- The analysis goal requires MS2 fragmentation spectra handling (e.g. DDA/DIA/SWATH deconvolution) — use IDSL.CSA for composite spectra analysis instead.
- Data are non-LC/HRMS (e.g. direct infusion MS or non-small-molecule proteomics data) since the pipeline is specifically built for organic small molecules in LC/HRMS untargeted studies.

## Inputs

- Raw MS1-level LC/HRMS data files (.mzXML, .mzML, .netCDF) from untargeted metabolomics studies (e.g. Thermo Fusion Orbitrap raw converted data)
- IPA parameter spreadsheet (IPA_parameters.xlsx) with user-selected parameters
- Optional retention index marker list for dynamic retention time correction
- Optional reference database of m/z and retention time for peak annotation

## Outputs

- Per-file peaklists with chromatographic information (m/z-RT) for detected monoisotopic peaks
- Aligned peak intensity data matrix across samples
- Peak property table (peak quality metrics, corrected mass, corrected retention time, retention index)
- Peak annotation against the reference m/z-RT database (when supplied)

## How to apply

Run the IDSL.IPA pipeline via the `IPA_workflow` function with a downloaded parameter spreadsheet (IPA_parameters.xlsx) as the single input, pointing at a directory of MS1-level mzXML/mzML/netCDF files. The pipeline first pairs all potential 12C/13C isotopologues in each individual mass spectrum to generate candidate m/z values, so only m/z values corresponding to the 12C isotopologue are carried into peak detection and each 12C peak is used in exactly one m/z cluster (no overlapping clusters), minimizing redundant isotope peaks. Candidate EICs are then smoothed, a local-noise baseline is developed, and each chromatographic peak is subject to peak quality determination; only the most abundant monoisotopic isotopologues from 12C/13C isotope clusters are retained. Mass correction uses an intensity weighted average mass, and retention time is corrected across multiple batches using a dynamic retention index marker approach (which removed the systematic RT drift for the 1649-sample MTBLS136 study). For large studies, enable the parallel computation of peak detection and alignment steps (studies ranging from 200 to 1600 samples were successfully evaluated).

## Related tools

- **IDSL.IPA** (Core R package implementing the isotopologue pairing, EIC peak detection, mass/RT correction, and peak alignment workflow) — https://github.com/idslme/IDSL.IPA
- **R** (Runtime environment; IDSL.IPA is available in the R CRAN repository) — https://cran.r-project.org/package=IDSL.IPA
- **enviPat** (Used for calculating theoretical isotopic profiles for molecular formulas to validate the R13C approach)
- **IDSL.UFA** (Downstream molecular formula annotation of the IDSL.IPA peaklists via isotopic profile matching (MS1 only)) — https://github.com/idslme/IDSL.UFA
- **IDSL.UFAx** (Downstream exhaustive elemental composition enumeration on IDSL.IPA peaks) — https://github.com/idslme/IDSL.UFAx
- **IDSL.CSA** (Downstream composite/fragmentation spectra (DDA/DIA/MSE/AIF/SWATH) deconvolution paired with IDSL.IPA peaklists) — https://github.com/idslme/IDSL.CSA

## Examples

```
library(IDSL.IPA); IPA_workflow("Address of the IPA parameter spreadsheet")
```

## Evaluation signals

- Peak-count sensitivity check: on the MTBLS1684 test file 003.d, IDSL.IPA detected 20,253 total peaks, a 1.6×, 5.4×, and 2.2× fold increase over MZmine 2, xcms-CAMERA, and MS-DIAL respectively — similar sensitivity should be observed on comparable LC/HRMS data.
- Carbon-number prediction check: for detected peaks, predicted carbon number from R13C¯≈1.0816*Cn (Cn ≤ 92) should correlate with theoretical values for authentic standards (Spearman ρ ≤ 0.92, p ≤ 2.2e−16 for 367 standards).
- Retention time drift check: after dynamic retention index marker correction, systematic RT drift across batches/samples should be removed (validated on MTBLS136, 1649 samples).
- Schema check: output aligned matrix should have one row per m/z-RT peak, one column per sample, with no overlapping 12C m/z clusters (each 12C peak used in only one cluster).
- No redundant isotope peaks: the output should contain only the most abundant monoisotopic isotopologue per 12C/13C isotope cluster.

## Limitations

- A majority of the rejected EICs in the benchmark had low abundance but passed the S/N threshold — the article explicitly notes 'there is a need to develop additional rules to automatically filter out these rejected low abundant peaks'.
- IDSL.IPA aligned data matrices must be further processed by additional computational tools for adduct/fragment identification, statistical filtering and molecular formula annotation.
- The pipeline is limited to MS1-based peak detection of organic small molecules; it does not deconvolute MS2 fragmentation spectra (that is IDSL.CSA's role).
- Software version was not stated in the publication's discussion section.

## Evidence

- [readme] README: sensitivity claim: [evidence span withheld — anchor status: external_doc]
- [readme] README: supported file formats: [evidence span withheld — anchor status: not_found]
