---
name: chromatographic-peak-ms2-spectra-extraction
description: Use when you have an xcms result object with centWave-detected chromatographic
  peaks (e.g. from an mzML LC-MS/MS run like HighIS_Mix07_CE20_POS.mzML) and need
  the tandem MS2 spectra behind those peaks to compare against a reference library
  such as an HMDB 5.0 CompDb.
when_to_use_negative:
- Your MS2 spectra already exist as standalone files (MGF, MSP, mzML) — import them
  directly into a Spectra object with MsBackendMgf/MsBackendMsp instead of reconstructing
  from an xcms result
- The dataset is MS1-only (no tandem MS2 scans were acquired) — chromPeakSpectra cannot
  produce fragment spectra; restrict annotation to matchMz/MzRtParam matching
- You only need to identify peaks by precursor m/z and retention time (matchMz or
  MzRtParam) — MS2 extraction is unnecessary for MS1-level annotation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: xcms
  role: Provides the result object and the chromPeakSpectra function that extracts
    MS2 spectra for selected chromatographic peaks
  repo: https://github.com/jorainer/MetaboAnnotationTutorials
- name: Spectra
  role: Defines the Spectra container returned by chromPeakSpectra and supplies filterIntensity/addProcessing/lengths
    used to clean and scale the extracted spectra
  repo: https://rformassspectrometry.github.io/Spectra/
- name: MetaboAnnotation
  role: Consumes the cleaned query Spectra downstream in matchSpectra against a reference
    library
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: MetaboAnnotationTutorials
  role: Distributes the example mzML data (HighIS_Mix07_CE20_POS.mzML) and the documented
    workflow this extraction step belongs to
  repo: https://github.com/jorainer/MetaboAnnotationTutorials
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:d76b599f04b666a1367fdd5d0e711c1d3d8038bdc836a0413b3e86be001315e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jorainer/MetaboAnnotationTutorials
  tools:
  - CompoundDb
  - MetaboAnnotation
  - MetaboAnnotationTutorials
  - R
  - Spectra
  - xcms
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# chromatographic-peak-ms2-spectra-extraction

## Summary

Extracts MS2 fragment spectra associated with xcms-detected chromatographic peaks from an xcms result object using chromPeakSpectra, producing a Spectra object ready for cleaning (5% BPI intensity filter, ≥3-peak filter, 0–100 intensity scaling) and downstream MS2 library matching with matchSpectra.

## When to use

Use when you have an xcms result object with centWave-detected chromatographic peaks (e.g. from an mzML LC-MS/MS run like HighIS_Mix07_CE20_POS.mzML) and need the tandem MS2 spectra behind those peaks to compare against a reference library such as an HMDB 5.0 CompDb.

## When NOT to use

- Your MS2 spectra already exist as standalone files (MGF, MSP, mzML) — import them directly into a Spectra object with MsBackendMgf/MsBackendMsp instead of reconstructing from an xcms result
- The dataset is MS1-only (no tandem MS2 scans were acquired) — chromPeakSpectra cannot produce fragment spectra; restrict annotation to matchMz/MzRtParam matching
- You only need to identify peaks by precursor m/z and retention time (matchMz or MzRtParam) — MS2 extraction is unnecessary for MS1-level annotation

## Inputs

- xcms result object with detected/refined chromatographic peaks (from mzML input, e.g. HighIS_Mix07_CE20_POS.mzML)
- data.frame of annotated chromatographic peaks with a peak_id column (e.g. from matchMz with Mass2MzParam, adducts [M+H]+ and [M+Na]+, ppm = 10)

## Outputs

- Spectra object of MS2 spectra keyed to the selected chromatographic peaks
- Cleaned Spectra object with <5% BPI peaks removed, spectra with <3 peaks dropped, and intensities scaled to 0–100 (ready as query for matchSpectra)

## How to apply

