---
name: lipid-adduct-annotation-maldi-msi
description: Use when a MALDI-MSI peak list (e.g. imzML or exported intensity CSV
  from SCiLS/MSiReader) contains unassigned high-abundance ions whose pairwise mass
  differences match known matrix adducts, and you need to flag them as matrix-adduct
  duplicates rather than distinct metabolite signals.
when_to_use_negative:
- The ion of interest is a true metabolite signal whose mass difference to a neighbor
  is coincidental — mass-difference matching alone is insufficient; require the spatial-correlation
  and/or MS2 fragment confirmation step before labeling as adduct.
- 'Input is raw, unbinned imzML without preprocessing: mass2adduct needs a peak-binned
  intensity matrix (`peakBin()` must be run in Cardinal first; it does not process
  centroid-free profiles).'
- You only have a simple list of masses and no pixel data — `corrPairsMSI` spatial
  validation is impossible; the workflow reduces to mass-difference screening only.
- The adduct of interest is not in the `adducts`/`adducts2` reference tables and no
  custom data.frame (name, formula, mass) is supplied.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mass2adduct
  role: 'Core R package: imports `msimat` peak-intensity objects, computes `massdiff`
    pairwise differences, matches against built-in `adducts` table via `adductMatch`,
    and tests parent–adduct spatial correlation with `corrPairsMSI`.'
  repo: https://github.com/kbseah/mass2adduct
- name: R
  role: Runtime environment; package is installed via devtools `install_github("kbseah/mass2adduct")`
    with optional `build_vignettes=TRUE`.
  repo: https://github.com/kbseah/mass2adduct
- name: imzML
  role: Emerging-standard raw MSI format read through Cardinal and converted to msimat
    for this workflow.
- name: Cardinal
  role: Reads imzML, normalizes, smooths, reduces baseline, and `peakBin()`s peaks
    before `cardinal2msimat()` converts to msimat.
- name: SCiLS / MSiReader
  role: Standard MSI software whose plain-text CSV 'intensity export' is the primary
    non-Cardinal input format.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:267861323b6d42cb0a4d24761daad3b7304e6fc237f25a2320b2912fb88fdb70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbseah/mass2adduct
  tools:
  - Cardinal
  - R
  - SCiLS / MSiReader
  - imzML
  - mass2adduct
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# lipid-adduct-annotation-maldi-msi

## Summary

Annotates lipid/metabolite ion peaks in MALDI-MSI data as matrix adducts (e.g. [M + (DHB-H2O) + H]+) by combining mass-difference matching against a reference adduct table with spatial co-localization of parent and adduct ion maps, using the R package mass2adduct.

## When to use

Use when a MALDI-MSI peak list (e.g. imzML or exported intensity CSV from SCiLS/MSiReader) contains unassigned high-abundance ions whose pairwise mass differences match known matrix adducts, and you need to flag them as matrix-adduct duplicates rather than distinct metabolite signals.

## When NOT to use

- The ion of interest is a true metabolite signal whose mass difference to a neighbor is coincidental — mass-difference matching alone is insufficient; require the spatial-correlation and/or MS2 fragment confirmation step before labeling as adduct.
- Input is raw, unbinned imzML without preprocessing: mass2adduct needs a peak-binned intensity matrix (`peakBin()` must be run in Cardinal first; it does not process centroid-free profiles).
- You only have a simple list of masses and no pixel data — `corrPairsMSI` spatial validation is impossible; the workflow reduces to mass-difference screening only.
- The adduct of interest is not in the `adducts`/`adducts2` reference tables and no custom data.frame (name, formula, mass) is supplied.

## Inputs

