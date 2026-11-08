---
name: pairwise-mass-difference-histogram-analysis
description: Use when you have peak-mass lists from MALDI-MSI datasets (e.g. SCiLS/MSiReader
  CSV exports or peakBinned Cardinal imzML) and need to find which peaks are matrix-
  or salt-adduct partners of parent ions.
when_to_use_negative:
- Peaks have not been peak-binned/processed — mass2adduct requires Cardinal data to
  be preprocessed with peakBin() before cardinal2msimat()
- The CSV is too large for available RAM and has not been converted to triplet format
  with the msimunging.pl script (e.g. 40 Gb file on a 32 Gb system)
- You only have raw spectra without peak lists — the skill operates on discrete peak
  masses, not raw profiles
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mass2adduct
  role: R package providing msimat(), massdiff(), hist/adductMatch(), topAdducts(),
    and diffGetPeaks() for the pairwise mass-difference histogram workflow
  repo: https://github.com/kbseah/mass2adduct
- name: Cardinal
  role: Used to read imzML and preprocess/peakBin() MSI data before converting to
    msimat with cardinal2msimat()
  repo: http://cardinalmsi.org/
- name: SCiLS
  role: Source of plain-text CSV intensity exports consumed by msimat()
- name: MSiReader
  role: Alternative source of plain-text CSV intensity exports consumed by msimat()
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:61d475ec6fa723d9e7e04c11664bf42a025ac8abef1ad1ab971bd29c8feb5fd1
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# pairwise-mass-difference-histogram-analysis

## Summary

Annotate putative molecular adducts in MS/MSI data by computing all pairwise peak mass differences, binning them into a histogram, and matching bins against known adduct masses (e.g. Na+, K+, 13C, DHB-H2O).

## When to use

Use when you have peak-mass lists from MALDI-MSI datasets (e.g. SCiLS/MSiReader CSV exports or peakBinned Cardinal imzML) and need to find which peaks are matrix- or salt-adduct partners of parent ions.

## When NOT to use

- Peaks have not been peak-binned/processed — mass2adduct requires Cardinal data to be preprocessed with peakBin() before cardinal2msimat()
- The CSV is too large for available RAM and has not been converted to triplet format with the msimunging.pl script (e.g. 40 Gb file on a 32 Gb system)
- You only have raw spectra without peak lists — the skill operates on discrete peak masses, not raw profiles

## Inputs

- msimat object from SCiLS/MSiReader CSV intensity export (plain-text comma-delimited)
- Cardinal MSProcessedImagingExperiment or MSContinuousImagingExperiment (imzML, preprocessed and peakBinned, Cardinal >= 2.2)
- Plain-text file of mass values (one per line) when pixel intensities are not needed

## Outputs

- massdiff object (parent mass 1, parent mass 2, mass difference) for every peak pair
- R histogram object of binned mass differences with per-bin counts
- adductMatch data.frame reporting counts and quantiles for known adducts (e.g. Na+, K+, 13C, [M + (DHB-H2O) + H]+, [M + DHB + Na]+)
- topAdducts ranked list of highest-count mass differences with matches to known adducts
- diffGetPeaks subset of massdiff containing parent/derivative peak pairs for a chosen adduct

## How to apply

Import peak intensities with msimat() (from SCiLS/MSiReader CSV) or cardinal2msimat() (from Cardinal MSProcessedImagingExperiment/MSContinuousImagingExperiment after peakBin()). Compute all pairwise mass differences with massdiff() and bin them into a histogram whose bin width (e.g. 0.01) matches your instrument's known mass precision. Match known adducts from the built-in adducts/adducts2 data sets using adductMatch() on the histogram, and use topAdducts() to rank unannotated high-count bins. For a specific adduct (e.g. DHB-H2O at 136.01600) call diffGetPeaks() with a tolerance window of 1 mDa or in ppm to extract candidate parent–adduct peak pairs; spatial co-localization (corrPairsMSI, Pearson, p=0.05 with Bonferroni correction) then validates which pairs reflect real adduct formation.

## Related tools

- **mass2adduct** (R package providing msimat(), massdiff(), hist/adductMatch(), topAdducts(), and diffGetPeaks() for the pairwise mass-difference histogram workflow) — https://github.com/kbseah/mass2adduct
- **Cardinal** (Used to read imzML and preprocess/peakBin() MSI data before converting to msimat with cardinal2msimat()) — http://cardinalmsi.org/
- **SCiLS** (Source of plain-text CSV intensity exports consumed by msimat())
- **MSiReader** (Alternative source of plain-text CSV intensity exports consumed by msimat())

## Examples

```
d <- msimat("msi.csv", sep=";"); d.diff <- massdiff(d); d.diff.hist <- hist(d.diff); head(adductMatch(d.diff.hist))
```

## Evaluation signals

- adductMatch() returns counts and quantile values for each known adduct; quantiles are expected to be high because most mass differences have zero to few counts
- Histogram bin width must match instrument mass precision (e.g. 0.01 in the worked example) — verify chosen bins resolve known adduct masses like DHB-H2O at 136.01600
- diffGetPeaks() output should be a proper subset of the original massdiff object
- On large data sets, use corrPairsMSIchunks() with mem.limit=N instead of corrPairsMSI() to avoid exceeding memory
- Compare peak histograms from Cardinal vs mass2adduct after conversion — plot(d_peaks) and plot(d_msimat) should look the same

## Limitations

- Calculated mass differences are misleadingly precise — they must be binned into a histogram because measurement error and uncertainty are not otherwise accounted for
- Large CSV exports (several Gb, e.g. 40 Gb) may not fit into R memory; msimunging.pl conversion to triplet format is memory-bound and may fail
- adductMatch only finds known adducts; use topAdducts to explore unmatched high-count bins
- corrPairsMSI loads the whole result table in memory; on large data sets use corrPairsMSIchunks with mem.limit

## Evidence

- [methods] Take all possible pairs of masses and calculate the mass difference for each pair. These mass differences represent potential molecular adducts.: "Take all possible pairs of masses and calculate the mass difference for each pair. These mass differences represent potential molecular adducts."
- [readme] The calculated mass differences are misleadingly precise, because measurement error and uncertainty are not taken into account. They should be binned into a histogram: [evidence span withheld — anchor status: external_doc]
- [methods] This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object.: "This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object."
- [methods] data objects in the `MSProcessedImagingExperiment` or `MSContinuousImagingExperiment` formats can be converted to mass2adduct's `msimat` format: "data objects in the `MSProcessedImagingExperiment` or `MSContinuousImagingExperiment` formats can be converted to mass2adduct's `msimat` format"