After peak detection (findChromPeaks with centWave) and peak refinement (refineChromPeaks with MergeNeighboringPeaksParam(expandRt = 3)), call chromPeakSpectra on the xcms result object with return.type = "Spectra" and peaks = pks_match$peak_id to pull only the MS2 spectra for the peaks of interest (e.g. those annotated via matchMz with Mass2MzParam(ppm = 10)). The rationale is that MS2-based annotation requires fragment spectra, but these are not in the peak table — chromPeakSpectra reconstructs them from the raw data for each chromatographic peak. Then clean the extracted Spectra: filterIntensity to remove spectral peaks below 5% of each spectrum's base peak intensity, drop spectra with fewer than 3 peaks (lengths(std_spectra) > 2), and scale intensities to 0–100 via addProcessing with a custom scaling function so similarity scores are comparable to the reference library.

## Related tools

- **xcms** (Provides the result object and the chromPeakSpectra function that extracts MS2 spectra for selected chromatographic peaks) — https://github.com/jorainer/MetaboAnnotationTutorials
- **Spectra** (Defines the Spectra container returned by chromPeakSpectra and supplies filterIntensity/addProcessing/lengths used to clean and scale the extracted spectra) — https://rformassspectrometry.github.io/Spectra/
- **MetaboAnnotation** (Consumes the cleaned query Spectra downstream in matchSpectra against a reference library) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **MetaboAnnotationTutorials** (Distributes the example mzML data (HighIS_Mix07_CE20_POS.mzML) and the documented workflow this extraction step belongs to) — https://github.com/jorainer/MetaboAnnotationTutorials

## Examples

```
std_spectra <- chromPeakSpectra(std_data, return.type = "Spectra", peaks = pks_match$peak_id); std_spectra <- filterIntensity(std_spectra, intensity = low_int); std_spectra <- std_spectra[lengths(std_spectra) > 2]
```

## Evaluation signals

- chromPeakSpectra was called with return.type = "Spectra" and the output is a Spectra object with one entry per requested peak that has MS2 data
- After filtering, no spectrum retains fragment peaks below 5% of its base peak intensity, and no spectrum has fewer than 3 peaks (lengths > 2)
- Spectral intensities are scaled within each spectrum to the 0–100 range before similarity computation
- When used as a query against the HMDB 5.0 CompDb with CompareSpectraParam(ppm = 50, requirePrecursor = FALSE), extracted spectra yield matches with scores in [0, 1] (normalized dot-product, default 0.7 threshold), and the first query spectrum matches Caffeine as in the tutorial
- whichQuery identifies which chromatographic peaks received annotation, confirming spectra extraction covered the intended peak set

## Limitations

- MS2 spectra are only available for chromatographic peaks that actually triggered MS2 scans; peaks without associated spectra are dropped after the ≥3-peak filter
- Filtering below 5% of base peak intensity and requiring ≥3 peaks removes noisy/empty spectra but may also discard genuine low-intensity fragments
- The tutorial sets requirePrecursor = FALSE in the downstream matchSpectra step, so some matches may have precursor mismatch and need manual confirmation

## Evidence

- [methods] extraction of MS2 spectra for annotated chromatographic peaks with chromPeakSpectra: [evidence span withheld — anchor status: not_found]
- [methods] Remove peaks with an intensity below 5% of BPI: "' Remove peaks with an intensity below 5% of BPI
std_spectra <- filterIntensity(std_spectra, intensity = low_int)'"
- [methods] spectra with fewer than 3 peaks removed: "std_spectra <- std_spectra[lengths(std_spectra) > 2]"
- [methods] xcms data distribution and peak detection context: "Below we load the data which is distributed within the `MetaboAnnotationTutorials` package and perform the chromatographic peak detection using the `r Biocpkg("xcms")` package."
- [readme] tutorial package provides the workflow and test data: "This repository provides use cases and examples for the annotation of non-target
metabolomics or small compound MS data using the `MetaboAnnotation` and
`MetaboCoreUtils` R packages."
