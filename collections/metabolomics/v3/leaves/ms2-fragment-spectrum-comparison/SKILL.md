---
name: ms2-fragment-spectrum-comparison
description: Use when a m/z peak in a MALDI-MSI dataset (e.g., mouse brain imzML data
  acquired with DHB matrix) is suspected to be a matrix adduct of a known lipid, and
  a deposited on-tissue MS2 spectrum is available to verify the fragment-ion series
  against the parent ion.
when_to_use_negative:
- No tandem MS2 spectrum is available for the candidate adduct peak — in that case
  use mass-difference matching (adductMatch) and ion-map correlation (corrPairsMSI)
  alone, as in the paper's primary screen
- Input is a feature/peak table without per-pixel intensities (e.g., a simple numeric
  vector of masses), so ion-map spatial correlation cannot be computed
- The mass difference between the two peaks does not match any entry in the built-in
  adducts reference (e.g., not a DHB matrix or salt-ion adduct), making the parent–adduct
  hypothesis unsupported
tools:
- name: mass2adduct
  role: R package providing adductMatch (match mass differences to known adducts like
    DHB-H2O at 136.01600), corrPairsMSI (two-tailed Pearson correlation of parent–adduct
    ion maps with Bonferroni-corrected p=0.05), and msimat/cardinal2msimat for importing
    the MSI intensity matrix
  repo: https://github.com/kbseah/mass2adduct
- name: R
  role: Execution environment for mass2adduct; the adduct-confirmation workflow (msimat
    import, adductMatch, corrPairsMSI, CSV export) is run as R code with library(mass2adduct)
- name: imzML
  role: Raw MALDI-MSI data format that is read into R (e.g., via Cardinal's readImzML,
    normalized, smoothed, baseline-reduced and peak-binned) before conversion to msimat
    for correlation analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:ea0dcd76265c1f89dd3b5446dceed8bb9643ea9ef93decfe8875c91348174083
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbseah/mass2adduct
  tools:
  - R
  - imzML
  - mass2adduct
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# ms2-fragment-spectrum-comparison

## Summary

Confirms a candidate matrix adduct in MALDI-MSI by comparing the on-tissue MS2 fragment spectrum of the adduct peak with that of its putative parent ion, then cross-validating with spatial correlation of their ion maps. In this workflow, an ion like m/z 924.632 is assigned as [PC(36:1) + (DHB-H2O) + H]+ only if its fragment series matches [PC(36:1)+H]+ and their pixel intensities co-localize.

## When to use

Use when a m/z peak in a MALDI-MSI dataset (e.g., mouse brain imzML data acquired with DHB matrix) is suspected to be a matrix adduct of a known lipid, and a deposited on-tissue MS2 spectrum is available to verify the fragment-ion series against the parent ion.

## When NOT to use

- No tandem MS2 spectrum is available for the candidate adduct peak — in that case use mass-difference matching (adductMatch) and ion-map correlation (corrPairsMSI) alone, as in the paper's primary screen
- Input is a feature/peak table without per-pixel intensities (e.g., a simple numeric vector of masses), so ion-map spatial correlation cannot be computed
- The mass difference between the two peaks does not match any entry in the built-in adducts reference (e.g., not a DHB matrix or salt-ion adduct), making the parent–adduct hypothesis unsupported

## Inputs

- MSI peak-intensity matrix exported as plain-text CSV from SCiLS or MSiReader, or an imzML file peak-binned via Cardinal (MSProcessedImagingExperiment/MSContinuousImagingExperiment) and converted with cardinal2msimat()
- Candidate adduct ion m/z and parent ion m/z pair (e.g., m/z 924.632 and m/z 788.617)
- Deposited on-tissue MS2 spectra for both the candidate adduct peak and the parent ion
- Built-in mass2adduct 'adducts' reference table (name, formula, mass columns) or a custom adducts data.frame

## Outputs

- Fragment-spectrum comparison summary (matched m/z values and relative intensities of the adduct vs parent MS2 spectra, including the neutral-loss fragment)
- corrPairsMSI output data.frame with Pearson correlation coefficient and p-values (Bonferroni-corrected, cutoff p=0.05) for the ion pair
- Verification table, e.g., adduct_ms2_correlation_verification.csv

## How to apply

