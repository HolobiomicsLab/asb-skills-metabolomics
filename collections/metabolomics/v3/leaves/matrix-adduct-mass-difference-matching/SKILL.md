---
name: matrix-adduct-mass-difference-matching
description: Use when a MALDI-MSI peak list contains unexplained ions whose mass differences
  from annotated metabolite peaks match known matrix or salt adducts, and you need
  putative adduct assignment supported by ion-map correlation.
when_to_use_negative:
- Input is raw unprocessed imzML without peak picking/binning — mass2adduct requires
  a peak list (e.g. Cardinal peakBin output), not raw spectra
- The goal is MS2 fragment-based adduct confirmation rather than MS1 mass-difference
  screening — this skill covers mass-difference matching and co-localization, not
  tandem-MS spectral comparison
- Adduct candidates fall outside the built-in/custom reference table and occur at
  low counts — adductMatch only finds known adducts; use topAdducts() exploratory
  ranking instead, expecting less confident assignment
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: mass2adduct
  role: R package providing msimat import, massdiff pairwise mass differences, adductMatch
    reference matching, diffGetPeaks, and corrPairsMSI spatial correlation
  repo: https://github.com/kbseah/mass2adduct
- name: R
  role: Execution environment; package installed with install_github('kbseah/mass2adduct')
    and loaded with library(mass2adduct)
- name: imzML
  role: Emerging-standard XML-based MSI input format readable via Cardinal before
    conversion to msimat
- name: Cardinal
  role: Used to read and peak-bin imzML data (normalize, smoothSignal, reduceBaseline,
    peakBin) before cardinal2msimat() conversion
- name: SCiLS
  role: Source of plain-text CSV peak-intensity exports consumable by msimat()
- name: MSiReader
  role: Source of 'intensity export' CSV files consumable by msimat()
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:a9c0a608c4a4b8d97bde2501d425eb94c6678e1744a61cdf048cd88e816c39f9
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# matrix-adduct-mass-difference-matching

## Summary

Identifies metabolite–matrix adducts (e.g. [M + (DHB-H2O) + H]+, [M + DHB + Na]+) in MALDI-MSI data by matching all pairwise peak mass differences against a reference adduct table and confirming candidates via spatial co-localization of adduct and parent ions.

## When to use

Use when a MALDI-MSI peak list contains unexplained ions whose mass differences from annotated metabolite peaks match known matrix or salt adducts, and you need putative adduct assignment supported by ion-map correlation.

## When NOT to use

- Input is raw unprocessed imzML without peak picking/binning — mass2adduct requires a peak list (e.g. Cardinal peakBin output), not raw spectra
- The goal is MS2 fragment-based adduct confirmation rather than MS1 mass-difference screening — this skill covers mass-difference matching and co-localization, not tandem-MS spectral comparison
- Adduct candidates fall outside the built-in/custom reference table and occur at low counts — adductMatch only finds known adducts; use topAdducts() exploratory ranking instead, expecting less confident assignment

## Inputs

- MALDI-MSI peak-intensity matrix as plain-text CSV exported from SCiLS or MSiReader
- Cardinal MSProcessedImagingExperiment or MSContinuousImagingExperiment (peak-binned with peakBin())
- Plain-text list of mass values (one per line) if pixel intensities are not needed
- Reference adduct table (built-in 'adducts' data set, or custom data.frame with columns name, formula, mass)

## Outputs

- massdiff object of parent/derivative mass pairs
- Histogram of binned mass-difference counts
- adductMatch table of known-adduct matches with counts and quantiles
- corrPairsMSI data.frame of Pearson correlation p-values for parent-adduct ion pairs
- Ion maps / annotated spectra visualizing parent and adduct peaks

## How to apply

