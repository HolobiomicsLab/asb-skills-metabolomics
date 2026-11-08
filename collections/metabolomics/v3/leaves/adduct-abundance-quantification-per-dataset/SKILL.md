---
name: adduct-abundance-quantification-per-dataset
description: Use when you have a peak table exported from MALDI-MSI software (SCiLS/MSiReader
  CSV or Cardinal peakBinned imzML) and want to estimate what fraction of peaks are
  adducts rather than true metabolite signals.
when_to_use_negative:
- Input is raw full-spectrum imaging data that has not been peak-picked or binned
  — mass2adduct works on peak lists, so Cardinal data must already be processed with
  peakBin() before conversion.
- The goal is structural identification of a metabolite rather than abundance of adduct
  peaks; this skill flags adduct-vs-parent relationships by mass and co-localization,
  it does not provide database identifications.
- Only a single mass spectrum with no spatial information is available — the corrPairsMSI
  co-localization step requires per-pixel intensity data from an msimat object.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mass2adduct
  role: R package providing msimat(), massdiff(), adductMatch(), diffGetPeaks(), and
    corrPairsMSI() to count and identify putative metabolite–matrix adducts and estimate
    their abundance in MALDI-MSI peak data
  repo: https://github.com/kbseah/mass2adduct
- name: Cardinal
  role: 'Optional R front-end: peakBin()-processed MSProcessedImagingExperiment/MSContinuousImagingExperiment
    objects are converted to msimat format with cardinal2msimat()'
  repo: http://cardinalmsi.org/
- name: SCiLS
  role: Source of plain-text CSV peak-intensity exports consumed by msimat()
- name: MSiReader
  role: Alternative source of CSV 'intensity export' peak tables consumed by msimat()
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:648a34a0571bc7e3fd9294d59b7454cfc1f70ba4ff104f5386448ffdc2bdea73
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# adduct-abundance-quantification-per-dataset

## Summary

Quantifies the fraction of MS peaks in a MALDI-MSI dataset attributable to metabolite–matrix adducts (Na+, K+, 13C, [M + (DHB-H2O) + H]+, [M + CHCA + Na]+) by combining pairwise mass-difference calculation, adduct-mass matching, and spatial co-localization of parent/adduct ion pairs in mass2adduct.

## When to use

Use when you have a peak table exported from MALDI-MSI software (SCiLS/MSiReader CSV or Cardinal peakBinned imzML) and want to estimate what fraction of peaks are adducts rather than true metabolite signals.

## When NOT to use

- Input is raw full-spectrum imaging data that has not been peak-picked or binned — mass2adduct works on peak lists, so Cardinal data must already be processed with peakBin() before conversion.
- The goal is structural identification of a metabolite rather than abundance of adduct peaks; this skill flags adduct-vs-parent relationships by mass and co-localization, it does not provide database identifications.
- Only a single mass spectrum with no spatial information is available — the corrPairsMSI co-localization step requires per-pixel intensity data from an msimat object.

## Inputs

- Peak table with per-pixel intensities exported as plain-text CSV from SCiLS or MSiReader (very large files can be pre-converted to triplet format with the msimunging.pl script)
- Cardinal MSProcessedImagingExperiment or MSContinuousImagingExperiment object (Cardinal >= v2.2) with peaks already binned via peakBin(), convertible via cardinal2msimat()
- Plain-text file of mass values (one per line) if only mass-difference analysis is needed
- Optional custom adduct reference data.frame with columns name, formula, mass (built-in adducts and adducts2 tables otherwise)

## Outputs

- massdiff object listing parent–derivative mass pairs and their differences
- Mass-difference histogram object with counts per bin
- adductMatch table reporting counts and quantiles for each known adduct (Na+, K+, 13C, DHB and CHCA matrix adducts)
- corrPairsMSI data.frame of correlation p-values for each candidate parent–adduct ion pair (significance cutoff p=0.05 with Bonferroni correction)
- Per-dataset adduct fractions (significant matched peaks per adduct class divided by total peak counts) suitable for cross-platform comparison

## How to apply

