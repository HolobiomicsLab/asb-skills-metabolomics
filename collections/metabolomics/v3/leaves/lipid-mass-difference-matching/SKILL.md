---
name: lipid-mass-difference-matching
description: Use when you have a peak list (plain-text CSV from SCiLS/MSiReader, imzML
  processed with Cardinal, or a simple mass list) from MALDI-MSI data and need to
  identify which peaks are metabolite–matrix or salt adducts rather than genuine metabolite
  ions.
when_to_use_negative:
- Input is already an annotated feature table or identification list where adduct
  assignment has already been performed — this skill is for raw peak-pair mass-difference
  screening, not post-hoc annotation correction.
- Data are not from MS imaging (i.e. no spatial pixel dimension) — `corrPairsMSI()`
  requires `msimat` MSI data with pixel coordinates; for non-MSI spectra only the
  mass-difference histogram part applies.
- The instrument's mass precision is unknown or much lower than the bin width used
  — misleadingly precise mass differences will not bin correctly and `adductMatch()`
  will fail to find close-matching bins.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mass2adduct
  role: Primary R package implementing mass-difference tabulation, adduct matching,
    and MSI co-localization testing for this skill.
  repo: github.com/kbseah/mass2adduct
- name: R
  role: Runtime environment in which mass2adduct is loaded and invoked (`library(mass2adduct)`).
- name: imzML
  role: Input format for MSI data that can be read and peak-binned in Cardinal before
    conversion to `msimat`.
- name: Cardinal
  role: Used to pre-process (normalize, smooth, reduce baseline, peak-bin) imzML data
    before conversion to mass2adduct's `msimat` format with `cardinal2msimat()`.
  repo: http://cardinalmsi.org/
- name: SCiLS
  role: Source of plain-text CSV exports of MSI intensity data used as direct input
    to `msimat()`.
- name: MSiReader
  role: Source of plain-text CSV exports of MSI intensity data used as direct input
    to `msimat()`.
- name: METASPACE
  role: Annotation platform whose results can be cross-checked against detected adducts
    to avoid false-positive annotations caused by matrix adducts.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:8e97410e8611570f3bcea23f191ddd308f832ce784ac234ab58fc204f61f97d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - METASPACE
  - MSiReader
  - R
  - SCiLS
  - imzML
  - mass2adduct
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# lipid-mass-difference-matching

## Summary

Match metabolite/lipid peak pairs in MALDI-MSI peak lists to known adduct mass differences (e.g. DHB-H2O at 136.01600) using the mass2adduct R package, then confirm putative parent–adduct pairs by spatial co-localization correlation, illuminating the 'dark metabolome' that otherwise confounds annotation.

## When to use

Use when you have a peak list (plain-text CSV from SCiLS/MSiReader, imzML processed with Cardinal, or a simple mass list) from MALDI-MSI data and need to identify which peaks are metabolite–matrix or salt adducts rather than genuine metabolite ions.

## When NOT to use

- Input is already an annotated feature table or identification list where adduct assignment has already been performed — this skill is for raw peak-pair mass-difference screening, not post-hoc annotation correction.
- Data are not from MS imaging (i.e. no spatial pixel dimension) — `corrPairsMSI()` requires `msimat` MSI data with pixel coordinates; for non-MSI spectra only the mass-difference histogram part applies.
- The instrument's mass precision is unknown or much lower than the bin width used — misleadingly precise mass differences will not bin correctly and `adductMatch()` will fail to find close-matching bins.

## Inputs

- Plain-text comma-delimited CSV of MSI intensities exported from SCiLS or MSiReader (imported with `msimat()`)
- Cardinal v2.2+ `MSProcessedImagingExperiment` or `MSContinuousImagingExperiment` objects that have been peak-binned (converted via `cardinal2msimat()`)
- Plain-text file of mass values, one per line (imported with `scan()`)
- Built-in `adducts`/`adducts2` reference data frames (name, formula, mass) or a user-supplied custom data frame in the same three-column format

## Outputs

- `msimat` object summarising total intensities per peak
- `massdiff` object containing all pairwise mass differences (two parent masses and their difference)
- Histogram object of binned mass differences
- Ranked table of adduct matches with counts and quantiles from `adductMatch()`
- Ranked table of top unannotated mass differences with any known-adduct matches from `topAdducts()`
- Subset `massdiff` object of parent–adduct peak pairs from `diffGetPeaks()`
- Data frame of p-values for spatial co-localization of each putative parent–adduct pair from `corrPairsMSI()`

## How to apply

