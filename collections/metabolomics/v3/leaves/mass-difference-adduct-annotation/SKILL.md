---
name: mass-difference-adduct-annotation
description: Use when a pre-processed MALDI-MSI peak table (plain-text CSV exported
  from SCiLS or MSiReader, or a binned Cardinal MSProcessedImagingExperiment) needs
  putative adduct annotation, and you want to recover abundant Δmass peaks such as
  21.985 and 37.955 Da from pairwise peak differences.
when_to_use_negative:
- The input spectrum still contains matrix-cluster peaks — template subtraction (subtractPeaks)
  should be run before re-running adduct identification, not this annotation step
  on raw peaks
- You only need peak picking or matrix subtraction rather than adduct assignment —
  this skill assumes an already pre-processed, peak-binned table, not raw imzML spectra
- A very large CSV will not fit in memory and no triplet-format conversion has been
  done — use the msimunging.pl pre-step or corrPairsMSIchunks instead of plain corrPairsMSI
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mass2adduct
  role: Core R package providing msimat import, massdiff pairwise difference computation,
    adductMatch/topAdducts annotation against the built-in adducts reference, diffGetPeaks
    and corrPairsMSI for spatial confirmation.
  repo: github.com/kbseah/mass2adduct
- name: R
  role: Runtime environment; mass2adduct is installed from GitHub with devtools and
    its functions are called from R scripts.
- name: SCiLS
  role: Upstream MALDI-MSI software from which the plain-text CSV peak tables are
    exported before import into mass2adduct.
- name: MSiReader
  role: Alternative upstream MSI software whose 'intensity export' output (CSV) serves
    as the primary import format for msimat().
- name: Cardinal
  role: R MSI package; peak-binned MSProcessedImagingExperiment/MSContinuousImagingExperiment
    objects are converted with cardinal2msimat() for use with mass2adduct.
  repo: http://cardinalmsi.org/
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:5929ebf65d173020ab56e35f9d3ff64006827364f930e0ae8927b67e2630b9d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - MSiReader
  - R
  - SCiLS
  - imzML
  - mass2adduct
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# mass-difference-adduct-annotation

## Summary

A MALDI-MSI skill that tabulates all pairwise mass differences between detected peaks, bins them into a Δmass histogram, and matches abundant bins against a reference adduct table (e.g. the built-in `adducts` list in mass2adduct) to annotate metabolite–matrix and salt adducts such as [M + Na]+ and [M + (DHB-H2O) + H]+.

## When to use

Use when a pre-processed MALDI-MSI peak table (plain-text CSV exported from SCiLS or MSiReader, or a binned Cardinal MSProcessedImagingExperiment) needs putative adduct annotation, and you want to recover abundant Δmass peaks such as 21.985 and 37.955 Da from pairwise peak differences.

## When NOT to use

- The input spectrum still contains matrix-cluster peaks — template subtraction (subtractPeaks) should be run before re-running adduct identification, not this annotation step on raw peaks
- You only need peak picking or matrix subtraction rather than adduct assignment — this skill assumes an already pre-processed, peak-binned table, not raw imzML spectra
- A very large CSV will not fit in memory and no triplet-format conversion has been done — use the msimunging.pl pre-step or corrPairsMSIchunks instead of plain corrPairsMSI

## Inputs

- Plain-text comma-delimited peak-intensity table (CSV) exported from SCiLS or MSiReader via the 'intensity export' function
- Peak-binned Cardinal `MSProcessedImagingExperiment` or `MSContinuousImagingExperiment` object (Cardinal v2.2+, processed with peakBin())
- Plain-text file of mass values, one per line (when pixel intensities are not needed)
- Custom adduct reference data.frame with `name`, `formula`, `mass` columns, or the built-in `adducts`/`adducts2` lists

## Outputs

- `massdiff` object holding parent mass, derivative mass, and their difference for every peak pair
- R histogram object of binned mass differences, plottable with plot(d.diff.hist, xlim=c(100,150)) to zoom into regions
- Ranked table of top mass differences with closest matches to known adducts (from `adductMatch` / `topAdducts`)
- Subset `massdiff` object of parent/derivative peak pairs for a specific adduct (e.g. DHB-H2O at 136.01600)
- data.frame of Pearson correlation p-values (Bonferroni-corrected, default cutoff p=0.05) for each candidate parent/adduct ion pair

## How to apply