Import the MSI peak-intensity matrix (CSV exported from SCiLS or MSiReader, or a peak-binned Cardinal MSProcessedImagingExperiment converted via cardinal2msimat) into mass2adduct's msimat format. Compute all pairwise mass differences with massdiff(), bin them into a histogram (bin width matched to instrument mass precision, e.g. 0.01 in the vignette), then call adductMatch() against the built-in adducts reference table to count matches to known adducts such as DHB-H2O (mass 136.01600). Extract the parent/adduct peak pairs for a given adduct with diffGetPeaks() using a tolerance window (e.g. mDa=1, i.e. +/- 0.5 mDa, or ppm-based), and test spatial co-localization of each ion pair with corrPairsMSI() (two-tailed Pearson by default, significance cutoff p=0.05 with Bonferroni correction). The rationale is that a true adduct shares its parent's spatial distribution, so high correlation plus an exact mass-difference match provides putative assignment; topAdducts() is the complementary function for high-count differences with no reference match.

## Related tools

- **mass2adduct** (R package providing msimat import, massdiff pairwise mass differences, adductMatch reference matching, diffGetPeaks, and corrPairsMSI spatial correlation) — https://github.com/kbseah/mass2adduct
- **R** (Execution environment; package installed with install_github('kbseah/mass2adduct') and loaded with library(mass2adduct))
- **imzML** (Emerging-standard XML-based MSI input format readable via Cardinal before conversion to msimat)
- **Cardinal** (Used to read and peak-bin imzML data (normalize, smoothSignal, reduceBaseline, peakBin) before cardinal2msimat() conversion)
- **SCiLS** (Source of plain-text CSV peak-intensity exports consumable by msimat())
- **MSiReader** (Source of 'intensity export' CSV files consumable by msimat())

## Examples

```
library(mass2adduct); d <- msimat('msi.csv', sep=';'); d.diff <- massdiff(d); d.diff.hist <- hist(d.diff); adductMatch(d.diff.hist)
```

## Evaluation signals

- The mass difference between adduct and parent peaks matches the reference adduct mass (e.g. DHB-H2O at 136.01600) within the tolerance window (default mDa=1, i.e. +/- 0.5 mDa)
- corrPairsMSI reports significant spatial correlation for the ion pair at p < 0.05 after Bonferroni correction; validated example reached Pearson r ≈ 0.9 for the PC(36:1) [M + (DHB-H2O) + H]+ pair
- adductMatch returns the expected adduct among top hits with sensible counts and quantiles given the mass-difference histogram bin width (e.g. 0.01)
- Adduct ion map visually co-localizes with the parent ion map (e.g. m/z 924.632 vs 788.617 in mouse brain DHB dataset)
- For Cardinal-derived inputs, plot(d_peaks) and plot(d_msimat) peak histograms should look the same after conversion

## Limitations

- Metabolite–matrix adducts can influence up to one third of total peak counts, so unmatched peaks may still be adducts not present in the reference table
- Calculated mass differences are 'misleadingly precise' because measurement error is not accounted for — binning width must be chosen according to instrument mass precision
- Large CSV exports (several Gb) may exceed available memory; use the msimunging.pl script to convert to triplet format and/or corrPairsMSIchunks with a mem.limit setting
- Cardinal conversion requires Cardinal v2.2+ and data pre-processed with peakBin(); adduct abundance estimates rely on co-localization, which is a putative, not definitive, assignment criterion

## Evidence

- [abstract] We took advantage of the co-localization of adducts with their parent ions and the accuracy of high mass resolution to estimate adduct abundance: "We took advantage of the co-localization of adducts with their parent ions and the accuracy of high mass resolution to estimate adduct abundance"
- [readme] massdiff and adductMatch workflow: "Take all possible pairs of masses and calculate the mass difference for each pair. These mass differences represent potential molecular adducts."
- [readme] adductMatch function: "The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above."
- [readme] corrPairsMSI correlation test: "This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object."
- [readme] binning rationale: [evidence span withheld — anchor status: external_doc]
- [abstract] impact on peak counts: "reveals a major influence of so far unrecognized metabolite–matrix adducts on total peak counts (up to one third)"