- MALDI-MSI peak-intensity table exported as plain-text CSV from SCiLS or MSiReader ('intensity export'), or Cardinal-processed imzML data peak-binned to a peak list
- imzML raw MSI dataset (e.g. mouse brain MALDI-MSI dataset #10)
- Peak list of measured m/z values (one per line) for adduct-screening without pixel intensities
- On-tissue MS2 spectra for candidate parent/adduct ion pairs (for confirmatory fragment comparison)

## Outputs

- massdiff object of all pairwise mass differences (parent1, parent2, difference)
- Histogram of binned mass differences with matches to known adducts (adductMatch output: counts and quantiles per adduct)
- Ranked table of top mass differences with known-adduct annotations (topAdducts)
- Data.frame of parent–adduct ion pairs with Pearson correlation coefficients and Bonferroni-corrected p-values (e.g. r ≈ 0.9 for PC(36:1) vs its DHB adduct)
- Annotated mass spectrum marking parent vs adduct ions in different colors (pointsAdducts)
- Verified adduct assignment summary, e.g. adduct_ms2_correlation_verification.csv pairing MS2 fragment comparison with spatial correlation result

## How to apply

1. Load the peak-intensity table into mass2adduct as an `msimat` object via `msimat()` (direct CSV import) or `cardinal2msimat()` for Cardinal `MSProcessedImagingExperiment`/`MSContinuousImagingExperiment` data peak-binned with `peakBin()`. 2. Compute all pairwise mass differences with `massdiff()` and bin them into a histogram (e.g. bin width 0.01 m/z, scaled to instrument precision) to convert misleadingly precise measured differences into discrete bins. 3. Screen for candidate adducts using `adductMatch` against the built-in `adducts`/`adducts2` reference table (formulas such as DHB-H2O, mass 136.01600) and rank unmatched differences with `topAdducts`. 4. For a specific candidate (e.g. m/z 924.632 vs parent 788.617 in mouse brain DHB datasets), select the ion pair with `diffGetPeaks` (window ±1 mDa or ppm proportional) and test spatial correlation of the two ion maps with `corrPairsMSI` (two-tailed Pearson, Bonferroni-corrected p < 0.05; use `corrPairsMSIchunks` with `mem.limit` for datasets too large for memory). 5. Confirm adduct identity by comparing the MS2 fragment-ion series of the adduct and its parent — an adduct such as [PC(36:1) + (DHB-H2O) + H]+ fragments to the same product ions as [PC(36:1) + H]+, including the parent ion at m/z 788.616 arising from neutral loss of DHB-H2O. Rationale: co-localization plus high-mass-accuracy mass-difference matching separates real metabolite signals from matrix background that can occupy up to one third of total peak counts.

## Related tools

- **mass2adduct** (Core R package: imports `msimat` peak-intensity objects, computes `massdiff` pairwise differences, matches against built-in `adducts` table via `adductMatch`, and tests parent–adduct spatial correlation with `corrPairsMSI`.) — https://github.com/kbseah/mass2adduct
- **R** (Runtime environment; package is installed via devtools `install_github("kbseah/mass2adduct")` with optional `build_vignettes=TRUE`.) — https://github.com/kbseah/mass2adduct
- **imzML** (Emerging-standard raw MSI format read through Cardinal and converted to msimat for this workflow.)
- **Cardinal** (Reads imzML, normalizes, smooths, reduces baseline, and `peakBin()`s peaks before `cardinal2msimat()` converts to msimat.)
- **SCiLS / MSiReader** (Standard MSI software whose plain-text CSV 'intensity export' is the primary non-Cardinal input format.)

## Examples

```
d <- msimat("msi.csv", sep=";"); d.diff <- massdiff(peaks); d.diff.DHBH2O <- diffGetPeaks(d.diff, mass=136.01600, mDa=1); d.diff.DHBH2O.corr <- corrPairsMSI(d, d.diff.DHBH2O)
```

## Evaluation signals

- Mass difference between candidate adduct and parent ion matches a known matrix adduct in `adducts`/`adducts2` within ±1 mDa (or ppm-proportional) window using `adductMatch`/`diffGetPeaks`.
- Pearson correlation between parent and adduct ion maps is high (e.g. r ≈ 0.9 for the PC(36:1) DHB adduct in mouse brain dataset #10) and Bonferroni-corrected p < 0.05 via `corrPairsMSI`.
- MS2 fragment series of the adduct ion is identical to the parent's product ions, with the parent ion itself present as a fragment via neutral loss (e.g. m/z 788.616 from neutral loss of DHB-H2O in [PC(36:1) + (DHB-H2O) + H]+).
- `msimat` import succeeds and `plot(d)` total-intensity mass spectrum matches the original SCiLS/MSiReader or Cardinal peak histogram after `cardinal2msimat()` conversion.
- Fraction of total peak counts attributed to metabolite–matrix adducts stays within the reported envelope (up to one third) — much higher values suggest mis-annotation or poor peak binning.

## Limitations

- Mass differences are misleadingly precise and must be binned into a histogram whose bin width matches instrument mass precision (e.g. 0.01) before `adductMatch` matching.
- Spatial correlation alone cannot distinguish true adduct pairs from co-localized metabolites; confirmation requires MS2 fragment comparison or on-tissue fragmentation.
- Very large CSV intensity exports (multi-GB) may exceed R memory; use `corrPairsMSIchunks` with `mem.limit=N`, or preprocess with `msimunging.pl` to triplet format and filter low-frequency peaks.
- Quantile statistics from `adductMatch` are usually high because the majority of mass differences have zero to few counts — treat them as ranking, not p-values.
- Filtering must remove low-abundance peaks first; most entries in the intensity matrix are zeros that waste memory and weaken correlations.

## Evidence

- [abstract] Mass-difference matching with spatial correlation workflow: "We took advantage of the co-localization of adducts with their parent ions and the accuracy of high mass resolution to estimate
adduct abundance in 20 datasets from different vendors of mass"
- [abstract] Unannotated peaks occupy up to one third of peak counts: "reveals a major influence of so far unrecognized metabolite–matrix
adducts on total peak counts (up to one third)"
- [intro] mass2adduct package purpose: "The software and installation instructions are available on GitHub at https://github.com/kbseah/mass2adduct"
- [methods] workflow: import, diff, match, correlate: "data objects in the `MSProcessedImagingExperiment` or `MSContinuousImagingExperiment` formats can be converted to mass2adduct's `msimat` format"
- [methods] corrPairsMSI detail: "This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object."
- [methods] peak binning prerequisite: [evidence span withheld — anchor status: external_doc]
- [results] MS2 confirmation of adduct identity: [evidence span withheld — anchor status: not_found]
- [results] Correlation threshold achieved for adduct confirmation: [evidence span withheld — anchor status: not_found]
- [methods] memory workaround for large datasets: "use `corrPairsMSIchunks` instead of `corrPairsMSI`, and specify the maximum available memory with `mem.limit=N`"
- [methods] Adduct window parameters: [evidence span withheld — anchor status: external_doc]
- [methods] adductMatch reference table: "The package comes with a small data set of common molecular adducts, called adducts (use the command help(adducts) to see a description)"
- [intro] Motivation: distinguish background from true metabolite: "we lack easy methods to distinguish chemical background signals from
real metabolite signals."
