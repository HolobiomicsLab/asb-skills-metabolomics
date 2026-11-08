---
name: spatial-intensity-pearson-correlation
description: Use when you have an AFADESI-style pixel-by-ion peak intensity table
  and need to confirm which near-1.0035-Da m/z pairs are true isotopic peaks based
  on spatial co-variation.
when_to_use_negative:
- Input is raw mass-spectrometry imaging data rather than a preprocessed peak table
  matrix — SMQVP expects first two columns to be X/Y coordinates and subsequent columns
  to be m/z ion intensities
- Data has not undergone background pixel removal and noise ion filtering, since spurious
  ions will corrupt the spatial correlation estimates
- The goal is adduct ion identification rather than isotopologue detection — SMQVP
  provides a separate adduct ion ratio module for that purpose
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: SMQVP
  role: Web application whose isotopic peak ratio module applies the spatial Pearson
    correlation filter to isotopologue candidate pairs
  repo: https://github.com/mzlab-research/SMQVP
- name: MetaboCoreUtils
  role: R package providing the isotopologues function used to generate candidate
    isotopic peak pairs before spatial correlation filtering
- name: R
  role: Runtime environment (with Shiny) in which SMQVP and MetaboCoreUtils are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:ee4e28ad737f3fb9dbfacc6899d6c1b4ee217dd14915b0d745196231c35f807f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - MetaboCoreUtils
  - R
  - SMQVP
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spatial-intensity-pearson-correlation

## Summary

Validates candidate isotopic peak pairs in spatial metabolomics peak tables by computing the Pearson correlation of two ions' intensities across pixels, retaining only spatially co-varying pairs. Used within SMQVP's isotopic peak ratio (QVP7) module to distinguish genuine isotopologues from mass-difference coincidences.

## When to use

Use when you have an AFADESI-style pixel-by-ion peak intensity table and need to confirm which near-1.0035-Da m/z pairs are true isotopic peaks based on spatial co-variation.

## When NOT to use

- Input is raw mass-spectrometry imaging data rather than a preprocessed peak table matrix — SMQVP expects first two columns to be X/Y coordinates and subsequent columns to be m/z ion intensities
- Data has not undergone background pixel removal and noise ion filtering, since spurious ions will corrupt the spatial correlation estimates
- The goal is adduct ion identification rather than isotopologue detection — SMQVP provides a separate adduct ion ratio module for that purpose

## Inputs

- Preprocessed peak intensity table (X, Y pixel coordinates plus one column per m/z ion) after background pixel removal and noise ion filtering (noise score threshold 90)
- m/z values of ion columns for isotopologue candidate scanning
- User-specified ppm error tolerance around the ~1.0035 Da M+1 offset
- User-defined Pearson correlation threshold for pair retention

## Outputs

- Set of validated isotopic peak pairs with their Pearson spatial correlation values
- Proportion of ions identified as isotopic peaks (QVP7 isotopic peak ratio, e.g. 14.44% for the AFADESI mouse brain dataset)
- Summary plot of the isotopic peak ratio comparable to Figure 2J

## How to apply

Load the preprocessed peak intensity matrix (rows = pixels with X, Y coordinates; columns = m/z ions) from the AFADESI mouse brain dataset after background pixel removal and noise ion filtering (noise score threshold 90). Apply the isotopologues function from the MetaboCoreUtils R package to scan for candidate M+1 pairs whose mass difference deviates from the theoretical ~1.0035 Da offset by no more than a user-specified ppm error tolerance. For each candidate pair, compute the Pearson correlation of the two ions' intensities across all pixels and retain pairs whose correlation exceeds the user-defined threshold as true isotopic peaks. The rationale is that genuine isotopologues of the same molecule must show coincident spatial distributions, whereas random m/z matches will not. Finally, compute the proportion of ions flagged as isotopic to obtain the isotopic peak ratio, comparable to Figure 2J of the SMQVP paper.

## Related tools

- **SMQVP** (Web application whose isotopic peak ratio module applies the spatial Pearson correlation filter to isotopologue candidate pairs) — https://github.com/mzlab-research/SMQVP
- **MetaboCoreUtils** (R package providing the isotopologues function used to generate candidate isotopic peak pairs before spatial correlation filtering)
- **R** (Runtime environment (with Shiny) in which SMQVP and MetaboCoreUtils are executed)

## Examples

```
library(MetaboCoreUtils); cand <- isotopologues(mz_vals, error = 5); ok <- cand[sapply(cand, function(p) cor(X[,p[1]], X[,p[2]])) > 0.8]
```

## Evaluation signals

- Computed isotopic ion percentage should fall in the vicinity of the reported 14.44% for the AFADESI mouse brain dataset (QVP7, Figure 2J)
- Retained pairs must simultaneously satisfy both filters: m/z difference within the specified ppm tolerance of ~1.0035 Da AND Pearson correlation above the threshold
- High-intensity ions should reliably show matching isotopic peaks, as observed in Figure 2J
- Correlation must be computed across all pixels of the filtered table, so the number of correlated pairs should not exceed the number of isotopologue candidates returned by MetaboCoreUtils
- Input table schema check: first two columns are X and Y pixel coordinates; remaining columns are m/z ion intensities (53,812 pixels and 2,654 ion peaks in the reference example)

## Limitations

- No software version or license information is specified for SMQVP in the article discussion
- The ppm tolerance and correlation threshold are user-defined, so results are sensitive to these parameters and not fixed by the method
- The reported rate was demonstrated only on a single AFADESI mouse brain dataset in positive ion mode (OMIX009541); transfer to other platforms or polarities is unvalidated

## Evidence

- [other] Apply the isotopologues function from the MetaboCoreUtils R package to scan for candidate isotopic peak pairs: [evidence span withheld — anchor status: not_found]
- [other] Pearson correlation retention step: [evidence span withheld — anchor status: not_found]
- [readme] SMQVP isotopic peak identification feature: "Isotopic Peak Identification: Identify isotopic peaks to evaluate the reliability of ion detection."
- [readme] Reference dataset description: "The example data used in this study is spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode."
- [abstract] Isotopic and adduct ion identification as quality visualization point: "identification of isotopic and adduct ions"
