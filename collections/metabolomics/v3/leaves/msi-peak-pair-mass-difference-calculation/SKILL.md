---
name: msi-peak-pair-mass-difference-calculation
description: Use when you have binned MSI peak data (CSV from SCiLS/MSiReader or an
  msimat object) and need to enumerate candidate adduct ion pairs by mass difference
  before adduct annotation and spatial correlation.
when_to_use_negative:
- Data are raw, unbinned spectra — peaks must already be binned (e.g. with Cardinal's
  peakBin()) before pairwise differences are meaningful
- You need statistical evidence of adduct identity rather than candidate enumeration
  — massdiff alone gives mass differences; follow with corrPairsMSI() spatial correlation
- Memory-constrained sessions with multi-Gb CSV matrices — use the msimunging.pl triplet-format
  reformatting or corrPairsMSIchunks instead of loading the full matrix directly
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mass2adduct
  role: R package providing msimat(), massdiff(), hist()/adductMatch()/topAdducts(),
    and diffGetPeaks() for the pairwise mass-difference calculation and subsequent
    annotation
  repo: github.com/kbseah/mass2adduct
- name: Cardinal
  role: 'Optional upstream: readImzML() plus peakBin() pre-processing, converted to
    msimat with cardinal2msimat()'
- name: SCiLS
  role: Source of plain-text CSV peak-intensity exports that msimat() can import
- name: MSiReader
  role: Source of plain-text CSV peak-intensity exports (intensity export function)
    that msimat() can import
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:8f5d0b512f3e68133b00f463d7cf23f3b08c44077fa3cf0e611e932b1c58310a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - MSiReader
  - R
  - SCiLS
  - mass2adduct
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# msi-peak-pair-mass-difference-calculation

## Summary

Tabulate all pairwise mass differences between peaks in a MALDI-MSI peak table with mass2adduct's massdiff(), treating each difference as a candidate molecular adduct (e.g. metabolite–matrix adducts such as [M + (DHB-H2O) + H]+) for downstream annotation and correlation testing.

## When to use

Use when you have binned MSI peak data (CSV from SCiLS/MSiReader or an msimat object) and need to enumerate candidate adduct ion pairs by mass difference before adduct annotation and spatial correlation.

## When NOT to use

- Data are raw, unbinned spectra — peaks must already be binned (e.g. with Cardinal's peakBin()) before pairwise differences are meaningful
- You need statistical evidence of adduct identity rather than candidate enumeration — massdiff alone gives mass differences; follow with corrPairsMSI() spatial correlation
- Memory-constrained sessions with multi-Gb CSV matrices — use the msimunging.pl triplet-format reformatting or corrPairsMSIchunks instead of loading the full matrix directly

## Inputs

- Peak-binned MSI intensity table exported as plain-text CSV from SCiLS or MSiReader (semicolon- or comma-delimited, peaks as columns, pixels as rows)
- mass2adduct msimat object (imported via msimat() or cardinal2msimat() from peak-binned Cardinal experiments)
- Plain-text list of mass values (one per line) when pixel intensities are not needed

## Outputs

- massdiff object with three elements: the two parent masses and the difference between them
- Histogram of binned mass differences (standard R histogram object) for overview and adduct matching

## How to apply

Import the preprocessed, peak-binned MSI intensity table into mass2adduct's msimat format with msimat() (e.g. msimat('msi.csv', sep=';')), or convert peak-binned Cardinal MSProcessedImagingExperiment/MSContinuousImagingExperiment objects with cardinal2msimat(). Run massdiff(peaks) to take all possible pairs of masses and calculate the difference for each pair; the output is a massdiff object holding the two parent masses and their difference. Because the raw differences are misleadingly precise, bin them into a histogram (hist(d.diff)) with a user-specified bin width — the paper's example uses 0.01 — matched to your instrument's mass precision. The resulting massdiff object is then fed to adductMatch()/topAdducts() for annotation against the built-in adducts/adducts2 tables, and subsets for a specific adduct mass (e.g. DHB-H2O, 136.01600, within +/-0.5 mDa via diffGetPeaks) are the input to corrPairsMSI().

## Related tools

- **mass2adduct** (R package providing msimat(), massdiff(), hist()/adductMatch()/topAdducts(), and diffGetPeaks() for the pairwise mass-difference calculation and subsequent annotation) — github.com/kbseah/mass2adduct
- **Cardinal** (Optional upstream: readImzML() plus peakBin() pre-processing, converted to msimat with cardinal2msimat())
- **SCiLS** (Source of plain-text CSV peak-intensity exports that msimat() can import)
- **MSiReader** (Source of plain-text CSV peak-intensity exports (intensity export function) that msimat() can import)

## Examples

```
d <- msimat("msi.csv", sep=";"); d.diff <- massdiff(peaks); d.diff.hist <- hist(d.diff); plot(d.diff.hist, xlim=c(100,150))
```

## Evaluation signals

- Output object has class 'msimat' after import and class 'massdiff' after massdiff(), with three elements (two parent masses plus the difference)
- Number of pairwise differences matches all possible peak pairs; print(d) reports a summary consistent with the CSV peak count
- Histogram bin width (e.g. 0.01 in the worked example) is set to match the instrument's known mass precision, not the raw floating-point precision of the differences
- adductMatch() on the histogram returns counts for known adducts such as DHB-H2O (mass 136.01600); quantile values are expected to be high because most mass differences have zero to few counts
- diffGetPeaks(d.diff, mass=136.01600, mDa=1) returns a subset of the original massdiff object containing only parent/adduct pairs within the 1 mDa window

## Limitations

- Calculated mass differences are misleadingly precise because measurement error and uncertainty are not accounted for — they must be binned into a histogram with an instrument-appropriate bin width
- Pairwise differences on N peaks scale quadratically; very large MSI tables may not fit in memory and need the msimunging.pl triplet-format conversion or chunked processing
- msimunging.pl conversion is itself limited by available RAM (e.g. a 40 Gb CSV with 32 Gb RAM will probably not work)
- adductMatch() only finds adducts present in the reference table; use topAdducts() to catch unmatched high-count differences

## Evidence

- [readme] Take all possible pairs of masses and calculate the mass difference for each pair. These mass differences represent potential molecular adducts.: "Take all possible pairs of masses and calculate the mass difference for each pair. These mass differences represent potential molecular adducts."
- [readme] The calculated mass differences are misleadingly precise, because measurement error and uncertainty are not taken into account. They should be binned into a histogram: [evidence span withheld — anchor status: external_doc]
- [abstract] Metabolites ranging from lipids to amines and amino acids form matrix adducts with the commonly used 2,5-dihydroxybenzoic acid (DHB) matrix: "Metabolites ranging from lipids to amines and amino acids form matrix adducts with the commonly used 2,5-dihydroxybenzoic acid (DHB) matrix"
- [methods] This should be exported as plain-text CSV files from standard MSI software such as SCiLS or MSiReader.: "This should be exported as plain-text CSV files from standard MSI software such as SCiLS or MSiReader."
- [discussion] perform a simple mass difference calculation and spatial correlation analysis as a rapid and efficient way to screen for these adducts: "perform a simple mass difference calculation and spatial correlation analysis as a rapid and efficient way to screen for these adducts in existing MSI datasets"
