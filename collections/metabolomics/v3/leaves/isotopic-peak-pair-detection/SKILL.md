---
name: isotopic-peak-pair-detection
description: Use when a pixel x m/z feature matrix from an imaging mass spectrometry
  run (e.g., AFAD-ESI positive mode) must be classified into isotopic peak pairs and
  adduct ion pairs ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+) before metabolite
  annotation.
when_to_use_negative:
- Input is raw spectra or an mzML/imzML file that has not yet been peak-picked into
  a pixel x m/z feature matrix
- Data lack spatial pixel coordinates, so moran_bv-based spatial correlation validation
  cannot be computed
- The goal is database-driven metabolite annotation (mass accuracy, isotope pattern,
  adduct multi-evidence scoring) rather than classifying peaks into isotope/adduct
  pairs
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0092
tools:
- name: SMAnalyst
  role: Web platform whose metabolite-annotation module performs isotope/adduct identification
    with spatial correlation validation on the uploaded feature matrix
  repo: https://github.com/mzlab-research/SManalyst
- name: MetaboCoreUtils
  role: R package providing the isotopologues function used to screen peak pairs for
    theoretical isotopic mass differences within a ppm tolerance
- name: spdep
  role: R package providing the moran_bv function used to compute spatial correlation
    of candidate peak pairs across pixels and retain pairs above the correlation threshold
- name: R (>= v4.1.0)
  role: Runtime environment in which the SMAnalyst isotope/adduct identification workflow
    (MetaboCoreUtils + spdep) is executed locally
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:a61293710a637ed86930a9154f718683f23da27b2a70c4ca66dbf4623ad92f1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - MetaboCoreUtils
  - R (>= v4.1.0)
  - SMAnalyst
  - spdep
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# isotopic-peak-pair-detection

## Summary

Identifies which peaks in a spatial metabolomics feature matrix are isotopic peaks or adduct ions by screening m/z peak pairs for theoretical isotopic/adduct mass differences and validating each candidate pair with spatial correlation across pixels, as implemented in SMAnalyst's isotope/adduct identification module.

## When to use

Use when a pixel x m/z feature matrix from an imaging mass spectrometry run (e.g., AFAD-ESI positive mode) must be classified into isotopic peak pairs and adduct ion pairs ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+) before metabolite annotation.

## When NOT to use

- Input is raw spectra or an mzML/imzML file that has not yet been peak-picked into a pixel x m/z feature matrix
- Data lack spatial pixel coordinates, so moran_bv-based spatial correlation validation cannot be computed
- The goal is database-driven metabolite annotation (mass accuracy, isotope pattern, adduct multi-evidence scoring) rather than classifying peaks into isotope/adduct pairs

## Inputs

- Feature matrix CSV: columns 1-2 = pixel coordinates (X, Y), remaining columns = ion intensities at each m/z value, one row per spatial pixel
- m/z peak list with per-pixel intensities extracted from the feature matrix
- User-specified mass error tolerance (ppm) for isotopic mass differences
- Spatial correlation threshold (default 0.5 for adduct ion pairs)

## Outputs

- isotope_adduct_identification.csv: all isotope/adduct pair assignments and the percentage of total ion peaks classified as isotopic peaks and as adduct ions
- Percentage of total ion peaks identified as isotopic peaks (e.g., 5.92% on mouse brain Dataset 1)
- Percentage of total ions identified as adduct ions (e.g., 4.19% on Dataset 1)

## How to apply

Load the feature matrix (columns 1-2 are pixel X/Y coordinates, remaining columns are ion intensities per m/z; e.g., the SMAnalyst mouse brain Dataset 1 of 14,260 pixels x 3,044 unique ion peaks) and extract the m/z peak list with per-pixel intensities. Run the isotopologues function from the MetaboCoreUtils package to screen for peak pairs matching theoretical isotopic mass differences within a user-specified ppm mass error tolerance. Validate each candidate isotopic pair with the moran_bv function from the spdep package, computing spatial correlation across pixels and retaining only pairs exceeding the user-defined correlation threshold. For the surviving monoisotopic ions, screen for the common positive-mode adduct mass differences ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+) and confirm adduct pairs with the same moran_bv spatial correlation test using the default threshold of 0.5. Compute the fraction of total ion peaks classified as isotopic peaks and as adduct ions; the rationale is that true isotope/adduct pairs must agree both in mass difference and in spatial intensity pattern across tissue pixels, filtering out chance mass-difference coincidences.

## Related tools

- **SMAnalyst** (Web platform whose metabolite-annotation module performs isotope/adduct identification with spatial correlation validation on the uploaded feature matrix) — https://github.com/mzlab-research/SManalyst
- **MetaboCoreUtils** (R package providing the isotopologues function used to screen peak pairs for theoretical isotopic mass differences within a ppm tolerance)
- **spdep** (R package providing the moran_bv function used to compute spatial correlation of candidate peak pairs across pixels and retain pairs above the correlation threshold)
- **R (>= v4.1.0)** (Runtime environment in which the SMAnalyst isotope/adduct identification workflow (MetaboCoreUtils + spdep) is executed locally)

## Examples

```
Rscript isotope_adduct_identification.R  # writes isotope_adduct_identification.csv with pair assignments and isotope/adduct fractions
```

## Evaluation signals

- Reproduced fractions must match the reported values on mouse brain Dataset 1: 5.92% of total ion peaks as isotopic peaks and 4.19% of total ions as adduct ions
- Every retained isotopic/adduct pair must satisfy both the theoretical mass-difference criterion (within the user-specified ppm tolerance) and the spatial correlation threshold (default 0.5 for adducts)
- Adduct assignments are restricted to the supported set: [M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+
- Output CSV must contain all pair assignments plus the computed isotope and adduct fractions of total ion peaks
- Input matrix dimension check: pixel count and unique ion peak count (e.g., 14,260 x 3,044) must match the feature matrix before and after pair screening

## Limitations

- Spatial correlation validation depends on sufficient spatially structured intensity variation; pairs across very few or spatially uniform pixels may not be reliably validated
- Mass error tolerance (ppm) and correlation threshold are user-specified, so overly loose tolerances can inflate isotope/adduct fractions
- Only the listed positive-mode adduct forms ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+) are screened; other adducts are not detected
- Percentages apply to the feature matrix as loaded (e.g., 3,044 unique ion peaks); prior noise-ion filtering changes the denominator

## Evidence

- [readme] README description of the annotation module: "Isotope/adduct identification with spatial correlation validation"
- [readme] README dataset dimensions for the example data: "Acquisition: AFAD-ESI (+) mode, 100μm resolution
*   Content: 14,260 pixels × 3,044 ions"
- [intro] Article intro on the annotation scoring system: "a comprehensive metabolite annotation scoring system (mass accuracy, isotopic similarity, adduct evidence)"
