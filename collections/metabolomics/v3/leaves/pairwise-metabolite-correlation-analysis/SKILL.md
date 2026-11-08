---
name: pairwise-metabolite-correlation-analysis
description: Use when you have a QC-filtered, per-cell anndata object of normalized
  isotopologue-derived relative uptake fractions and need to test whether uptake of
  individual fatty acids co-varies across single cells (e.g. control/normoxic liver
  cancer cells).
when_to_use_negative:
- Input is bulk (well-level or pixel-level) isotopologue data rather than per-cell
  anndata — the correlation structure reported here is a single-cell property hidden
  in bulk analyses
- Cells have not passed the published QC filtering (size, ion number, total detected
  ion intensity) — unfiltered cells distort the correlation structure
- The goal is to estimate absolute de novo synthesis vs uptake fractions — use the
  binomial model instead, which per the article was only applied to ACLYkd data because
  fitting is unreliable when overall substrate usage is very low
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: Scanpy
  role: Stores and reads the pickled anndata objects containing per-cell normalized
    isotopologue and relative uptake values; used to subset the control cell population
    and extract the feature table for correlation
  repo: https://github.com/Buglakova/13C-SpaceM
- name: 13C-SpaceM
  role: Upstream pipeline (SpaceM registration, extract_isotopologue_ion_images.py,
    sc_analysis.py) that produces the single-cell anndata objects this correlation
    analysis consumes
  repo: https://github.com/Buglakova/13C-SpaceM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:2cf199aa9e2ba3a0a9155652d0f883c980f47492e176a15df072d1fdcef8fa2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C-SpaceM
  tools:
  - 13C-SpaceM
  - Scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# pairwise-metabolite-correlation-analysis

## Summary

Computes the pairwise Pearson correlation matrix of per-cell relative uptake fractions for the five measured fatty acids (myristate, palmitate, palmitoleate, stearate, oleate) from the 13C-SpaceM single-cell anndata object, revealing which fatty acids co-vary in uptake across single control cells.

## When to use

Use when you have a QC-filtered, per-cell anndata object of normalized isotopologue-derived relative uptake fractions and need to test whether uptake of individual fatty acids co-varies across single cells (e.g. control/normoxic liver cancer cells).

## When NOT to use

- Input is bulk (well-level or pixel-level) isotopologue data rather than per-cell anndata — the correlation structure reported here is a single-cell property hidden in bulk analyses
- Cells have not passed the published QC filtering (size, ion number, total detected ion intensity) — unfiltered cells distort the correlation structure
- The goal is to estimate absolute de novo synthesis vs uptake fractions — use the binomial model instead, which per the article was only applied to ACLYkd data because fitting is unreliable when overall substrate usage is very low

## Inputs

- Per-cell anndata object (.h5ad / pickled anndata) with normalized isotopologue distributions per fatty acid per cell
- Cell metadata (obs) containing condition labels (normoxic vs hypoxic) and QC flags (cell size, ion number, total detected ion intensity)
- Relative uptake fraction columns for myristate, palmitate, palmitoleate, stearate and oleate

## Outputs

- 5x5 Pearson correlation matrix of single-cell relative uptake values across the five fatty acids
- Correlation matrix heatmap figure styled like the corresponding manuscript single-cell panel (Figures 2-4 style)

## How to apply

Load the shared per-cell anndata object (Scanpy pickled anndata, .h5ad/pkl format) and select only the control (normoxic) population after the published QC filtering by cell size, ion number and total detected ion intensity. Extract the per-cell relative uptake fraction for each of the five fatty acids from the normalized isotopologue data — these are uptake fractions computed without the binomial fit, since the binomial model is only applicable to the ACLYkd data. Compute the 5x5 pairwise Pearson correlation matrix across single cells and render it as a heatmap styled like the corresponding single-cell manuscript panel (Figures 2-4 are based on this single-cell data). The decision point is whether myristate behaves independently: the expected biological pattern is no myristate correlation with any other fatty acid, while palmitate, palmitoleate, stearate and oleate are strongly positively correlated with each other.

## Related tools

- **Scanpy** (Stores and reads the pickled anndata objects containing per-cell normalized isotopologue and relative uptake values; used to subset the control cell population and extract the feature table for correlation) — https://github.com/Buglakova/13C-SpaceM
- **13C-SpaceM** (Upstream pipeline (SpaceM registration, extract_isotopologue_ion_images.py, sc_analysis.py) that produces the single-cell anndata objects this correlation analysis consumes) — https://github.com/Buglakova/13C-SpaceM

## Examples

```
import scanpy as sc; ad = sc.read('hypoxia_normoxia_anndata.pkl'); ctrl = ad[ad.obs['condition'] == 'normoxic']; corr = ctrl.to_df()[['myristate','palmitate','palmitoleate','stearate','oleate']].corr(method='pearson')
```

## Evaluation signals

- Myristate relative uptake shows no (near-zero) Pearson correlation with any of palmitate, palmitoleate, stearate or oleate
- Palmitate, palmitoleate, stearate and oleate show strong positive pairwise Pearson correlations with each other
- The reproduced correlation pattern matches the corresponding single-cell manuscript panel (Figures 2-4)
- Only cells that passed the published QC (filtering by cell size, ion number and total detected ion intensity) and belong to the control/normoxic population are included
- Feature table values correspond to normalized isotopologue-derived relative uptake fractions, not raw ion intensities or binomial-fit uptake parameters

## Limitations

- 13C-SpaceM provides only relative, not absolute, substrate contribution to the lipogenic acetyl-CoA pool, so correlations describe relative uptake co-variation only
- MALDI-AIF lipid-class selectivity must be considered: fatty acids analysed derive from a subset of lipids ionized in negative mode (primarily acidic glycerophospholipids), so uptake proportions may differ by lipid class
- Environment reproducibility issue: combinations of pandas, anndata, pickle and scanpy versions can cause errors reading the pickled anndata objects; use environment_dump.yml to match versions

## Evidence

- [results] No correlation of the relative uptake of myristate with any of the other fatty acids; in contrast, uptake values for palmitate, palmitoleate, stearate and oleate showed strong positive correlations.: [evidence span withheld — anchor status: not_found]
- [methods] filtering out cells by size and number and total intensity of detected ions: "filtering out cells by size and number and total intensity of detected ions"
- [readme] This project uses Scanpy's pickled `anndata` objects to store the single-cell data.: "This project uses Scanpy's pickled `anndata` objects to store the single-cell data."
- [readme] Figures 2-4 contain plots based on single-cell data.: "Figures 2-4 contain plots based on single-cell data."
