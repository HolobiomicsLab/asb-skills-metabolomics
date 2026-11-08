---
name: mass-difference-pairwise-computation-msi-peaks
description: Use when you have a preprocessed peak table exported as plain-text CSV
  from SCiLS or MSiReader (or an imzML/Cardinal peak-binned object) and want to screen
  for abundant adduct transformations among peaks in MALDI-mass spectrometry imaging
  data.
when_to_use_negative:
- If you only have raw continuous spectra without a preprocessed peak list — peaks
  must already be detected/binned (e.g. Cardinal peakBin()) before massdiff() can
  be applied
- If you need statistically validated adduct assignments rather than a screening histogram
  — use downstream spatial correlation (corrPairsMSI) instead of mass-difference counts
  alone
- If your CSV export is several Gb and cannot fit in memory — the README recommends
  reformatting with msimunging.pl to triplet format (and optionally filtering peaks
  by nonzero-pixel frequency) before importing
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3520
tools:
- name: mass2adduct
  role: Core R package providing msimat(), massdiff(), hist(), adductMatch(), topAdducts(),
    and cardinal2msimat() for pairwise mass-difference computation and histogram-based
    adduct screening
  repo: https://github.com/kbseah/mass2adduct
- name: SCiLS
  role: Source of the plain-text CSV peak/intensity exports consumed by msimat()
- name: MSiReader
  role: Alternative MSI software whose 'intensity export' CSV output is an accepted
    input format
- name: Cardinal
  role: R package whose peak-binned MSI objects are converted to mass2adduct's msimat
    format via cardinal2msimat()
  repo: http://cardinalmsi.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:9b6486ab3c3ae305c700be7880279deb4f5b00405d07d09545d2c821a71fafe8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbseah/mass2adduct
  tools:
  - Cardinal
  - MSiReader
  - R
  - SCiLS
  - mass2adduct
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# mass-difference-pairwise-computation-msi-peaks

## Summary

Compute all pairwise mass differences between detected MALDI-MSI peaks and bin them into a frequency histogram so that abundant metabolite–matrix adducts (e.g. Na+, K+, DHB-derived species) stand out above the integer-spaced background, as implemented in the R package mass2adduct.

## When to use

Use when you have a preprocessed peak table exported as plain-text CSV from SCiLS or MSiReader (or an imzML/Cardinal peak-binned object) and want to screen for abundant adduct transformations among peaks in MALDI-mass spectrometry imaging data.

## When NOT to use

- If you only have raw continuous spectra without a preprocessed peak list — peaks must already be detected/binned (e.g. Cardinal peakBin()) before massdiff() can be applied
- If you need statistically validated adduct assignments rather than a screening histogram — use downstream spatial correlation (corrPairsMSI) instead of mass-difference counts alone
- If your CSV export is several Gb and cannot fit in memory — the README recommends reformatting with msimunging.pl to triplet format (and optionally filtering peaks by nonzero-pixel frequency) before importing

## Inputs

- Plain-text comma-delimited CSV of MSI peak intensities exported from SCiLS or MSiReader (e.g. msi.csv with sep=";"), imported via msimat()
- Peak-binned Cardinal MSProcessedImagingExperiment or MSContinuousImagingExperiment object (Cardinal v2.2+), converted with cardinal2msimat()
- Plain-text list of mass values (one per line), e.g. scan("mass_example", what=numeric())
- Optional custom adduct reference data.frame with columns name, formula, mass

## Outputs

- massdiff object containing parent mass pairs and their mass differences
- Standard R histogram object (d.diff.hist) of binned mass-difference counts
- adductMatch report of known-adduct matches with counts and quantiles
- topAdducts ranked list of highest-count mass differences with matches to known adducts

## How to apply

