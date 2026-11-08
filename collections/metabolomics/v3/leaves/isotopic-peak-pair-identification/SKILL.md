---
name: isotopic-peak-pair-identification
description: Use when you have a preprocessed spatial metabolomics peak table (X/Y
  pixel coordinates plus m/z ion columns) and need to quantify what fraction of ions
  are isotopic peaks or flag ion pairs whose spatial intensity patterns co-vary.
when_to_use_negative:
- The input is raw MS imaging data (e.g., IMZML/mzML) rather than a preprocessed peak
  table matrix with pixel coordinates and m/z columns
- Noisy ions and background pixels have not yet been removed, since spurious correlations
  and mass offsets will inflate the isotopic pair count
- The goal is adduct ion identification rather than isotopologue detection — SMQVP
  provides a separate Adduct Ion Ratio module for adduct pairs
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMQVP
  role: Web application whose Isotopic Peak Ratio quality visualization module performs
    isotopic peak identification and evaluates spatial distribution consistency
  repo: github.com/mzlab-research/SMQVP
- name: MetaboCoreUtils
  role: R package whose isotopologues function scans for candidate isotopic peak pairs
    within a ppm mass-error tolerance of the theoretical offset
- name: R
  role: Runtime environment (with Shiny) in which SMQVP and MetaboCoreUtils are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:ad326a2a2a0349f3f7f9dbd669d2b3a765ab31fb179c2d709a30f633fed0544f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboCoreUtils
  - R
  - SMQVP
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# isotopic-peak-pair-identification

## Summary

Identifies isotopic peak pairs (e.g., M+1 offsets of ~1.0035 Da) among ions in a spatial metabolomics peak table, as implemented in SMQVP's Isotopic Peak Ratio module via the MetaboCoreUtils isotopologues approach, to evaluate the reliability of ion detection.

## When to use

Use when you have a preprocessed spatial metabolomics peak table (X/Y pixel coordinates plus m/z ion columns) and need to quantify what fraction of ions are isotopic peaks or flag ion pairs whose spatial intensity patterns co-vary.

## When NOT to use

- The input is raw MS imaging data (e.g., IMZML/mzML) rather than a preprocessed peak table matrix with pixel coordinates and m/z columns
- Noisy ions and background pixels have not yet been removed, since spurious correlations and mass offsets will inflate the isotopic pair count
- The goal is adduct ion identification rather than isotopologue detection — SMQVP provides a separate Adduct Ion Ratio module for adduct pairs

## Inputs

- Preprocessed peak table matrix (first two columns: X and Y pixel coordinates; subsequent columns: m/z ion intensities per pixel) from AFADESI-MSI data, e.g., 53,812 pixels x 2,654 ions
- User-specified ppm error tolerance around the theoretical ~1.0035 Da M+1 mass offset
- User-defined Pearson correlation threshold for spatial intensity co-variation

## Outputs

- Set of isotopic peak pairs (indices of M and M+1 ions) passing ppm and correlation criteria
- Isotopic peak ratio (proportion of ions flagged as isotopic, e.g., ~14.44% in the AFADESI mouse brain dataset)
- Summary/visualization plot of the isotopic peak ratio comparable to Figure 2J

## How to apply

Load the preprocessed peak intensity table (X, Y coordinates plus m/z ion columns) after background pixel removal and noise ion filtering (e.g., noise score threshold 90). Apply the isotopologues function from the MetaboCoreUtils R package to scan for candidate isotope pairs whose measured mass difference deviates from the theoretical ~1.0035 Da M+1 offset by no more than a user-specified ppm error tolerance. For each candidate pair, compute the Pearson correlation of the two ions' spatial intensities across pixels and retain only pairs whose correlation exceeds the user-defined threshold as true isotopic peaks. Compute the proportion of ions flagged as isotopic and visualize the isotopic peak ratio; in the AFADESI mouse brain dataset this yielded 14.44% of ions identified as isotopic peaks, with high-intensity ions reliably showing matching isotopic peaks (Figure 2J).

## Related tools

- **SMQVP** (Web application whose Isotopic Peak Ratio quality visualization module performs isotopic peak identification and evaluates spatial distribution consistency) — github.com/mzlab-research/SMQVP
- **MetaboCoreUtils** (R package whose isotopologues function scans for candidate isotopic peak pairs within a ppm mass-error tolerance of the theoretical offset)
- **R** (Runtime environment (with Shiny) in which SMQVP and MetaboCoreUtils are executed)

## Examples

```
Rscript -e "library(MetaboCoreUtils); iso <- isotopologues(peak_intensities, ppm = 5)"
```

## Evaluation signals

- Computed proportion of isotopic ions falls in the vicinity of the reported 14.44% for the AFADESI mouse brain dataset
- Every retained pair has a measured mass difference within the specified ppm tolerance of the theoretical ~1.0035 Da M+1 offset
- Every retained pair has a Pearson spatial-intensity correlation exceeding the user-defined threshold
- High-intensity ions are predominantly the ones with matching isotopic peaks, as reported in Figure 2J
- Input schema check: table has exactly two leading pixel-coordinate columns followed by m/z ion intensity columns

## Limitations

- Requires preprocessed data: background pixels and noise ions must be removed first, or the isotopic ratio will be biased
- Accuracy depends on user-specified ppm tolerance and Pearson correlation threshold; no default values are documented (no software version or changelog available for the repository)
- The approach targets M+1 isotopologue offsets and may miss higher-order isotope peaks or overlapping isotope/adduct signals

## Evidence

- [readme] readme: "Isotopic Peak Identification: Identify isotopic peaks to evaluate the reliability of ion detection."
- [intro] intro: "through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions"
- [readme] readme-usage: "**Isotopic Peak Ratio**: Identify isotopic peaks and evaluate their spatial distribution consistency."
- [results] results: "SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality."
