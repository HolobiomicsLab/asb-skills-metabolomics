---
name: mass-spectrometry-adduct-identification
description: Use when annotating MALDI-MSI peaks where suspected matrix or salt adducts
  (e.g., DHB) may inflate peak counts or cause false-positive metabolite identifications
  from exact mass alone.
when_to_use_negative:
- When the input is already a fully annotated feature table where adducts have been
  explicitly resolved and removed (e.g., post-annotation deconvolution), as this skill
  operates on raw peak m/z lists to discover adducts de novo.
- When the data are LC-MS/MS or non-imaging spectra without spatial/pixel information,
  since `corrPairsMSI()` co-localization confirmation relies on spatially resolved
  pixel intensities and `msimat` objects.
- When only exact-mass database annotation is needed and matrix/salt adducts are known
  to be negligible for the sample type—although the paper argues adducts are rarely
  negligible, the skill is not designed to replace conventional database searching
  itself.
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: mass2adduct
  role: R package that performs pairwise mass-difference calculation (`massdiff`),
    adduct matching (`adductMatch`, `topAdducts`), peak-pair extraction (`diffGetPeaks`),
    and spatial correlation testing (`corrPairsMSI`) to identify metabolite–matrix
    adducts in MALDI-MSI data
  repo: https://github.com/kbseah/mass2adduct
- name: R
  role: Runtime environment required to install and execute mass2adduct (installed
    via `install_github` with `devtools`), and to import data with `msimat()` or convert
    Cardinal objects
- name: Cardinal
  role: Used upstream to preprocess imzML data (`normalize()`, `smoothSignal()`, `reduceBaseline()`,
    `peakBin()`) and convert the resulting `MSProcessedImagingExperiment`/`MSContinuousImagingExperiment`
    objects to `msimat` format via `cardinal2msimat()`
- name: imzML
  role: Emerging standard XML-based file format read by Cardinal's `readImzML()` as
    an upstream input step before conversion to mass2adduct's `msimat`
- name: SCiLS
  role: Source of plain-text CSV peak-intensity exports that are imported via `msimat()`
    for adduct analysis
- name: MSiReader
  role: Source of 'intensity export' CSV files that serve as input to `msimat()`;
    also provides the `msimunging.pl` Perl script workaround for large-file reformatting
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:e25146d6dd1bb4421b2eaf0b97b02093ff744f99a57b11756663b32c09b81cb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbseah/mass2adduct
  tools:
  - Cardinal
  - MSiReader
  - R
  - SCiLS
  - imzML
  - mass2adduct
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# mass-spectrometry-adduct-identification

## Summary

A computational skill for identifying and quantifying metabolite adducts (e.g., DHB, salt) in MALDI-MSI data by computing all pairwise mass differences between detected m/z peaks, binning them, and matching them against a reference adduct list (e.g., DHB-H₂O at 136.016 Da) using spatial co-localization as orthogonal confirmation.

## When to use

Use when annotating MALDI-MSI peaks where suspected matrix or salt adducts (e.g., DHB) may inflate peak counts or cause false-positive metabolite identifications from exact mass alone.

## When NOT to use

- When the input is already a fully annotated feature table where adducts have been explicitly resolved and removed (e.g., post-annotation deconvolution), as this skill operates on raw peak m/z lists to discover adducts de novo.
- When the data are LC-MS/MS or non-imaging spectra without spatial/pixel information, since `corrPairsMSI()` co-localization confirmation relies on spatially resolved pixel intensities and `msimat` objects.
- When only exact-mass database annotation is needed and matrix/salt adducts are known to be negligible for the sample type—although the paper argues adducts are rarely negligible, the skill is not designed to replace conventional database searching itself.

## Inputs

- Plain-text CSV peak-intensity table exported from SCiLS or MSiReader (semicolon-delimited, as accepted by `msimat()`)
- Cardinal `MSProcessedImagingExperiment` or `MSContinuousImagingExperiment` object with peaks binned via `peakBin()`
- Plain text file of mass values (one per line), when pixel intensities are not needed
- Custom adduct reference data.frame with columns `name`, `formula`, `mass` (or built-in `adducts`/`adducts2`)

## Outputs

- `massdiff` object containing parent masses, derivative masses, and their pairwise differences
- Histogram of binned mass differences (standard R `hist` object) for adduct-matching
- Table from `adductMatch()` of counts and quantiles for known adducts
- Ranked list from `topAdducts()` of highest-counted mass differences with matches
- Subset `massdiff` object from `diffGetPeaks()` containing parent–adduct peak pairs for a specific adduct
- data.frame of Pearson correlation p-values (Bonferroni-corrected) from `corrPairsMSI()` confirming spatial co-localization

## How to apply

First, import peak-intensity tables from MSI software (SCiLS, MSiReader) into R as an `msimat` object via `msimat()` (or convert binned Cardinal `MSProcessedImagingExperiment` objects with `cardinal2msimat()`). Second, compute all pairwise m/z differences with `massdiff()`—these differences represent potential molecular adducts—and bin them into a histogram with a bin width matched to the instrument's known mass precision (e.g., 0.01 for the example data). Third, use `adductMatch()` to find closest-matching known adducts from the built-in `adducts`/`adducts2` reference data (or a custom data.frame with `name`, `formula`, `mass` columns), and `topAdducts()` to rank unassigned mass differences by occurrence. Fourth, for a candidate adduct like DHB-H₂O (mass 136.01600), extract the specific parent–derivative peak pairs with `diffGetPeaks(d.diff, mass=136.01600, mDa=1)` (±0.5 mDa window or ppm-based). Finally, confirm adduct assignment by testing spatial co-localization between parent and derivative ions using `corrPairsMSI()`, which applies a two-tailed Pearson correlation test with p<0.05 and Bonferroni correction—the rationale being that true parent–adduct pairs should be correlated in pixel intensity, unlike random mass-difference coincidences. Use `corrPairsMSIchunks()` with `mem.limit=N` if the peak table exceeds memory.

