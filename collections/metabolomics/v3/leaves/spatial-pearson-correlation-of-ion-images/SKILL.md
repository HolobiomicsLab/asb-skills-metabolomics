---
name: spatial-pearson-correlation-of-ion-images
description: Use when a peak table from spatial metabolomics data (pixels x m/z) contains
  mass-difference pairs matching known adducts (M+H, M+K, M+Na) and you must confirm
  which pairs co-localize spatially before assigning adduct annotations.
when_to_use_negative:
- Input is raw MS imaging data (e.g., imzML) rather than a preprocessed peak table
  matrix — SMQVP requires the peak table as input
- You need to distinguish isotopic peaks rather than adduct pairs — SMQVP provides
  a separate isotopic peak ratio module for that task
- Peaks lack meaningful spatial co-structure (e.g., merged or pooled samples with
  no pixel-level coordinates), making spatial correlation uninterpretable
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: SMQVP
  role: Web application whose adduct ion ratio module (QVP8) implements adduct pair
    search with spatial correlation on the uploaded peak table
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Runtime environment (with Shiny) in which SMQVP is implemented and can be
    run locally for the adduct identification workflow
  repo: https://github.com/mzlab-research/SMQVP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:d79efafa95ef7bbcc09c7e88e9f67eb48dcca54f0c33167cf331c7b5ab28aef2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  - SMQVP v1.0
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# spatial-pearson-correlation-of-ion-images

## Summary

Computes Pearson correlation between the spatial intensity profiles (ion images) of pairs of m/z peaks to confirm candidate adduct relationships, as implemented in SMQVP's adduct ion ratio module (QVP8) on AFADESI mouse brain data.

## When to use

Use when a peak table from spatial metabolomics data (pixels x m/z) contains mass-difference pairs matching known adducts (M+H, M+K, M+Na) and you must confirm which pairs co-localize spatially before assigning adduct annotations.

## When NOT to use

- Input is raw MS imaging data (e.g., imzML) rather than a preprocessed peak table matrix — SMQVP requires the peak table as input
- You need to distinguish isotopic peaks rather than adduct pairs — SMQVP provides a separate isotopic peak ratio module for that task
- Peaks lack meaningful spatial co-structure (e.g., merged or pooled samples with no pixel-level coordinates), making spatial correlation uninterpretable

## Inputs

- Preprocessed peak table matrix (pixels as rows, m/z ion intensity columns; SMQVP format with X/Y coordinate columns first)
- List of predefined positive-ion-mode adduct mass differences (e.g., M+H, M+K, M+Na; seven types used by SMQVP QVP8)
- User-defined Pearson correlation threshold for retaining adduct pairs

## Outputs

- Set of confirmed adduct ion pairs (mass-difference match plus spatially correlated ion images)
- Adduct distribution summary table (counts/proportion per adduct type, e.g., adduct_distribution.csv)
- Proportion of filtered ions identified as adduct ions (e.g., 15.88%)

## How to apply

Load the preprocessed peak intensity matrix (pixels as rows, m/z ions as columns; e.g., the AFADESI positive-ion-mode mouse brain table with 53,812 pixels and 2,654 ion peaks from OMIX009541) with pandas or R. Compute all pairwise m/z mass differences and retain pairs matching the seven predefined positive-ion-mode adduct types used by SMQVP's QVP8 module (e.g., M+H, M+K, M+Na). For each candidate pair, extract the two ions' spatial intensity profiles across pixels and calculate the Pearson correlation; keep only pairs exceeding the user-defined correlation threshold, mirroring the paper's QVP8 procedure. Finally, count distinct ions assigned as adduct partners and tabulate counts per adduct type — in the reported run, 15.88% of filtered ions were identified as adduct ions, with M+H most prevalent, followed by M+K and M+Na.

## Related tools

- **SMQVP** (Web application whose adduct ion ratio module (QVP8) implements adduct pair search with spatial correlation on the uploaded peak table) — https://github.com/mzlab-research/SMQVP
- **R** (Runtime environment (with Shiny) in which SMQVP is implemented and can be run locally for the adduct identification workflow) — https://github.com/mzlab-research/SMQVP

## Examples

```
import pandas as pd; df = pd.read_csv('peak_table.csv'); corr = df[mz1].corr(df[mz2])  # Pearson r between two candidate adduct ion images; keep if r > threshold
```

## Evaluation signals

- Adduct identification rate should be close to the reported 15.88% of filtered ions on the AFADESI mouse brain dataset
- M+H should be the most prevalent adduct, followed by significant proportions of M+K and M+Na in the adduct_distribution.csv table
- Every retained ion pair must satisfy both criteria: mass difference matching one of the seven predefined adduct types AND Pearson correlation above the user-defined threshold
- Identified adduct partners should show visually overlapping ion images across tissue pixels when plotted on the pixel grid

## Limitations

- Correlation threshold is user-defined; results (and the adduct rate) are sensitive to this choice
- Mass-difference matching alone cannot distinguish adducts from other small-mass-offset relationships (e.g., isotopes or neutral losses) — spatial correlation is an additional but imperfect filter
- No software version or changelog specified for SMQVP, complicating exact reproduction

## Evidence

- [readme] Adduct Ion Ratio module: "Adduct Ion Ratio: Search for and identify adduct ion pairs."
- [readme] eight quality visualization points: "eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions"
- [readme] example dataset scale: "The data includes 53,812 pixels and 2,654 ion peaks. This peak table and raw data are available in the public repository https://ngdc.cncb.ac.cn/omix/release/OMIX009541"
- [intro] module scope: "through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions"
