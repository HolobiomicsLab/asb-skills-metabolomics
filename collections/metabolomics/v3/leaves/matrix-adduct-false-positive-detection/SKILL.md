---
name: matrix-adduct-false-positive-detection
description: Use when a MALDI-MSI peak list (e.g. METASPACE annotations against LIPID_MAPS)
  contains exact-mass-only annotations and the matrix (DHB, 2,5-dihydroxybenzoic acid)
  may form adducts with metabolites, so false-positive ion assignments must be ruled
  out.
when_to_use_negative:
- The data are LC-MS or ESI-MS without a MALDI matrix, where matrix adducts like DHB-H2O
  are not expected.
- The input is already a validated metabolite annotation table where every peak has
  been re-annotated including adduct forms - this skill would be redundant.
- Only a single isolated m/z value is available without a full peak list, so pairwise
  mass-difference screening cannot be performed.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mass2adduct
  role: R package providing msimat import, massdiff pairwise difference computation,
    hist/adductMatch adduct matching, diffGetPeaks peak-pair extraction, and corrPairsMSI
    spatial correlation to identify matrix adducts in MALDI-MSI data
  repo: https://github.com/kbseah/mass2adduct
- name: R
  role: Runtime environment in which mass2adduct is installed and the adduct-screening
    workflow is executed
- name: Cardinal
  role: R package used upstream to read imzML and pre-process MSI data (normalize,
    smoothSignal, reduceBaseline, peakBin) before conversion with cardinal2msimat()
  repo: http://cardinalmsi.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:3ea150f1db381c5f278cbacc063e1b6699d0b20d2143dd48262a48455f155a22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbseah/mass2adduct
  tools:
  - Cardinal
  - R
  - mass2adduct
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# matrix-adduct-false-positive-detection

## Summary

Screen MALDI-MSI peak lists for metabolite-matrix adducts (e.g. DHB-derived [M + (DHB-H2O) + H]+) that masquerade as metabolite annotations, exposing exact-mass false positives such as PS(21:0/22:6) misassigned to a PC parent-DHB adduct.

## When to use

Use when a MALDI-MSI peak list (e.g. METASPACE annotations against LIPID_MAPS) contains exact-mass-only annotations and the matrix (DHB, 2,5-dihydroxybenzoic acid) may form adducts with metabolites, so false-positive ion assignments must be ruled out.

## When NOT to use

- The data are LC-MS or ESI-MS without a MALDI matrix, where matrix adducts like DHB-H2O are not expected.
- The input is already a validated metabolite annotation table where every peak has been re-annotated including adduct forms - this skill would be redundant.
- Only a single isolated m/z value is available without a full peak list, so pairwise mass-difference screening cannot be performed.

## Inputs

- MSI peak intensity table exported as plain-text CSV from SCiLS or MSiReader (imported with msimat())
- Cardinal MSProcessedImagingExperiment or MSContinuousImagingExperiment object (pre-processed with peakBin())
- Plain text list of mass values (one per line) when pixel intensities are not needed
- Mass difference histogram object produced by mass2adduct
- METASPACE annotation list (e.g. dataset MPIMM_030_QE_P_BP_CF_10 vs LIPID_MAPS-2016, FDR < 20%)

## Outputs

- massdiff object of all pairwise mass differences
- Histogram of binned mass differences (R hist object)
- adductMatch() table of known-adduct matches with counts and quantiles
- topAdducts() ranked list of highest-count mass differences
- corrPairsMSI() data.frame of spatial-correlation p-values per ion pair
- Set of flagged peaks whose annotation conflicts with a matrix-adduct interpretation (e.g. adduct_check_results.csv)

## How to apply

First import the MSI peak data into mass2adduct's msimat format (from SCiLS/MSiReader CSV export, or a Cardinal MSProcessedImagingExperiment/MSContinuousImagingExperiment pre-processed with peakBin, or a plain mass list). Tabulate all pairwise mass differences with massdiff(), then bin them into a histogram (e.g. 0.01 bin width) and match against the built-in adducts/adducts2 reference sets via adductMatch(), which finds the closest-matching bin for known adduct formulas like [M + (DHB-H2O) + H]+ (mass 136.01600). Use diffGetPeaks() with a 1 mDa (or ppm) window to pull out parent/derivative peak pairs for a specific adduct, then corrPairsMSI() to test spatial co-localization (Pearson, two-tailed, p=0.05 with Bonferroni correction) - a true adduct should co-localize with its parent ion. Finally, cross-check surviving adduct-matched peaks against the annotation platform's assignments: if a peak's METASPACE annotation implies a molecular formula different from the parent by the adduct's formula (e.g. C7H4O3 for the DHB-H2O adduct), flag it as a matrix-adduct false positive.

