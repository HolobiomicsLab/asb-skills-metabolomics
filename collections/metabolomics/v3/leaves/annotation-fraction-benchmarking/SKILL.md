---
name: annotation-fraction-benchmarking
description: Use when a pixel-by-m/z feature matrix from an imaging mass spectrometry
  run needs its isotope and adduct peak fractions quantified, or when reproducing
  SMAnalyst's reported annotation percentages (5.92% isotopic, 4.19% adduct) on the
  mouse brain Dataset 1.
when_to_use_negative:
- Input is raw IMS spectra rather than the pre-processed pixel-by-m/z feature matrix
  CSV that SManalyst requires
- The goal is metabolite identity annotation against HMDB/KEGG/LIPIDMAPS databases
  rather than quantifying the isotopic/adduct fraction of peaks
- The dataset has already had isotope/adduct peaks removed or collapsed, so the total-ion-peak
  denominator no longer reflects the full detected peak list
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMAnalyst
  role: Web platform whose isotope/adduct identification module (Metabolite Annotation
    tab) implements the isotopologue screening and spatial-correlation-validated adduct
    detection being benchmarked
  repo: https://metax.genomics.cn/app/SManalyst
- name: R
  role: Runtime (>= v4.1.0) for the local installation carrying MetaboCoreUtils and
    spdep used in the isotope/adduct identification workflow
- name: MetaboCoreUtils
  role: Provides the isotopologues function that screens peak pairs conforming to
    theoretical isotopic mass differences within the ppm tolerance
- name: spdep
  role: Provides the moran_bv function used to validate candidate isotope and adduct
    pairs by spatial correlation across pixels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:cf4ccc1ef137f0c6356436fda6c5e278267950eb181f3de9e22910fe9c199026
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboCoreUtils
  - R
  - SMAnalyst
  - spdep
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# annotation-fraction-benchmarking

## Summary

Benchmark the annotation burden of a spatial metabolomics feature matrix by quantifying what fraction of detected ion peaks are isotopic peaks versus adduct ions, replicating SMAnalyst's isotope/adduct identification workflow with spatial-correlation validation.

## When to use

Use when a pixel-by-m/z feature matrix from an imaging mass spectrometry run needs its isotope and adduct peak fractions quantified, or when reproducing SMAnalyst's reported annotation percentages (5.92% isotopic, 4.19% adduct) on the mouse brain Dataset 1.

## When NOT to use

- Input is raw IMS spectra rather than the pre-processed pixel-by-m/z feature matrix CSV that SManalyst requires
- The goal is metabolite identity annotation against HMDB/KEGG/LIPIDMAPS databases rather than quantifying the isotopic/adduct fraction of peaks
- The dataset has already had isotope/adduct peaks removed or collapsed, so the total-ion-peak denominator no longer reflects the full detected peak list

## Inputs

- Pixel-by-m/z feature matrix in CSV format (columns 1-2: pixel X,Y coordinates; remaining columns: ion intensities at each m/z)
- Mouse brain Dataset 1 example data: 14,260 pixels x 3,044 ions, AFAD-ESI (+) mode, 100 um resolution (built-in via SMAnalyst Tutorial Panel or NGDC OMIX OMIX009541)
- User-specified isotopic mass error tolerance (ppm)
- Spatial correlation threshold (default 0.5 for adduct validation)

## Outputs

- isotope_adduct_identification.csv containing all isotope/adduct pair assignments
- Percentage of total ion peaks identified as isotopic peaks (expected 5.92% on Dataset 1)
- Percentage of total ions identified as adduct ions ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+) (expected 4.19% on Dataset 1)

## How to apply

Load the feature matrix (for Dataset 1: 14,260 pixels x 3,044 unique ion peaks, mouse brain coronal section, AFAD-ESI (+) mode, 100 um resolution, NGDC OMIX OMIX009541) and extract the m/z peak list with per-pixel intensities. Screen for isotopic peak pairs with the isotopologues function from the MetaboCoreUtils R package, using a user-specified mass error tolerance in ppm, so only peak pairs matching theoretical isotopic mass differences survive. Validate each candidate isotopic pair with moran_bv from the spdep package, computing spatial (Moran's) correlation across pixels and retaining only pairs exceeding the user-defined correlation threshold. Screen the remaining monoisotopic ions for the five common positive-mode adduct mass differences ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+), confirming adduct pairs with the same spatial-correlation approach at the default threshold of 0.5. Compute the percentages of total ion peaks classified as isotopic peaks and as adduct ions, and write all pair assignments plus fractions to isotope_adduct_identification.csv.

## Related tools

- **SMAnalyst** (Web platform whose isotope/adduct identification module (Metabolite Annotation tab) implements the isotopologue screening and spatial-correlation-validated adduct detection being benchmarked) — https://metax.genomics.cn/app/SManalyst
- **R** (Runtime (>= v4.1.0) for the local installation carrying MetaboCoreUtils and spdep used in the isotope/adduct identification workflow)
- **MetaboCoreUtils** (Provides the isotopologues function that screens peak pairs conforming to theoretical isotopic mass differences within the ppm tolerance)
- **spdep** (Provides the moran_bv function used to validate candidate isotope and adduct pairs by spatial correlation across pixels)

## Examples

```
Rscript -e 'library(MetaboCoreUtils); library(spdep); iso_pairs <- isotopologues(mz, intensity); adduct_pairs <- screenAdducts(mz)'
```

## Evaluation signals

- Reproduced isotopic-peak fraction matches the reported 5.92% of total ion peaks on Dataset 1
- Reproduced adduct-ion fraction matches the reported 4.19% of total ions on Dataset 1
- Only the five positive-mode adduct types ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+) appear in the adduct assignments
- Every retained isotope/adduct pair exceeds the spatial correlation threshold (default 0.5) as computed by moran_bv
- Denominator check: percentages computed against the full 3,044 unique ion peaks of the 14,260-pixel feature matrix

## Limitations

- Results depend on the user-specified ppm mass error tolerance and correlation threshold; deviating from defaults (e.g., correlation threshold 0.5) will shift the reproduced fractions away from the reported values
- The adduct screen covers only five common positive-mode adducts ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+); other adduct species go unclassified
- Validated on a single mouse brain AFAD-ESI (+) mode dataset; fractions may differ for other tissues, polarities, or pixel sizes

## Evidence

- [readme] Isotope/adduct identification with spatial correlation validation: "Isotope/adduct identification with spatial correlation validation"
- [readme] Test dataset description: "Acquisition: AFAD-ESI (+) mode, 100μm resolution"
- [abstract] Dataset scale: "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
- [intro] Annotation scoring: "a comprehensive metabolite annotation scoring system (mass accuracy, isotopic similarity, adduct evidence)"
- [readme] Raw data accession: "Raw data: [NGDC OMIX Repository](https://ngdc.cncb.ac.cn/omix) ID: OMIX009541"
