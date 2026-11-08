---
name: spatial-autocorrelation-validation-of-peak-pairs
description: Use when a spatial metabolomics feature matrix (pixel coordinates × m/z
  intensities) must be screened for isotopic and adduct ion pairs, and candidate pairs
  need spatial-correlation confirmation to suppress spurious mass-difference matches.
when_to_use_negative:
- The input is already an annotated feature table with isotope/adduct assignments
  rather than a raw pixel × m/z intensity matrix
- Intensity data lacks spatial pixel coordinates, so spatial autocorrelation (moran_bv)
  cannot be computed across pixels
- The goal is metabolite identity annotation against HMDB/KEGG/LIPIDMAPS databases
  rather than isotope/adduct peak-pair classification
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3925
tools:
- name: MetaboCoreUtils
  role: Provides the isotopologues function used to screen peak pairs conforming to
    theoretical isotopic mass differences within a user-specified ppm tolerance
- name: spdep
  role: Provides the moran_bv function used to compute spatial correlation of candidate
    peak pairs across pixels, with a default correlation threshold of 0.5 for adduct
    validation
- name: SMAnalyst
  role: Web/R platform (R ≥ v4.1.0) hosting the isotope/adduct identification workflow
    with spatial correlation validation on the uploaded feature matrix
  repo: mzlab-research/SManalyst
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:62393fa396a783737e1211c40e55fa21ba85a585e091b06347c8e4e805a3e4fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - MetaboCoreUtils
  - SMAnalyst
  - spdep
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3925
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Spatial-autocorrelation validation of peak pairs

## Summary

Classifies ion peaks in a spatial metabolomics feature matrix into isotopic-peak pairs and adduct-ion pairs, then confirms each candidate pair with spatial autocorrelation (Moran's Bv) across pixel coordinates before computing the fraction of total peaks in each class.

## When to use

Use when a spatial metabolomics feature matrix (pixel coordinates × m/z intensities) must be screened for isotopic and adduct ion pairs, and candidate pairs need spatial-correlation confirmation to suppress spurious mass-difference matches.

## When NOT to use

- The input is already an annotated feature table with isotope/adduct assignments rather than a raw pixel × m/z intensity matrix
- Intensity data lacks spatial pixel coordinates, so spatial autocorrelation (moran_bv) cannot be computed across pixels
- The goal is metabolite identity annotation against HMDB/KEGG/LIPIDMAPS databases rather than isotope/adduct peak-pair classification

## Inputs

- Feature matrix CSV in SMAnalyst format (columns 1-2 = pixel X,Y coordinates; remaining columns = ion intensities at each m/z), e.g. 14,260 pixels × 3,044 ions
- m/z peak list with per-pixel intensities extracted from the feature matrix
- User-specified mass error tolerance (ppm) for isotopic/adduct mass-difference matching
- Spatial neighborhood weights over pixel coordinates (for spdep Moran's Bv)

## Outputs

- isotope_adduct_identification.csv containing isotope/adduct pair assignments and the percentage of total ion peaks per class (target: 5.92% isotopic, 4.19% adduct)
- List of validated isotopic peak pairs and adduct ion pairs with their Moran's Bv spatial correlation statistics

## How to apply

Load the feature matrix (e.g., the mouse brain AFAD-ESI positive-mode dataset of 14,260 pixels × 3,044 unique ion peaks from NGDC OMIX OMIX009541) and extract the m/z peak list with per-pixel intensities. Run the isotopologues function from the MetaboCoreUtils R package to screen for peak pairs conforming to theoretical isotopic mass differences within a user-specified mass error tolerance in ppm. Validate each candidate isotopic pair with the moran_bv function from the spdep package by computing spatial correlation across pixels, retaining only pairs that exceed the user-defined correlation threshold. Then screen remaining monoisotopic ions for the common adduct mass differences ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+) and confirm adduct ion pairs with the same spatial correlation approach using the default correlation threshold of 0.5. Finally compute the percentage of total ion peaks classified as isotopic peaks and as adduct ions and write all pair assignments and fractions to isotope_adduct_identification.csv.

## Related tools

- **MetaboCoreUtils** (Provides the isotopologues function used to screen peak pairs conforming to theoretical isotopic mass differences within a user-specified ppm tolerance)
- **spdep** (Provides the moran_bv function used to compute spatial correlation of candidate peak pairs across pixels, with a default correlation threshold of 0.5 for adduct validation)
- **SMAnalyst** (Web/R platform (R ≥ v4.1.0) hosting the isotope/adduct identification workflow with spatial correlation validation on the uploaded feature matrix) — mzlab-research/SManalyst

## Examples

```
Rscript -e "pairs <- MetaboCoreUtils::isotopologues(mz, tolerance = ppm_tol); moran_bv(as.numeric(intensities), listw)"
```

## Evaluation signals

- Reproduced fractions on the mouse brain Dataset 1 match the reported values: 5.92% isotopic peaks and 4.19% adduct ions of total ion peaks
- Every retained isotope/adduct pair conforms to the theoretical isotopic or adduct mass difference ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+) within the user-specified ppm tolerance
- Retained pairs exceed the spatial correlation threshold (default 0.5 for adduct pairs) under spdep moran_bv
- isotope_adduct_identification.csv is produced containing all pair assignments and the computed fractions
- Fractions are computed over the full 3,044-ion peak set of the 14,260-pixel feature matrix, not a filtered subset, unless QC (noise ion threshold score ≥ 30) is deliberately applied

## Limitations

- The spatial correlation threshold is user-defined (default 0.5 for adducts), so stricter/looser thresholds change which pairs are retained and the resulting percentages
- Validation reported here is specific to the mouse brain coronal section acquired in AFAD-ESI (+) mode at 100 μm resolution (OMIX009541); behavior on other tissues/modes is not established from this context
- Adduct screening covers only [M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, and [M+H-H2O]+; other adduct species are not classified
- Requires R ≥ v4.1.0 with spdep and MetaboCoreUtils installed locally when not using the SMAnalyst web app

## Evidence

- [abstract] a comprehensive metabolite annotation scoring system (mass accuracy, isotopic similarity, adduct evidence): "a comprehensive metabolite annotation scoring system (mass accuracy, isotopic similarity, adduct evidence)"
- [abstract] Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks): "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
- [readme] Isotope/adduct identification with spatial correlation validation: "Isotope/adduct identification with spatial correlation validation"
- [intro] Set mass error tolerance (ppm): "Set mass error tolerance (ppm)"
- [readme] Metabolite-level clustering: Group ions by spatial co-expression patterns: "Metabolite-level clustering: Group ions by spatial co-expression patterns"
- [readme] Raw data: [NGDC OMIX Repository](https://ngdc.cncb.ac.cn/omix) ID: OMIX009541: "Raw data: [NGDC OMIX Repository](https://ngdc.cncb.ac.cn/omix) ID: OMIX009541"