## Related tools

- **mass2adduct** (R package providing msimat import, massdiff pairwise difference computation, hist/adductMatch adduct matching, diffGetPeaks peak-pair extraction, and corrPairsMSI spatial correlation to identify matrix adducts in MALDI-MSI data) — https://github.com/kbseah/mass2adduct
- **R** (Runtime environment in which mass2adduct is installed and the adduct-screening workflow is executed)
- **Cardinal** (R package used upstream to read imzML and pre-process MSI data (normalize, smoothSignal, reduceBaseline, peakBin) before conversion with cardinal2msimat()) — http://cardinalmsi.org/

## Examples

```
Rscript -e 'library(mass2adduct); d <- msimat("msi.csv", sep=";"); d.diff <- massdiff(d); d.diff.hist <- hist(d.diff); print(head(adductMatch(d.diff.hist))); pair <- diffGetPeaks(d.diff, mass=136.01600, mDa=1); print(corrPairsMSI(d, pair))'
```

## Evaluation signals

- The known DHB-H2O adduct mass 136.01600 is recovered by adductMatch/diffGetPeaks within a 1 mDa window from real MSI data.
- Spatial correlation of candidate parent-adduct pairs passes corrPairsMSI significance (p < 0.05, Bonferroni-corrected).
- Cross-checking the flagged peak's annotation against the adduct formula reveals a formula offset (e.g. C7H4O3 for PC(P-16:0/18:2) + (DHB-H2O) + H vs its METASPACE annotation PS(21:0/22:6)).
- The peak pair's co-localization with the parent ion image visually matches (pickAdductPairImage / pointsAdducts plots).
- Up to one third of total peak counts in MALDI-MSI data may be attributable to matrix adducts - flagged adduct fraction should be reported against this scale.

## Limitations

- Computed mass differences are 'misleadingly precise, because measurement error and uncertainty are not taken into account' - they must be binned into a histogram with an instrument-appropriate bin width before adduct matching.
- Quantile values from adductMatch will usually be quite high because the majority of mass differences have zero to few counts; counts alone are a weak signal.
- corrPairsMSI on large datasets may exceed memory - use corrPairsMSIchunks with mem.limit=N to process serially in chunks.
- Large SCiLS/MSiReader CSVs (tens of Gb) need pre-filtering with msimunging.pl to triplet format before import into R.
- The method only screens for adducts in the built-in or user-supplied reference list; unknown adducts require topAdducts() manual inspection and remain putative without spatial correlation support.

## Evidence

- [abstract] metabolite-matrix adducts influence up to one third of total peak counts: "reveals a major influence of so far unrecognized metabolite–matrix
adducts on total peak counts (up to one third)"
- [abstract] take advantage of co-localization and high mass resolution: "We took advantage of the co-localization of adducts with their parent ions and the accuracy of high mass resolution to estimate
adduct abundance"
- [discussion] mass difference and spatial correlation workflow: "perform a simple mass difference calculation and spatial correlation analysis as a rapid and efficient way to screen for these adducts in existing MSI datasets"
- [discussion] false annotations are a risk of omitting adducts: "including metabolite–matrix adducts into database annotations can reduce the number of unannotated peaks and, on the other hand, will prevent possible false annotations"
- [methods] massdiff computes pairwise mass differences: "Take all possible pairs of masses and calculate the mass difference for each pair. These mass differences represent potential molecular adducts."
- [readme] histogram binning rationale: [evidence span withheld — anchor status: external_doc]
- [readme] corrPairsMSI default significance threshold: [evidence span withheld — anchor status: external_doc]
- [readme] diffGetPeaks window parameters: [evidence span withheld — anchor status: external_doc]
- [readme] adductMatch method: "The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above."
- [readme] memory limitation for correlation: [evidence span withheld — anchor status: external_doc]