Load the MSI peak-intensity matrix (e.g., exported from SCiLS or MSiReader as plain-text CSV, or converted from imzML via Cardinal's peakBin() and cardinal2msimat()) into mass2adduct's msimat format and extract ion maps for the adduct candidate and its parent. Verify the mass difference matches the candidate adduct (e.g., DHB-H2O, mass 136.01600) using adductMatch against the built-in adducts reference table at high mass resolution. Compare the deposited MS2 spectrum of the adduct peak with the deposited MS2 spectrum of the parent [M+H]+ ion: the adduct call is supported only if the fragment-ion series is identical in both m/z values and relative intensities, and includes the neutral-loss product (for DHB-H2O adducts, the fragment at the parent m/z, e.g., 788.616 from 924.632). Compute spatial correlation between the two ion maps with corrPairsMSI (default two-tailed Pearson, p=0.05 with Bonferroni correction); for the mouse brain case, the paper reports r ≈ 0.9 for the PC(36:1) parent–adduct pair. Write the fragment-spectrum comparison summary and correlation result to a verification CSV for audit.

## Related tools

- **mass2adduct** (R package providing adductMatch (match mass differences to known adducts like DHB-H2O at 136.01600), corrPairsMSI (two-tailed Pearson correlation of parent–adduct ion maps with Bonferroni-corrected p=0.05), and msimat/cardinal2msimat for importing the MSI intensity matrix) — https://github.com/kbseah/mass2adduct
- **R** (Execution environment for mass2adduct; the adduct-confirmation workflow (msimat import, adductMatch, corrPairsMSI, CSV export) is run as R code with library(mass2adduct))
- **imzML** (Raw MALDI-MSI data format that is read into R (e.g., via Cardinal's readImzML, normalized, smoothed, baseline-reduced and peak-binned) before conversion to msimat for correlation analysis)

## Examples

```
library(mass2adduct); d.diff.DHBH2O <- diffGetPeaks(d.diff, mass=136.01600, mDa=1); d.diff.DHBH2O.corr <- corrPairsMSI(d, d.diff.DHBH2O)
```

## Evaluation signals

- The MS2 fragment lists of the adduct peak (e.g., m/z 924.632) and the parent ion (m/z 788.617) are identical in m/z values and relative intensities, including the fragment at m/z 788.616 arising from neutral loss of DHB-H2O
- adductMatch against the built-in adducts table confirms the mass difference corresponds to the candidate adduct (DHB-H2O, mass 136.01600) within the specified window (e.g., +/- 0.5 mDa or ppm-proportional)
- corrPairsMSI returns a Pearson correlation consistent with the reported value (r ≈ 0.9 for the PC(36:1) parent–adduct pair) with a significant Bonferroni-corrected p-value (p < 0.05)
- The verification CSV (e.g., adduct_ms2_correlation_verification.csv) is written and contains both the fragment-spectrum comparison summary and the correlation result
- The ion maps of both m/z values extracted from the msimat object show visually co-localized spatial distributions, as expected for true parent–adduct pairs

## Limitations

- Calculated mass differences are 'misleadingly precise, because measurement error and uncertainty are not taken into account' and must be binned into a histogram with a bin width appropriate to the instrument's mass precision (e.g., 0.01 in the README example) before adduct matching
- corrPairsMSI requires the peak-intensity table to fit in memory; for large datasets the README advises using corrPairsMSIchunks with a mem.limit=N parameter instead
- The MS2 confirmation depends on deposited fragment spectra; fragment lists must match in both m/z and relative intensities — a match in m/z alone is insufficient
- Adduct abundance estimation relies on co-localization and high mass resolution; low-resolution data may produce ambiguous adductMatch results

## Evidence

- [other] On-tissue MS2 fragmentation confirmed m/z 924.632 as the [PC(36:1) + (DHB-H2O) + H]+ adduct of PC(36:1), showing identical fragments to [PC(36:1) + H]+ including the fragment at m/z 788.616 arising from neutral loss of DHB-H2O.: [evidence span withheld — anchor status: not_found]
- [abstract] Metabolites ranging from lipids to amines and amino acids form matrix adducts with the commonly used 2,5-dihydroxybenzoic acid (DHB) matrix: "Metabolites ranging from lipids to amines and amino acids form matrix adducts with the commonly used 2,5-dihydroxybenzoic acid (DHB) matrix"
- [methods] This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object.: "This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object."
- [readme] If they are truly related by molecular adduct formation, then their abundances should be correlated.: [evidence span withheld — anchor status: external_doc]
- [readme] The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above.: "The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above."
