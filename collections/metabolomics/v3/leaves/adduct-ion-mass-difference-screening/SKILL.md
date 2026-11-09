---
name: adduct-ion-mass-difference-screening
description: Use when a pixel-by-m/z feature matrix from an imaging MS dataset (e.g.
  AFAD-ESI mouse brain, 14,260 pixels × 3,044 ions) needs isotope/adduct decomposition
  before metabolite annotation, and the fraction of adduct vs. isotope peaks must
  be quantified.
when_to_use_negative:
- The input is raw MS imaging spectra rather than a peak-picked pixel × m/z feature
  matrix, since the workflow operates on extracted m/z peak lists
- The data lack spatial pixel coordinates, because candidate adduct pairs are confirmed
  via spatial (Moran's I) correlation across pixels and cannot be validated without
  them
- The goal is database-driven metabolite annotation (HMDB/KEGG/LIPIDMAPS scoring by
  mass accuracy, isotopic similarity, adduct evidence) rather than classifying peaks
  as isotope/adduct pairs
edam_topics: []
tools:
- name: SMAnalyst
  role: Web/R platform whose Metabolite Annotation module performs isotope/adduct
    identification with spatial correlation validation on the uploaded feature matrix
  repo: https://github.com/mzlab-research/SManalyst
- name: MetaboCoreUtils
  role: R package providing the isotopologues function used to screen peak pairs conforming
    to theoretical isotopic/adduct mass differences within a user-specified ppm tolerance
- name: spdep (moran_bv)
  role: Computes spatial correlation across pixels to validate candidate isotope/adduct
    pairs; pairs exceeding the default threshold of 0.5 are retained
- name: R (>= v4.1.0)
  role: Runtime environment for local execution of the SManalyst Shiny app and its
    adduct-screening workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:c580d28e3ebe2b264018cbc71b601d76c23b994ccb8a954a70a8179a07dcef41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - MetaboCoreUtils
  - R (>= v4.1.0)
  - SMAnalyst
  - spdep (moran_bv)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# adduct-ion-mass-difference-screening

## Summary

Classifies ion peaks in a spatial metabolomics feature matrix as adduct ions by screening for the common positive-mode adduct mass differences ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+) and validating candidate pairs by spatial correlation across pixels, yielding the fraction of total ion peaks attributable to adducts.

## When to use

Use when a pixel-by-m/z feature matrix from an imaging MS dataset (e.g. AFAD-ESI mouse brain, 14,260 pixels × 3,044 ions) needs isotope/adduct decomposition before metabolite annotation, and the fraction of adduct vs. isotope peaks must be quantified.

## When NOT to use

- The input is raw MS imaging spectra rather than a peak-picked pixel × m/z feature matrix, since the workflow operates on extracted m/z peak lists
- The data lack spatial pixel coordinates, because candidate adduct pairs are confirmed via spatial (Moran's I) correlation across pixels and cannot be validated without them
- The goal is database-driven metabolite annotation (HMDB/KEGG/LIPIDMAPS scoring by mass accuracy, isotopic similarity, adduct evidence) rather than classifying peaks as isotope/adduct pairs

## Inputs

- Feature matrix CSV in SMAnalyst format: columns 1–2 = pixel X,Y coordinates; subsequent columns = ion intensities at each m/z; one row per spatial pixel
- m/z peak list with per-pixel intensities from the Dataset 1 mouse brain AFAD-ESI(+) section (NGDC OMIX OMIX009541; 14,260 pixels × 3,044 unique ion peaks)
- User-specified mass error tolerance in ppm for candidate pair screening

## Outputs

- isotope_adduct_identification.csv containing all isotope/adduct pair assignments
- Percentage of total ion peaks classified as adduct ions ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+) and as isotopic peaks
- List of validated adduct ion pairs passing the spatial correlation threshold (default 0.5)

## How to apply

Load the feature matrix (columns 1–2 are X,Y pixel coordinates; remaining columns are ion intensities at each m/z) and extract the m/z peak list with pixel intensities. First remove isotopic peak pairs using MetaboCoreUtils' isotopologues function to screen for theoretical isotopic mass differences within a user-specified ppm mass error tolerance, validated spatially with moran_bv (spdep). Screen the remaining monoisotopic ions for the common adduct mass differences [M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+, and confirm each adduct ion pair with the same spatial correlation approach using the default moran_bv correlation threshold of 0.5. Compute the percentage of total ion peaks classified as adduct ions (Dataset 1 reported 4.19% of total ions, alongside 5.92% isotopic peaks) and write all pair assignments and fractions to isotope_adduct_identification.csv. The spatial-correlation step is the rationale for pair validation: real isotope/adduct partners must co-vary across tissue pixels, not merely differ by a nominal mass.

## Related tools

- **SMAnalyst** (Web/R platform whose Metabolite Annotation module performs isotope/adduct identification with spatial correlation validation on the uploaded feature matrix) — https://github.com/mzlab-research/SManalyst
- **MetaboCoreUtils** (R package providing the isotopologues function used to screen peak pairs conforming to theoretical isotopic/adduct mass differences within a user-specified ppm tolerance)
- **spdep (moran_bv)** (Computes spatial correlation across pixels to validate candidate isotope/adduct pairs; pairs exceeding the default threshold of 0.5 are retained)
- **R (>= v4.1.0)** (Runtime environment for local execution of the SManalyst Shiny app and its adduct-screening workflow)

## Examples

```
library(MetaboCoreUtils); library(spdep); pairs <- isotopologues(mz_peak_list, ppm = tol); valid <- Filter(function(p) moran_bv(m[, p], nb, wt)$I[1] > 0.5, pairs)
```

## Evaluation signals

- Reproduced adduct fraction on Dataset 1 matches the reported 4.19% of total ion peaks ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+)
- Reproduced isotopic-peak fraction on Dataset 1 matches the reported 5.92% of total ion peaks
- Every retained isotope/adduct pair exceeds the moran_bv spatial correlation threshold (default 0.5) across the 14,260 pixels
- Candidate pairs satisfy theoretical mass differences within the user-specified ppm tolerance before spatial validation
- isotope_adduct_identification.csv contains pair assignments and the computed fractions for both isotope and adduct classes

## Limitations

- Fractions depend on user-defined parameters (mass error tolerance in ppm and the spatial correlation threshold, default 0.5), so reproduced percentages can shift with different settings
- Requires spatial pixel structure for pair validation; adduct assignments cannot be confirmed on non-spatial peak tables
- The reported validation values (5.92% isotopic, 4.19% adduct) are specific to the mouse brain AFAD-ESI(+) Dataset 1 at 100 μm resolution (OMIX009541) and may not transfer to other tissues or acquisition modes

## Evidence

- [readme] Isotope/adduct identification with spatial correlation validation: "Isotope/adduct identification with spatial correlation validation"
- [intro] a comprehensive metabolite annotation scoring system (mass accuracy, isotopic similarity, adduct evidence): "a comprehensive metabolite annotation scoring system (mass accuracy, isotopic similarity, adduct evidence)"
- [readme] Content: 14,260 pixels × 3,044 ions: "Content: 14,260 pixels × 3,044 ions"
