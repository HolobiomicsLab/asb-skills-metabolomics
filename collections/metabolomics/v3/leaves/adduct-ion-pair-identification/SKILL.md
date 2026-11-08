---
name: adduct-ion-pair-identification
description: Use when a preprocessed spatial metabolomics peak intensity matrix (pixels
  x m/z ions) from MALDI/DESI-MSI is available and you need to flag which ions are
  adduct forms and quantify the adduct distribution.
when_to_use_negative:
- Input is raw mass-spectrometry imaging data rather than a preprocessed peak table
  matrix with X/Y coordinates and m/z intensity columns
- The goal is isotopic peak identification or noise ion filtering — these are separate
  SMQVP quality visualization points and should use their own procedures
- Data were acquired in negative ion mode without the corresponding negative-mode
  adduct differences defined, since the module targets seven predefined positive-ion-mode
  adduct types
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMQVP
  role: Web application providing the adduct ion ratio quality visualization point
    (QVP8) that searches for and identifies adduct ion pairs in the uploaded peak
    table
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Runtime environment (with Shiny) in which SMQVP v1.0 is implemented and can
    be run locally
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:6600760f2f7d9e79997a90a59cfe408671d6527c986b80222807d973254577bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  - SMQVP v1.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8260869565217391
n_claims: 23
---

# adduct-ion-pair-identification

## Summary

Identify adduct ions (e.g., M + H, M + K, M + Na) in spatial metabolomics peak tables by searching for predefined positive-ion-mode adduct mass differences among m/z peaks and confirming candidate pairs via spatial Pearson correlation, as implemented in SMQVP's adduct ion ratio module (QVP8).

## When to use

Use when a preprocessed spatial metabolomics peak intensity matrix (pixels x m/z ions) from MALDI/DESI-MSI is available and you need to flag which ions are adduct forms and quantify the adduct distribution.

## When NOT to use

- Input is raw mass-spectrometry imaging data rather than a preprocessed peak table matrix with X/Y coordinates and m/z intensity columns
- The goal is isotopic peak identification or noise ion filtering — these are separate SMQVP quality visualization points and should use their own procedures
- Data were acquired in negative ion mode without the corresponding negative-mode adduct differences defined, since the module targets seven predefined positive-ion-mode adduct types

## Inputs

- Preprocessed peak table matrix (pixels as rows, m/z ion intensities as columns) derived from AFADESI-based mouse brain data (e.g., OMIX009541)
- List of predefined positive-ion-mode adduct mass differences (seven common types, e.g., M + H, M + K, M + Na)
- User-defined spatial correlation threshold for retaining adduct pairs

## Outputs

- Proportion of filtered ions identified as adduct ions (expected ~15.88% for the AFADESI mouse brain dataset)
- Per-adduct-type counts/ratios (e.g., adduct_distribution.csv with M + H most prevalent, then M + K and M + Na)
- Identified adduct ion pairs with their spatial Pearson correlation values

## How to apply

Load the peak intensity matrix (pixels as rows, m/z values as columns; SMQVP's expected input has X/Y pixel coordinates in the first two columns) using pandas. Compute all pairwise m/z mass differences and match pairs whose difference corresponds to the seven predefined common positive-ion-mode adduct types used by SMQVP's adduct ion ratio module (QVP8), e.g., M + H, M + K, M + Na. For each candidate adduct pair, compute the Pearson correlation of the two ions' spatial intensity profiles across pixels and retain only pairs exceeding the user-defined correlation threshold, since true adducts must co-localize spatially. Finally, report the proportion of filtered ions assigned as adduct ions and tabulate counts per adduct type. On the AFADESI mouse brain dataset (53,812 pixels, 2,654 ion peaks, OMIX009541), this procedure identified 15.88% of filtered ions as adduct ions with M + H most prevalent, followed by M + K and M + Na.

## Related tools

- **SMQVP** (Web application providing the adduct ion ratio quality visualization point (QVP8) that searches for and identifies adduct ion pairs in the uploaded peak table) — https://github.com/mzlab-research/SMQVP
- **R** (Runtime environment (with Shiny) in which SMQVP v1.0 is implemented and can be run locally)

## Examples

```
import pandas as pd; df = pd.read_csv('peak_table.csv'); diffs = pd.DataFrame([[abs(a-b) for b in df.columns[2:]] for a in df.columns[2:]]); pairs = diffs[diffs.isin(adduct_masses)]  # then Pearson-correlate spatial profiles per candidate pair
```

## Evaluation signals

- Adduct identification rate is close to the reported 15.88% of filtered ions on the AFADESI mouse brain dataset
- M + H is the most prevalent adduct, followed by significant proportions of M + K and M + Na
- Every retained adduct pair exceeds the user-defined spatial Pearson correlation threshold across pixels
- Output table (e.g., adduct_distribution.csv) covers only the seven predefined adduct mass-difference types
- Input matrix conforms to SMQVP's expected format: first two columns are X and Y pixel coordinates, subsequent columns are m/z intensities

## Limitations

- Identification depends on the seven predefined positive-ion-mode adduct types; adducts outside this set will not be detected
- No software version beyond SMQVP v1.0, no changelog, and no license information are documented in the repository
- Retention of candidate pairs depends on a user-defined correlation threshold, so results are sensitive to that parameter choice

## Evidence

- [readme] Adduct Ion Ratio: Search for and identify adduct ion pairs.: "Adduct Ion Ratio: Search for and identify adduct ion pairs."
- [intro] eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions: "through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions"
- [results] SMQVP's QVP8 module identified adduct ions for 15.88% of the filtered ions, with M + H as the most prevalent adduct: [evidence span withheld — anchor status: not_found]
- [readme] The example data used in this study is spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode. The data includes 53,812 pixels and 2,654 ion peaks.: "The example data used in this study is spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode. The data includes 53,812 pixels and 2,654 ion peaks."
- [readme] First two columns: X and Y coordinates of the pixels; Subsequent columns: m/z values with corresponding intensities at each pixel: "First two columns: X and Y coordinates of the pixels
- Subsequent columns: m/z values with corresponding intensities at each pixel"
