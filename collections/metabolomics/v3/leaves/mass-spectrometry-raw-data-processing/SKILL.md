---
name: mass-spectrometry-raw-data-processing
description: Use when the input is raw MS1-level LC/HRMS data (mzXML, mzML, or netCDF)
  from population-scale untargeted metabolomics/exposomics studies and you need a
  comprehensive, high-quality peak table of carbon-containing organic small molecules
  rather than alignment-ready region-of-interest output.
when_to_use_negative:
- Input is already an aligned peak table or processed data matrix — the pipeline expects
  raw mzXML/mzML/netCDF spectral data.
- The goal is MS2/fragmentation spectral deconvolution or structural annotation from
  tandem spectra — that is the domain of IDSL.CSA or MS2-based annotation tools, not
  IDSL.IPA.
- Peak detection must cover all ions regardless of isotopic signature (e.g. non-carbon
  inorganic species or standards-focused low-noise runs) — IDSL.IPA explicitly isolates
  only signals for carbon-containing compounds via 12C/13C pairing, which would discard
  such peaks.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.IPA
  role: 'Core pipeline tool: implements the 12C/13C ion-pairing, EIC peak detection,
    recursive mass correction, peak quality evaluation, retention time correction,
    alignment and annotation steps.'
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Runtime environment for the IDSL.IPA package; the workflow is invoked as an
    R function (`IPA_workflow`) with the parameter spreadsheet as input.
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: enviPat
  role: Used for calculating theoretical isotopic profiles for molecular formulas,
    complementing IDSL.IPA's isotopologue-pairing logic.
- name: IDSL.UFA
  role: Downstream annotation package that consumes IDSL.IPA peaklists and aligned
    peak tables to annotate molecular formulas via isotopic profile matching using
    only MS1 data.
  repo: https://github.com/idslme/IDSL.UFA
- name: IDSL.CSA
  role: Downstream companion package for deconvoluting fragmentation spectra (MS1-only
    CSA, DDA, DIA) to improve chemical structure annotation of IDSL.IPA peaks.
  repo: https://github.com/idslme/IDSL.CSA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:c449f2c163739efdeb309042525733055ad39680e326222b383b7662c1f8d1da
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.CSA
  - IDSL.IPA
  - IDSL.UFA
  - R
  - enviPat
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mass-spectrometry-raw-data-processing

## Summary

Process raw untargeted LC/HRMS files (mzXML/mzML/netCDF) into peak intensity data matrices by pairing 12C/13C isotopologues in individual mass spectra and then detecting chromatographic peaks on extracted ion chromatograms, as implemented in the IDSL.IPA R package.

## When to use

Use when the input is raw MS1-level LC/HRMS data (mzXML, mzML, or netCDF) from population-scale untargeted metabolomics/exposomics studies and you need a comprehensive, high-quality peak table of carbon-containing organic small molecules rather than alignment-ready region-of-interest output.

## When NOT to use

- Input is already an aligned peak table or processed data matrix — the pipeline expects raw mzXML/mzML/netCDF spectral data.
- The goal is MS2/fragmentation spectral deconvolution or structural annotation from tandem spectra — that is the domain of IDSL.CSA or MS2-based annotation tools, not IDSL.IPA.
- Peak detection must cover all ions regardless of isotopic signature (e.g. non-carbon inorganic species or standards-focused low-noise runs) — IDSL.IPA explicitly isolates only signals for carbon-containing compounds via 12C/13C pairing, which would discard such peaks.

## Inputs

- Raw MS1-level LC/HRMS data files in mzXML, mzML, or netCDF format (e.g. Thermo 003.d from MTBLS1684)
- IPA parameter spreadsheet (IPA_parameters.xlsx) with data location, output location and thread count set
- Reference database of m/z and retention time values for peak annotation (optional step)
- Retention index marker information for retention time correction across batches

## Outputs

- Individual peaklist table per data file (peaks with m/z, retention time, chromatographic peak properties)
- Aligned peak-intensity data matrix across samples for carbon-containing compounds
- Annotated peak list cross-referenced against a reference database of m/z and retention time

## How to apply

Run the IDSL.IPA R package's `IPA_workflow` on an IPA parameter spreadsheet: set PARAM0007 to the MS1-level HRMS data location, PARAM0010 to the output location, and PARAM0006 to the number of processing threads. The pipeline first pairs all 12C/13C isotopologues in each individual mass spectrum to generate a list of candidate monoisotopic m/z values and removes redundant isotope peaks, keeping only the most abundant monoisotopic isotopologues for downstream processing. It then detects and characterizes chromatographic peaks on extracted ion chromatograms via recursive mass correction, peak smoothing, and baseline development for local noise measurement, retaining only peaks that pass IDSL.IPA peak-quality thresholds (including S/N, peak width and asymmetry). For multi-file studies, retention time is corrected and peaks are cross-referenced across samples via a dynamic retention index marker approach, and peak detection/alignment steps are parallelized (n=200–1,600 samples validated). As a sanity check on a single file, the pipeline reported 20,253 total peaks for test file 003.d from the MTBLS1684 study.

## Related tools

- **IDSL.IPA** (Core pipeline tool: implements the 12C/13C ion-pairing, EIC peak detection, recursive mass correction, peak quality evaluation, retention time correction, alignment and annotation steps.) — https://github.com/idslme/IDSL.IPA
- **R** (Runtime environment for the IDSL.IPA package; the workflow is invoked as an R function (`IPA_workflow`) with the parameter spreadsheet as input.) — https://cran.r-project.org/package=IDSL.IPA
- **enviPat** (Used for calculating theoretical isotopic profiles for molecular formulas, complementing IDSL.IPA's isotopologue-pairing logic.)
- **IDSL.UFA** (Downstream annotation package that consumes IDSL.IPA peaklists and aligned peak tables to annotate molecular formulas via isotopic profile matching using only MS1 data.) — https://github.com/idslme/IDSL.UFA
- **IDSL.CSA** (Downstream companion package for deconvoluting fragmentation spectra (MS1-only CSA, DDA, DIA) to improve chemical structure annotation of IDSL.IPA peaks.) — https://github.com/idslme/IDSL.CSA

## Examples

```
library(IDSL.IPA); IPA_workflow("path/to/IPA_parameters.xlsx")
```

## Evaluation signals

- On the MTBLS1684 test file 003.d, the per-file peaklist should contain 20,253 total peaks — an exact reproducibility check for the pipeline.
- Every retained peak should correspond to a candidate m/z from a 12C/13C isotopologue pair; peaks failing the isotopologue pairing or peak-quality thresholds must be absent from the output.
- Each individual 12C m/z peak should appear in only one m/z cluster, not overlapping clusters.
- Only the most abundant monoisotopic isotopologues from 12C/13C isotope clusters should be retained; redundant isotope peaks must be removed from the peaklist.
- For multi-sample studies, retention time drift after correction should show no systematic bias (verified via retention index and polynomial regression correction plots on e.g. MTBLS136, 1649 samples).

## Limitations

- A majority of rejected EICs had low abundance but passed the S/N ratio threshold; additional rules are still needed to automatically filter out these rejected low-abundance peaks.
- IDSL.IPA aligned data matrices need to be further processed for identifying adducts and fragments, filtering for statistics and annotating molecular formulas by additional computational tools (e.g. IDSL.UFA/UFAx).
- IDSL.IPA only detects the most abundant monoisotopic isotopologues from 12C/13C isotope clusters and removes redundant isotope peaks, so signals for non-carbon or isotope-depleted species are outside its design scope.

## Evidence

- [readme] MS2 deconvolution scope: [evidence span withheld — anchor status: external_doc]