Import the MSI peak data into R as a mass2adduct `msimat` object (e.g. via `msimat()` for CSV files from SCiLS/MSiReader or `cardinal2msimat()` for peak-binned Cardinal objects, or `scan()` for a plain mass list). Compute all pairwise mass differences with `massdiff(peaks)` and bin them into a histogram with `hist(d.diff)` using a bin width (e.g. 0.01) matched to the instrument's known mass precision. Match the histogram bins against the built-in `adducts`/`adducts2` reference data frames using `adductMatch(d.diff.hist)` to count how many peak pairs correspond to known adducts, and complement this with `topAdducts()` to surface unannotated high-count mass differences. Extract specific parent–derivative pairs with `diffGetPeaks(d.diff, mass=136.01600, mDa=1)` (window in mDa or ppm) and confirm their adduct relationship by testing spatial co-localization with `corrPairsMSI(d, d.diff.DHBH2O)`, applying the default significance cutoff p=0.05 with Bonferroni correction — the rationale is that true adduct pairs should be spatially correlated with their parent ions.

## Related tools

- **mass2adduct** (Primary R package implementing mass-difference tabulation, adduct matching, and MSI co-localization testing for this skill.) — github.com/kbseah/mass2adduct
- **R** (Runtime environment in which mass2adduct is loaded and invoked (`library(mass2adduct)`).)
- **imzML** (Input format for MSI data that can be read and peak-binned in Cardinal before conversion to `msimat`.)
- **Cardinal** (Used to pre-process (normalize, smooth, reduce baseline, peak-bin) imzML data before conversion to mass2adduct's `msimat` format with `cardinal2msimat()`.) — http://cardinalmsi.org/
- **SCiLS** (Source of plain-text CSV exports of MSI intensity data used as direct input to `msimat()`.)
- **MSiReader** (Source of plain-text CSV exports of MSI intensity data used as direct input to `msimat()`.)
- **METASPACE** (Annotation platform whose results can be cross-checked against detected adducts to avoid false-positive annotations caused by matrix adducts.)

## Examples

```
library(mass2adduct); d <- msimat("msi.csv", sep=";"); d.diff <- massdiff(d); d.diff.hist <- hist(d.diff); head(adductMatch(d.diff.hist)); d.diff.DHBH2O <- diffGetPeaks(d.diff, mass=136.01600, mDa=1); corrPairsMSI(d, d.diff.DHBH2O)
```

## Evaluation signals

- The mass-difference histogram must show a peak at 136.01600 (DHB-H2O) when DHB matrix data are analysed; `adductMatch()` should report counts for this bin.
- `diffGetPeaks()` with mass=136.01600 and a ±1 mDa window must return a non-empty subset of the original `massdiff` object containing parent and derivative peaks.
- `corrPairsMSI()` output must be a data frame with a p-value for every ion pair, and the default significance cutoff (p=0.05, Bonferroni-corrected) should mark true adduct pairs as significant.
- Total peak counts attributable to metabolite–matrix adducts should be substantial — the paper reports up to one third of total peak counts in MALDI-MSI datasets.
- Imported `msimat` object must reproduce the same peak histogram as the original Cardinal object when converted via `cardinal2msimat()` (the README notes 'should look the same').

## Limitations

- Mass differences from `massdiff()` are 'misleadingly precise' because measurement error and uncertainty are not accounted for — they must be binned into a histogram with a user-specified bin width appropriate to the instrument.
- For very large data sets, `corrPairsMSI()` may exceed memory; use `corrPairsMSIchunks()` with an explicit `mem.limit=N` or pre-filter peaks with the Perl script `msimunging.pl`.
- The `adducts`/`adducts2` built-in reference lists are small ('a small data set of common molecular adducts'); custom adduct lists must be supplied as a data.frame with columns name, formula, mass.
- The method performs a simple mass difference calculation and spatial correlation analysis — it provides putative assignments only, not definitive structural identifications.
- Quantile values reported by `adductMatch()` are usually high because the majority of mass differences have zero to few counts, so counts should be interpreted relative to this background.

## Evidence

- [readme] Mass differences represent potential molecular adducts; bin width must reflect instrument precision: "Take all possible pairs of masses and calculate the mass difference for each pair. These mass differences represent potential molecular adducts."
- [readme] Bin width choice is critical: [evidence span withheld — anchor status: external_doc]
- [readme] adductMatch matches known adducts via histogram bins: "The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above."
- [readme] Specific adduct extraction with mDa window: [evidence span withheld — anchor status: external_doc]
- [readme] Spatial co-localization rationale for adduct pairs: [evidence span withheld — anchor status: external_doc]
- [readme] Default significance cutoff for correlation test: [evidence span withheld — anchor status: external_doc]
- [abstract] Co-localization with parent ions is the basis of adduct estimation: "We took advantage of the co-localization of adducts with their parent ions and the accuracy of high mass resolution to estimate
adduct abundance"
- [abstract] Adducts influence up to one third of peak counts: "reveals a major influence of so far unrecognized metabolite–matrix
adducts on total peak counts (up to one third)"
- [discussion] Including matrix adducts prevents false annotations: "including metabolite–matrix adducts into database annotations can reduce the number of unannotated peaks and, on the other hand, will prevent possible false annotations"
- [readme] Built-in adduct reference lists are small: "The package comes with a small data set of common molecular adducts, called `adducts` (use the command `help(adducts)` to see a description)."
- [readme] Cardinal pre-processing required before conversion: [evidence span withheld — anchor status: external_doc]