## Related tools

- **mass2adduct** (R package that performs pairwise mass-difference calculation (`massdiff`), adduct matching (`adductMatch`, `topAdducts`), peak-pair extraction (`diffGetPeaks`), and spatial correlation testing (`corrPairsMSI`) to identify metabolite–matrix adducts in MALDI-MSI data) — https://github.com/kbseah/mass2adduct
- **R** (Runtime environment required to install and execute mass2adduct (installed via `install_github` with `devtools`), and to import data with `msimat()` or convert Cardinal objects)
- **Cardinal** (Used upstream to preprocess imzML data (`normalize()`, `smoothSignal()`, `reduceBaseline()`, `peakBin()`) and convert the resulting `MSProcessedImagingExperiment`/`MSContinuousImagingExperiment` objects to `msimat` format via `cardinal2msimat()`)
- **imzML** (Emerging standard XML-based file format read by Cardinal's `readImzML()` as an upstream input step before conversion to mass2adduct's `msimat`)
- **SCiLS** (Source of plain-text CSV peak-intensity exports that are imported via `msimat()` for adduct analysis)
- **MSiReader** (Source of 'intensity export' CSV files that serve as input to `msimat()`; also provides the `msimunging.pl` Perl script workaround for large-file reformatting)

## Examples

```
library(mass2adduct); d <- msimat('msi.csv', sep=';'); d.diff <- massdiff(d); d.diff.hist <- hist(d.diff); adductMatch(d.diff.hist)
```

## Evaluation signals

- Confirm that the histogram of pairwise mass differences (`hist(d.diff)`) shows a clear bin at the expected adduct mass (e.g., 136.016 Da for DHB-H₂O) when plotted with `plot(d.diff.hist)`, and that `adductMatch()` returns a high count for that bin.
- Verify that `diffGetPeaks()` output contains parent–derivative pairs whose mass difference falls within the specified window (e.g., ±0.5 mDa around 136.01600), and that the derivative masses correspond to plausibly known parent ions.
- Check `corrPairsMSI()` output: a valid adduct assignment should yield p-values < 0.05 (Bonferroni-corrected) for the majority of parent–adduct pairs, indicating true spatial co-localization.
- Cross-check the fraction of total peaks attributable to adducts: the paper reports up to one third of total peak counts can be metabolite–matrix adducts, so if no adducts are found in a dataset where this is plausible, suspect input-format errors or bin-width miscalibration.
- If using `corrPairsMSIchunks()`, ensure results are consistent with the serial `corrPairsMSI()` run on a memory-limited subset, and that `mem.limit=N` is set to actual available RAM to avoid silent truncation.

## Limitations

- Measured mass differences are 'misleadingly precise' because measurement error is not inherent in the raw difference; bin width must be chosen manually based on known instrument mass precision (e.g., 0.01 in the example).
- Large CSV files (several GB) from SCiLS/MSiReader may exceed available memory when imported directly; the Perl script `msimunging.pl` can convert to triplet format but is still bounded by system RAM (e.g., a 40 Gb CSV with 32 Gb RAM will likely fail).
- `corrPairsMSI()` may be slow or memory-intensive on large datasets; use `corrPairsMSIchunks()` with an explicit `mem.limit` to avoid crashes.
- Adduct identification is putative; the method relies on co-localization and mass difference rather than MS/MS fragmentation, so co-eluting or spatially correlated but chemically unrelated peaks can produce false positives.
- The built-in `adducts` reference list is small and may not cover rare or sample-specific adducts; custom adduct data.frames must be supplied for comprehensive screening.

## Evidence

- [abstract] metabolite–matrix adducts on total peak counts (up to one third): "reveals a major influence of so far unrecognized metabolite–matrix adducts on total peak counts (up to one third)"
- [abstract] We developed a software package, mass2adduct, for the community: "We developed a software package, mass2adduct, for the community for an automated putative assignment and quantification of metabolite–matrix adducts"
- [readme] massdiff(d) pairwise mass differences represent potential molecular adducts: "Take all possible pairs of masses and calculate the mass difference for each pair. These mass differences represent potential molecular adducts."
- [readme] bin width depends on known mass precision, e.g., 0.01: [evidence span withheld — anchor status: external_doc]
- [readme] corrPairsMSI spatial correlation with Bonferroni-corrected p-values: "This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object."
- [readme] adductMatch finds known adducts by closest-matching bin: "The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above."
- [readme] diffGetPeaks for DHB-H2O with 1 mDa window: [evidence span withheld — anchor status: external_doc]
- [intro] gap in transforming measured m/z values to knowledge: "there is a gap in transforming the measured m/z values to knowledge. This means that we lack easy methods to distinguish chemical background signals from real metabolite signals."
- [readme] msimunging.pl for large CSV files: [evidence span withheld — anchor status: external_doc]
- [methods] Cardinal objects can be converted to mass2adduct's msimat: "data objects in the `MSProcessedImagingExperiment` or `MSContinuousImagingExperiment` formats can be converted to mass2adduct's `msimat` format with the `cardinal2msimat()` function"
- [abstract] co-localization of adducts with their parent ions: "We took advantage of the co-localization of adducts with their parent ions and the accuracy of high mass resolution to estimate adduct abundance in 20 datasets from different vendors of mass"