Import the exported peak/intensity table into mass2adduct as an `msimat` object (or convert a peak-binned Cardinal `MSProcessedImagingExperiment`/`MSContinuousImagingExperiment` via `cardinal2msimat()`, or scan a plain list of masses). Run `massdiff(peaks)` to tabulate the mass difference between every possible pair of masses; the output is a `massdiff` object holding the two parent masses and their difference. Bin these differences into a histogram with `hist(d.diff)` using a bin width matched to your instrument's mass precision — the README example uses 0.01 Da — because raw pairwise differences are 'misleadingly precise'. Plot with `plot(d.diff.hist)`, optionally zooming via `xlim` (e.g. 100–150 Da for matrix-derived adducts), and annotate/interpret the prominent bins against the built-in `adducts` reference data frame (columns: name, formula, mass) with `adductMatch(d.diff.hist)` or rank all bins by count with `topAdducts(d.diff.hist, n=10)`. Abundant matrix adducts (e.g. [M + (DHB-H2O) + H]+, [M + DHB + Na]+) appear as high-count bins, so the rationale is that true adduct chemistry yields many shared mass-difference pairs while random matches fall in the near-integer background.

## Related tools

- **mass2adduct** (Core R package providing msimat(), massdiff(), hist(), adductMatch(), topAdducts(), and cardinal2msimat() for pairwise mass-difference computation and histogram-based adduct screening) — https://github.com/kbseah/mass2adduct
- **SCiLS** (Source of the plain-text CSV peak/intensity exports consumed by msimat())
- **MSiReader** (Alternative MSI software whose 'intensity export' CSV output is an accepted input format)
- **Cardinal** (R package whose peak-binned MSI objects are converted to mass2adduct's msimat format via cardinal2msimat()) — http://cardinalmsi.org/

## Examples

```
d <- msimat("msi.csv", sep=";"); d.diff <- massdiff(d); d.diff.hist <- hist(d.diff); plot(d.diff.hist); head(adductMatch(d.diff.hist))
```

## Evaluation signals

- The massdiff output object contains exactly three elements: the two parent masses and the difference between them
- The histogram bin width matches the instrument's known mass precision (README example: 0.01); binning too coarsely would merge distinct adduct bins
- Abundant adduct bins rise above the near-integer-spaced background — e.g. a prominent bin near 136.016 Da (DHB-H2O) or Na/K adduct differences
- adductMatch() returns counts and quantiles where quantiles are typically high because most mass differences have zero to few counts
- For Cardinal imports, the plotted peak histograms from the original d_peaks and the converted d_msimat should look the same

## Limitations

- Calculated mass differences are 'misleadingly precise' because measurement error and uncertainty are not taken into account — they must be binned into a histogram
- adductMatch only finds known adducts from the reference list; novel adduct bins are missed unless topAdducts() is used
- Note that quantile values will usually be quite high because the majority of mass differences have zero to few counts
- Large CSV exports (several Gb) may exceed available memory and require preprocessing via the msimunging.pl script to triplet format
- Cardinal conversion requires pre-processed data with peaks already binned using peakBin()

## Evidence

- [readme] Take all possible pairs of masses and calculate the mass difference for each pair. These mass differences represent potential molecular adducts.: "Take all possible pairs of masses and calculate the mass difference for each pair. These mass differences represent potential molecular adducts."
- [readme] The calculated mass differences are misleadingly precise, because measurement error and uncertainty are not taken into account. They should be binned into a histogram with a user-specified bin width: [evidence span withheld — anchor status: external_doc]
- [abstract] Metabolites ranging from lipids to amines and amino acids form matrix adducts with the commonly used 2,5-dihydroxybenzoic acid (DHB) matrix: "Metabolites ranging from lipids to amines and amino acids form matrix adducts with the commonly used 2,5-dihydroxybenzoic acid (DHB) matrix"
- [discussion] We developed a software pipeline mass2adduct to perform a simple mass difference calculation and spatial correlation analysis: "We developed a software pipeline mass2adduct to perform a simple mass difference calculation and spatial correlation analysis"
- [readme] The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above.: "The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above."
- [methods] data objects in the `MSProcessedImagingExperiment` or `MSContinuousImagingExperiment` formats can be converted to mass2adduct's `msimat` format: "data objects in the `MSProcessedImagingExperiment` or `MSContinuousImagingExperiment` formats can be converted to mass2adduct's `msimat` format"
