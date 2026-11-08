---
name: mass-difference-matching-against-adduct-list
description: Use when a positive-ion-mode spatial metabolomics peak table (pixels
  × m/z ions) needs adduct annotation, e.g., quantifying what proportion of filtered
  ions are M+H, M+K, or M+Na adducts of the same metabolite.
when_to_use_negative:
- The peak table is negative-ion-mode data — the predefined adduct list (M+H, M+K,
  M+Na) covers common positive-ion-mode adducts only
- The input is raw imaging data rather than a peak table matrix; SMQVP requires a
  peak table with X/Y coordinates and m/z intensity columns
- Only a single peak must be annotated (no partner ion present in the matrix), since
  the method relies on detecting mass-difference pairs between ions
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMQVP v1.0
  role: Provides the adduct ion ratio module (QVP8) that searches for and identifies
    adduct ion pairs based on predefined adduct mass differences and spatial correlation
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Runtime environment for SMQVP (a Shiny web application); R with pandas-equivalent
    data handling can also be used to replicate the pairwise mass-difference and Pearson
    correlation computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:3c9406c5a1f061d9e50a21708396f1056524cf5a3588bf11eb52ae20379d2e8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP v1.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Mass-difference matching against an adduct list

## Summary

Identifies adduct ions in positive-ion-mode spatial metabolomics peak tables by computing pairwise m/z mass differences, matching them against a predefined adduct list (M+H, M+K, M+Na, etc.), and confirming candidates via spatial Pearson correlation, as implemented in SMQVP's adduct ion ratio module (QVP8).

## When to use

Use when a positive-ion-mode spatial metabolomics peak table (pixels × m/z ions) needs adduct annotation, e.g., quantifying what proportion of filtered ions are M+H, M+K, or M+Na adducts of the same metabolite.

## When NOT to use

- The peak table is negative-ion-mode data — the predefined adduct list (M+H, M+K, M+Na) covers common positive-ion-mode adducts only
- The input is raw imaging data rather than a peak table matrix; SMQVP requires a peak table with X/Y coordinates and m/z intensity columns
- Only a single peak must be annotated (no partner ion present in the matrix), since the method relies on detecting mass-difference pairs between ions

## Inputs

- Preprocessed peak intensity matrix (pixels as rows, m/z ions as columns) from AFADESI mouse brain data (OMIX009541), with X/Y pixel coordinates in the first two columns
- List of predefined positive-ion-mode adduct mass differences (seven common types, e.g., M+H, M+K, M+Na)
- User-defined spatial Pearson correlation threshold for retaining adduct pairs

## Outputs

- Set of identified adduct ion pairs (m/z pair, adduct type, Pearson correlation)
- Proportion of filtered ions identified as adduct ions (e.g., ~15.88% in the reference run)
- Per-adduct-type count/summary table (e.g., adduct_distribution.csv)

## How to apply

Load the preprocessed peak intensity matrix from the AFADESI mouse brain dataset (OMIX009541) with pixels as rows and m/z ions as columns. Compute all pairwise m/z mass differences among ion peaks and match pairs whose differences correspond to the seven predefined common positive-ion-mode adduct types used by SMQVP's QVP8 module. For each candidate adduct pair, calculate the Pearson correlation of the two ions' spatial intensity profiles across pixels and retain only pairs exceeding the user-defined correlation threshold, which guards against spurious mass coincidences. Finally, compute the proportion of filtered ions assigned as adduct ions and tabulate counts per adduct type (e.g., M+H, M+K, M+Na) into a summary table. This follows SMQVP's rationale that adduct pairs should share both a characteristic mass offset and a consistent spatial distribution.

## Related tools

- **SMQVP v1.0** (Provides the adduct ion ratio module (QVP8) that searches for and identifies adduct ion pairs based on predefined adduct mass differences and spatial correlation) — https://github.com/mzlab-research/SMQVP
- **R** (Runtime environment for SMQVP (a Shiny web application); R with pandas-equivalent data handling can also be used to replicate the pairwise mass-difference and Pearson correlation computation)

## Examples

```
import pandas as pd, numpy as np; df = pd.read_csv('peak_table.csv'); r = np.corrcoef(df['mz1'], df['mz2'])[0,1]  # retain pair if r > threshold
```

## Evaluation signals

- Adduct identification rate of filtered ions is close to the reported 15.88% on the AFADESI mouse brain dataset
- M+H is the most prevalent adduct type, followed by significant proportions of M+K and M+Na
- Every retained ion pair has a mass difference matching one of the seven predefined adduct masses
- Every retained ion pair exceeds the user-defined spatial Pearson correlation threshold across pixels
- Summary table (adduct_distribution.csv) reconciles: total assigned adduct ions / total filtered ions equals the reported proportion

## Limitations

- The adduct list covers only seven predefined common positive-ion-mode adduct types, so less common adducts will be missed
- The result depends on prior noise-ion filtering; ions not passing SMQVP's noise filter change the denominator of the adduct ratio
- The retained-pair decision hinges on a user-defined correlation threshold; no fixed default is documented in the article or README

## Evidence

- [readme] Adduct Ion Ratio: Search for and identify adduct ion pairs.: "Adduct Ion Ratio: Search for and identify adduct ion pairs."
- [readme] Adduct Ion Identification: Accurately identify adduct forms of ions for subsequent identification processes.: "Adduct Ion Identification: Accurately identify adduct forms of ions for subsequent identification processes."
- [results] SMQVP's QVP8 module identified adduct ions for 15.88% of the filtered ions, with M + H as the most prevalent adduct: [evidence span withheld — anchor status: not_found]
- [intro] eight quality visualization points, including ... identification of isotopic and adduct ions: "through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions"