Import the peak table into R as a mass2adduct `msimat` object with `msimat()` (or convert a peak-binned Cardinal experiment with `cardinal2msimat()`), then run `massdiff(peaks)` to compute the mass difference for every pair of peaks. Bin the resulting `massdiff` object into a histogram with a bin width matched to the instrument's mass precision (0.01 is used in the README example), because raw pairwise differences are misleadingly precise before measurement error is accounted for. Call `adductMatch(d.diff.hist)` to find closest-matching bins against the built-in `adducts`/`adducts2` reference (a data.frame with `name`, `formula`, `mass` columns) and `topAdducts(d.diff.hist, n=10)` to also surface abundant unannotated differences. For the mouse-brain dataset this recovers prominent Δmass peaks at ~21.985 Da ([M + H]+/[M + Na]+), ~37.955 Da ([M + H]+/[M + K]+), and ~136.016 Da (DHB-H2O). Downstream, use `diffGetPeaks(d.diff, mass=136.01600, mDa=1)` to extract the parent/derivative peak pairs and `corrPairsMSI(d, subset)` for a spatial Pearson correlation test (p=0.05, Bonferroni-corrected) confirming the candidate adduct pairs co-localize.

## Related tools

- **mass2adduct** (Core R package providing msimat import, massdiff pairwise difference computation, adductMatch/topAdducts annotation against the built-in adducts reference, diffGetPeaks and corrPairsMSI for spatial confirmation.) — github.com/kbseah/mass2adduct
- **R** (Runtime environment; mass2adduct is installed from GitHub with devtools and its functions are called from R scripts.)
- **SCiLS** (Upstream MALDI-MSI software from which the plain-text CSV peak tables are exported before import into mass2adduct.)
- **MSiReader** (Alternative upstream MSI software whose 'intensity export' output (CSV) serves as the primary import format for msimat().)
- **Cardinal** (R MSI package; peak-binned MSProcessedImagingExperiment/MSContinuousImagingExperiment objects are converted with cardinal2msimat() for use with mass2adduct.) — http://cardinalmsi.org/

## Examples

```
Rscript -e 'library(mass2adduct); d <- msimat("msi.csv", sep=";"); d.diff <- massdiff(d); d.diff.hist <- hist(d.diff); head(adductMatch(d.diff.hist)); topAdducts(d.diff.hist, n=10)'
```

## Evaluation signals

- Histogram must show prominent Δmass peaks at ~21.985 and ~37.955 Da (Na/K adducts) plus 136.016 Da (DHB-H2O), above the integer-spaced background, matching Figure 1A of the article
- adductMatch() output should report counts and quantile values for each matched adduct bin; quantiles are expected to be high because most mass differences have zero to few counts
- diffGetPeaks() window must resolve the specific adduct within ±0.5 mDa (or an equivalent ppm window) around the reference mass, e.g. 136.01600 for DHB-H2O
- corrPairsMSI() p-values must pass the default significance cutoff of p=0.05 with Bonferroni correction for true parent/adduct pairs
- Dataset-level invariant: metabolite–matrix adducts should account for up to one third of total peak counts when fully annotated

## Limitations

- Pairwise mass differences are misleadingly precise before binning; the bin width must match the known mass precision of the instrument (0.01 used in the example)
- Correlation tests assume pixel-intensity data are available; for lists of masses alone, only histogram-based matching is possible
- Large CSV matrices (several Gb) may exceed RAM; msimunging.pl triplet conversion is memory-limited too (e.g. a 40 Gb CSV with 32 Gb RAM will likely fail)
- adductMatch only finds adducts present in the reference table; topAdducts is needed to surface abundant differences with no reference match
- Results are putative assignments, not confirmed identifications — co-localization and high mass accuracy only estimate adduct abundance

## Evidence

- [abstract] pairwise Δmass histogram recovers abundant adducts: "We developed a software package, mass2adduct, for the community for an automated putative assignment and quantification of metabolite–matrix adducts"
- [methods] massdiff computes all pairwise peak differences: "Take all possible pairs of masses and calculate the mass difference for each pair. These mass differences represent potential molecular adducts."
- [methods] adductMatch finds closest-matching reference bin: "The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above."
- [methods] topAdducts ranks differences by occurrence: "`topAdducts` ranks mass differences by their occurrences, and reports them in descending order, as well as matches to known adducts, if any"
- [readme] histogram bin width matched to mass precision: [evidence span withheld — anchor status: external_doc]
- [readme] spatial correlation confirms candidate pairs: [evidence span withheld — anchor status: external_doc]
- [abstract] co-localization and high mass resolution estimate abundance: "We took advantage of the co-localization of adducts with their parent ions and the accuracy of high mass resolution to estimate adduct abundance"