Export the peak list and per-pixel intensities from SCiLS or MSiReader as plain-text CSV (or convert a Cardinal MSProcessedImagingExperiment/MSContinuousImagingExperiment object with cardinal2msimat) and load it with msimat() into the mass2adduct R package. Tabulate all pairwise mass differences with massdiff(), bin them into a histogram (bin width chosen to match instrument mass precision, e.g. 0.01 in the worked example), and match histogram bins to the built-in adducts/adducts2 reference tables with adductMatch(); use topAdducts() to rank unannotated high-count differences. For candidate adducts such as DHB-H2O (mass 136.01600), retrieve parent/adduct peak pairs with diffGetPeaks() within a ±0.5 mDa (1 mDa full width) or ppm-proportional window, then test spatial co-localization of each pair with corrPairsMSI(), which applies a two-tailed Pearson correlation with a Bonferroni-corrected p cutoff of 0.05 — switch to corrPairsMSIchunks() with mem.limit for datasets too large for memory. Count significant matched peaks per adduct class and divide by the total number of peaks to obtain per-dataset adduct fractions, the quantity reported to differ between atmospheric-pressure AP-SMALDI/Orbitrap datasets (~31% of peaks adduct-driven with DHB matrix) and high-vacuum MALDI-MSI platforms (~6%).

## Related tools

- **mass2adduct** (R package providing msimat(), massdiff(), adductMatch(), diffGetPeaks(), and corrPairsMSI() to count and identify putative metabolite–matrix adducts and estimate their abundance in MALDI-MSI peak data) — https://github.com/kbseah/mass2adduct
- **Cardinal** (Optional R front-end: peakBin()-processed MSProcessedImagingExperiment/MSContinuousImagingExperiment objects are converted to msimat format with cardinal2msimat()) — http://cardinalmsi.org/
- **SCiLS** (Source of plain-text CSV peak-intensity exports consumed by msimat())
- **MSiReader** (Alternative source of CSV 'intensity export' peak tables consumed by msimat())

## Examples

```
d.diff.DHBH2O <- diffGetPeaks(d.diff, mass=136.01600, mDa=1); d.diff.DHBH2O.corr <- corrPairsMSI(d, d.diff.DHBH2O)
```

## Evaluation signals

- The adductMatch output should return counts and quantiles for every entry in the adducts reference table; quantiles are usually high because most mass differences have zero to few counts.
- For the DHB-H2O probe (mass 136.01600), diffGetPeaks() with mDa=1 should return a subset massdiff containing both parent and derivative peaks within ±0.5 mDa.
- corrPairsMSI output must contain a p-value for every tested ion pair and the significant subset should be defined by the Bonferroni-adjusted cutoff of p=0.05.
- Per-dataset adduct fractions should reproduce the published platform pattern: DHB-matrix AP-SMALDI/Orbitrap datasets near ~31% of peaks, high-vacuum MALDI-MSI platforms near ~6%.
- Histogram bin width must match instrument mass precision (0.01 in the worked example); check that 'zoomed' plots (e.g. xlim=c(100,150)) resolve expected adduct bins rather than smearing counts across bins.

## Limitations

- Calculated mass differences are misleadingly precise — they must be binned into a histogram with a bin width reflecting the instrument's known mass precision before matching.
- corrPairsMSI holds the full pairwise output in memory; for large datasets corrPairsMSIchunks() with mem.limit is required, and even msimunging.pl conversion is limited by available RAM (e.g. a 40 Gb CSV on a 32 Gb RAM machine will probably not work).
- Mass-difference matching plus co-localization only yields putative adduct assignments; the paper presents them as candidate ion relationships, not confirmed structural identifications.
- Reading multi-Gb SCiLS/MSiReader CSV exports directly into R may be infeasible; the majority of matrix entries are zeroes and the triplet-format conversion script must be used first.

## Evidence

- [methods] This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object.: "This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object."
- [readme] By default the cutoff for significance is p=0.05 with Bonferroni correction.: [evidence span withheld — anchor status: external_doc]
- [methods] The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above.: "The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above."
- [abstract] We took advantage of the co-localization of adducts with their parent ions and the accuracy of high mass resolution to estimate adduct abundance: "We took advantage of the co-localization of adducts with their parent ions and the accuracy of high mass resolution to estimate
adduct abundance"
- [readme] they should be binned into a histogram with a user-specified bin width, that depends on the known mass precision of your instrument. In this example, we use the value of 0.01: [evidence span withheld — anchor status: not_found]
- [readme] Find parent and derivative mass peaks associated with a specific molecular adduct, e.g. DHB-H2O (mass 136.01600), within a window of 1 mDa, i.e. +/- 0.5 mDa.: [evidence span withheld — anchor status: external_doc]
