---
name: instrument-stability-quality-assessment-msi
description: Use when a spatial metabolomics peak table (pixels × m/z ion intensities)
  must be checked for stable acquisition before downstream analysis, by testing whether
  manually or lasso-delineated off-tissue background regions yield mutually consistent
  spectra.
when_to_use_negative:
- The goal is assessing tissue-region data quality (e.g. noise ion filtering, isotopic/adduct
  identification) rather than off-tissue acquisition stability — use the corresponding
  SMQVP module instead
- No background/off-tissue regions can be delineated in the dataset, so no acquisition-stability
  reference exists
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SMQVP
  role: Web application used to upload the peak table, delineate background regions
    with the lasso tool, and run the background-consistency quality visualization
    point
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Environment (with Shiny/SMQVP) for loading the peak table, computing per-region
    median spectra and the pairwise Pearson correlation matrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:6e1074f422a4c9a0f25c5f59c51f5ee3ce69c664fb5012c86207f94a73b587b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  - SMQVP v1.0
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# instrument-stability-quality-assessment-msi

## Summary

Assess acquisition stability in spatial metabolomics by computing pairwise Pearson correlations between the spectra of delineated background (off-tissue) regions, using SMQVP's background-consistency quality visualization point; correlations above 0.9 indicate consistent, stable signal acquisition.

## When to use

Use when a spatial metabolomics peak table (pixels × m/z ion intensities) must be checked for stable acquisition before downstream analysis, by testing whether manually or lasso-delineated off-tissue background regions yield mutually consistent spectra.

## When NOT to use

- The goal is assessing tissue-region data quality (e.g. noise ion filtering, isotopic/adduct identification) rather than off-tissue acquisition stability — use the corresponding SMQVP module instead
- No background/off-tissue regions can be delineated in the dataset, so no acquisition-stability reference exists

## Inputs

- Spatial metabolomics peak table matrix (first two columns X/Y pixel coordinates; remaining columns m/z ion intensities per pixel), e.g. AFADESI mouse brain data: 53,812 pixels × 2,654 ion peaks
- Pixel-coordinate map for delineating background regions on the interaction image

## Outputs

- Pairwise Pearson correlation matrix of background-region spectra (e.g. background_correlations.csv)
- Background-consistency QC verdict: pass when all pairwise coefficients exceed the 0.9 QVP2 threshold (paper reports >0.99 for the AFADESI mouse brain dataset)

## How to apply

Load the peak table (e.g., the 53,812-pixel × 2,654-ion AFADESI mouse brain table from OMIX009541) into SMQVP or R, with the first two columns holding X/Y pixel coordinates and remaining columns m/z intensities. Use the lasso tool on the interaction image to delineate background regions outside the tissue, mirroring the paper's manually delineated tissue and background regions. For each background region, aggregate ion intensities across its pixels (e.g., per-ion median) to build a representative spectrum. Compute pairwise Pearson correlation coefficients between all background-region spectra and apply the SMQVP QVP2 criterion that a coefficient exceeding 0.9 indicates consistent, stable acquisition. Rationale: off-tissue regions should reflect instrument signal stability alone, so high inter-region correlation isolates acquisition drift from biological variation.

## Related tools

- **SMQVP** (Web application used to upload the peak table, delineate background regions with the lasso tool, and run the background-consistency quality visualization point) — https://github.com/mzlab-research/SMQVP
- **R** (Environment (with Shiny/SMQVP) for loading the peak table, computing per-region median spectra and the pairwise Pearson correlation matrix)

## Examples

```
Rscript -e 'pt <- read.csv("afadesi_brain_peaks.csv"); meds <- sapply(bg_region_list, function(idx) apply(pt[idx, -c(1,2)], 2, median)); cm <- cor(meds); write.csv(cm, "background_correlations.csv")'
```

## Evaluation signals

- All pairwise Pearson correlations between background-region spectra exceed the 0.9 QVP2 threshold
- On the AFADESI mouse brain dataset (OMIX009541), all background-region correlations exceed 0.99, matching the paper's reported result
- Output correlation matrix is square and symmetric with dimension equal to the number of delineated background regions and unit diagonal
- Delineated background regions contain only low-intensity pixels outside the tissue, mirroring the paper's five manually delineated tissue and background regions

## Limitations

- Requires user-defined (lasso-delineated) background regions; results depend on where the user draws regions
- No software version or changelog is specified for SMQVP in the repository, complicating reproducibility
- No license information is stated in the repository documentation

## Evidence

- [readme] background consistency evaluation: "It provides a comprehensive solution for researchers to evaluate and enhance the reliability of their data through eight quality visualization points, including background consistency evaluation"
- [intro] workflow and data source: "This peak table and raw data are available in the public repository https://ngdc.cncb.ac.cn/omix/release/OMIX009541"
- [intro] lasso region delineation: "Use the lasso tool to delineate background and tissue regions on the interaction image"
- [methods] QVP2 threshold from source card: [evidence span withheld — anchor status: not_found]
- [results] reported result: [evidence span withheld — anchor status: not_found]
