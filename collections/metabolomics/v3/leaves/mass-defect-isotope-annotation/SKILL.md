---
name: mass-defect-isotope-annotation
description: Use when a preprocessed spatial metabolomics peak table (X, Y coordinates
  plus m/z ion columns) needs isotope annotation to assess ion-detection reliability,
  e.g. when computing the isotopic peak ratio quality metric.
when_to_use_negative:
- Input is raw instrument data rather than a preprocessed peak table matrix (X, Y
  coordinates plus m/z columns) — SMQVP expects the peak table format
- 'Data has not been denoised first: background pixels should be removed and noise
  ions filtered (noise score threshold 90) before isotope scanning, or spurious pairs
  will inflate the ratio'
- The goal is adduct ion identification rather than isotopologue detection — SMQVP
  provides a separate adduct ion ratio module
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMQVP
  role: Web application whose isotopic peak ratio quality visualization point applies
    the MetaboCoreUtils isotopologues approach to the uploaded peak table
  repo: https://github.com/mzlab-research/SMQVP
- name: MetaboCoreUtils
  role: R package providing the isotopologues function used to scan candidate isotope
    pairs within a ppm tolerance of the ~1.0035 Da M+1 offset
- name: R
  role: Runtime environment (with Shiny) in which SMQVP and the isotopologues-based
    isotope scan are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:17f6b34ce71ca3ba7aac5a58239193bb17d44b059b2f7139347c3c0dc39f3f0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - MetaboCoreUtils
  - R
  - SMQVP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mass-defect-isotope-annotation

## Summary

Identify isotopic peak pairs in spatial metabolomics peak tables by scanning for mass differences near the theoretical ~1.0035 Da M+1 offset (MetaboCoreUtils isotopologues approach) and confirming spatial co-localization via Pearson correlation. In the SMQVP workflow this flags the isotopic peak ratio (14.44% of AFADESI mouse brain ions) as a quality visualization point.

## When to use

Use when a preprocessed spatial metabolomics peak table (X, Y coordinates plus m/z ion columns) needs isotope annotation to assess ion-detection reliability, e.g. when computing the isotopic peak ratio quality metric.

## When NOT to use

- Input is raw instrument data rather than a preprocessed peak table matrix (X, Y coordinates plus m/z columns) — SMQVP expects the peak table format
- Data has not been denoised first: background pixels should be removed and noise ions filtered (noise score threshold 90) before isotope scanning, or spurious pairs will inflate the ratio
- The goal is adduct ion identification rather than isotopologue detection — SMQVP provides a separate adduct ion ratio module

## Inputs

- Preprocessed peak table matrix with first two columns as X and Y pixel coordinates and subsequent columns as m/z ion intensities at each pixel
- User-specified ppm error tolerance for the ~1.0035 Da M+1 mass offset
- User-defined Pearson correlation threshold for spatial intensity agreement of candidate isotope pairs

## Outputs

- Set of ions flagged as isotopic peaks
- Isotopic peak ratio (proportion of ions identified as isotopic, e.g. 14.44% for the AFADESI mouse brain dataset)
- Summary plot of the isotopic peak ratio comparable to Figure 2J

## How to apply

Load the preprocessed peak intensity table from the AFADESI mouse brain dataset after background pixel removal and noise ion filtering (noise score threshold 90). Apply the isotopologues function from the MetaboCoreUtils R package to scan for candidate isotopic peak pairs whose measured mass difference deviates from the theoretical ~1.0035 Da M+1 offset by no more than a user-specified ppm error tolerance. For each candidate pair, compute the Pearson correlation of the two ions' spatial intensities across pixels and retain pairs whose correlation exceeds the user-defined threshold as true isotopic peaks. Finally, compute the proportion of ions flagged as isotopic and generate a summary plot comparable to Figure 2J; the rationale is that high-intensity ions reliably show matching isotopic peaks, so the ratio serves as a quality indicator.

## Related tools

- **SMQVP** (Web application whose isotopic peak ratio quality visualization point applies the MetaboCoreUtils isotopologues approach to the uploaded peak table) — https://github.com/mzlab-research/SMQVP
- **MetaboCoreUtils** (R package providing the isotopologues function used to scan candidate isotope pairs within a ppm tolerance of the ~1.0035 Da M+1 offset)
- **R** (Runtime environment (with Shiny) in which SMQVP and the isotopologues-based isotope scan are executed)

## Examples

```
Rscript -e 'library(MetaboCoreUtils); iso <- isotopologues(peaks$`m/z`, ppm = 5)'
```

## Evaluation signals

- Computed isotopic ion percentage falls in the vicinity of the reported 14.44% of ions for the AFADESI mouse brain dataset
- Retained isotope pairs all show mass differences within the user-specified ppm tolerance of the theoretical ~1.0035 Da M+1 offset
- All retained pairs have Pearson spatial-intensity correlations above the user-defined threshold across pixels
- High-intensity ions reliably show matching isotopic peaks, consistent with Figure 2J

## Limitations

- The ppm tolerance and Pearson correlation threshold are user-defined, so results (and the isotopic ratio) vary with these settings
- The reported 14.44% benchmark is specific to the AFADESI mouse brain dataset after background pixel removal and noise ion filtering (noise score threshold 90); other datasets will differ
- No software version or changelog is specified for SMQVP, and no license information is stated

## Evidence

- [readme] Isotopic Peak Identification: Identify isotopic peaks to evaluate the reliability of ion detection.: "Isotopic Peak Identification: Identify isotopic peaks to evaluate the reliability of ion detection."
- [other] Apply the isotopologues function from the MetaboCoreUtils R package to scan for candidate isotopic peak pairs whose measured mass difference deviates from the theoretical ~1.0035 Da M+1 offset by no more than a user-specified ppm error tolerance.: [evidence span withheld — anchor status: not_found]
- [other] SMQVP identified isotopic peaks in 14.44% of the ions in the AFADESI mouse brain dataset, with high-intensity ions reliably showing matching isotopic peaks (Figure 2J).: [evidence span withheld — anchor status: not_found]
- [intro] through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions: "through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions"
