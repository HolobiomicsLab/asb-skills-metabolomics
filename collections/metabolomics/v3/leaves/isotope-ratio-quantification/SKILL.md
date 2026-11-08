---
name: isotope-ratio-quantification
description: Use when a spatial metabolomics peak table (X/Y pixel coordinates plus
  m/z ion columns, e.g. AFADESI mouse brain data) needs an isotopic peak ratio quality
  metric, or when candidate M+1 isotopic pairs must be validated by spatial intensity
  correlation.
when_to_use_negative:
- The input is raw mass spectra (e.g. mzML/imzML) rather than a preprocessed peak
  table matrix with pixel coordinates — SMQVP requires the peak table format
- The goal is identifying adduct ion pairs rather than isotopic peaks — use SMQVP's
  separate Adduct Ion Ratio module
- Background pixels and noise ions have not been removed yet — the ratio should be
  computed after background noise removal and noise ion filtering (noise score threshold
  90) to avoid distorted proportions
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SMQVP
  role: Web application whose Isotopic Peak Ratio quality visualization point identifies
    isotopic peaks and evaluates their spatial distribution consistency
  repo: github.com/mzlab-research/SMQVP
- name: MetaboCoreUtils
  role: R package providing the isotopologues function used to scan for candidate
    isotopic peak pairs within the ppm mass-difference tolerance
- name: R
  role: Runtime environment (with Shiny) in which SMQVP and MetaboCoreUtils are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:b3ea680dcbce7c2fd09c13873f5b114317b1421608c101459f86c5fbc0a63593
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboCoreUtils
  - R
  - SMQVP
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# isotope-ratio-quantification

## Summary

Quantifies the proportion of ions in a spatial metabolomics peak table that are flagged as isotopic peaks, using the MetaboCoreUtils isotopologues approach inside SMQVP's Isotopic Peak Ratio module. It serves as a quality-visualization point to gauge ion-detection reliability in AFADESI-MSI data.

## When to use

Use when a spatial metabolomics peak table (X/Y pixel coordinates plus m/z ion columns, e.g. AFADESI mouse brain data) needs an isotopic peak ratio quality metric, or when candidate M+1 isotopic pairs must be validated by spatial intensity correlation.

## When NOT to use

- The input is raw mass spectra (e.g. mzML/imzML) rather than a preprocessed peak table matrix with pixel coordinates — SMQVP requires the peak table format
- The goal is identifying adduct ion pairs rather than isotopic peaks — use SMQVP's separate Adduct Ion Ratio module
- Background pixels and noise ions have not been removed yet — the ratio should be computed after background noise removal and noise ion filtering (noise score threshold 90) to avoid distorted proportions

## Inputs

- Preprocessed peak table matrix with first two columns as X and Y pixel coordinates and subsequent columns as m/z ion intensities per pixel
- User-specified ppm error tolerance around the theoretical ~1.0035 Da M+1 mass offset
- User-defined Pearson correlation threshold for spatial intensity concordance of candidate isotopic pairs

## Outputs

- Set of validated isotopic peak pairs (ion m/z pairs passing ppm and correlation criteria)
- Isotopic peak ratio (proportion of ions identified as isotopic peaks; 14.44% for the AFADESI mouse brain dataset)
- Summary plot of the isotopic peak ratio, comparable to Figure 2J

## How to apply

Load the preprocessed peak intensity table — pixels as rows (X, Y coordinates) and m/z ion columns — after background pixel removal and noise ion filtering with a noise score threshold of 90, so the ratio is computed on high-quality ions only. Apply the isotopologues function from the MetaboCoreUtils R package to scan for candidate isotopic peak pairs whose measured mass difference deviates from the theoretical ~1.0035 Da M+1 offset by no more than a user-specified ppm error tolerance. For each candidate pair, compute the Pearson correlation of the two ions' spatial intensities across pixels and retain pairs whose correlation exceeds the user-defined threshold as true isotopic peaks. Finally, compute the proportion of ions flagged as isotopic and generate a summary plot of the isotopic peak ratio (comparable to Figure 2J of the SMQVP paper). In the AFADESI mouse brain dataset (53,812 pixels, 2,654 ion peaks), high-intensity ions reliably show matching isotopic peaks, validating the metric.

## Related tools

- **SMQVP** (Web application whose Isotopic Peak Ratio quality visualization point identifies isotopic peaks and evaluates their spatial distribution consistency) — github.com/mzlab-research/SMQVP
- **MetaboCoreUtils** (R package providing the isotopologues function used to scan for candidate isotopic peak pairs within the ppm mass-difference tolerance)
- **R** (Runtime environment (with Shiny) in which SMQVP and MetaboCoreUtils are executed)

## Examples

```
Rscript -e 'library(MetaboCoreUtils); iso_pairs <- isotopologues(peak_table_mz, ppm = 5)'
```

## Evaluation signals

- The computed isotopic ion percentage should fall in the vicinity of the reported 14.44% of ions for the AFADESI mouse brain dataset
- Every retained isotopic pair must satisfy both criteria: measured mass difference within the user-specified ppm tolerance of the theoretical ~1.0035 Da M+1 offset, and Pearson spatial correlation above the user-defined threshold
- High-intensity ions should reliably show matching isotopic peaks, as reported in Figure 2J
- The output summary plot should be structurally comparable to Figure 2J (isotopic peak ratio visualization)
- Input schema check: peak table must have exactly two leading coordinate columns (X, Y) followed by m/z intensity columns, matching SMQVP's required format

## Limitations

- Isotopic identification depends on user-specified ppm tolerance and Pearson correlation threshold; inappropriate settings will miss true M+1 pairs or retain false ones
- No software version is specified for SMQVP in the article or README, complicating exact reproduction
- Results are demonstrated only on AFADESI-based mouse brain data in positive ion mode; behavior on other acquisition platforms or ion modes is not established

## Evidence

- [intro] intro usage of SMQVP module: "Isotopic Peak Ratio: Identify isotopic peaks and evaluate their spatial distribution consistency."
- [intro] scope of eight quality visualization points: "through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions"
- [readme] README feature description: "**Isotopic Peak Identification**: Identify isotopic peaks to evaluate the reliability of ion detection."
- [readme] README example dataset scale: "The data includes 53,812 pixels and 2,654 ion peaks."
