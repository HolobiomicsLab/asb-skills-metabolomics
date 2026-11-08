---
name: single-cell-metabolomics-matrix-preparation
description: Use when a raw single-cell metabolomics dataset (e.g. METASPACE annotations
  or an .rds single-cell matrix) must be reshaped into a metabolites-by-cells input
  matrix with condition labels before running S2IsoMEr ORA/MSEA.
when_to_use_negative:
- The input is already a metabolites-by-cells matrix with condition labels and FDR-filtered
  annotations
- The data are bulk (non-single-cell) metabolomics measurements without per-cell resolution
- The goal is enrichment over lipid-specific ontologies requiring LipidMaps/SwissLipids
  identifiers not present in the HMDB-annotated matrix
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: S2IsoMEr
  role: R package whose initEnrichment object consumes the prepared matrix ('scmatrix');
    this skill produces its required input
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: metaspace-converter
  role: Python package to download METASPACE datasets and convert them to AnnData
    objects for import into R
  repo: https://github.com/metaspace2020/metaspace-converter
- name: anndataR
  role: R package to import the AnnData object exported by metaspace-converter into
    R
  repo: https://github.com/scverse/anndataR
- name: dplyr
  role: Used for filtering annotation rows (FDR <= 0.1, HMDB database, per-condition)
    and reshaping metadata
- name: METASPACE
  role: Source knowledge base for FDR-controlled metabolite annotations used to build
    the marker query and background universe
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:2fa455b432fa8e8e50a1fe123eb4b67ebc395d1338b6e39ed6587fc1a2640e4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/S2IsoMEr
  tools:
  - METASPACE
  - R
  - S2IsoMEr
  - anndataR
  - dplyr
  - metaspace-converter
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-metabolomics-matrix-preparation

## Summary

Prepares METASPACE-derived single-cell metabolomics matrices for S2IsoMEr enrichment analysis by downloading the SpaceM NASH dataset (NASH_scm_dataset.rds), transposing it to a metabolites-by-cells matrix with per-cell condition labels, and filtering METASPACE annotations at FDR <= 0.1 to define the marker query and custom background universe.

## When to use

Use when a raw single-cell metabolomics dataset (e.g. METASPACE annotations or an .rds single-cell matrix) must be reshaped into a metabolites-by-cells input matrix with condition labels before running S2IsoMEr ORA/MSEA.

## When NOT to use

- The input is already a metabolites-by-cells matrix with condition labels and FDR-filtered annotations
- The data are bulk (non-single-cell) metabolomics measurements without per-cell resolution
- The goal is enrichment over lipid-specific ontologies requiring LipidMaps/SwissLipids identifiers not present in the HMDB-annotated matrix

## Inputs

- NASH_scm_dataset.rds — SpaceM single-cell metabolomics matrix (Rappez_2021_SpaceM project on METASPACE)
- SpaceM_metaspace_ds_annotations.rds — METASPACE annotation results for each dataset
- METASPACE dataset exportable as AnnData via metaspace-converter (Python >= 3.11)
- Condition-to-dataset mapping (condition labels such as 'U' and 'F')

## Outputs

- Metabolites-by-cells matrix with per-cell condition labels, usable as the S2IsoMEr 'scmatrix' input
- FDR <= 0.1 filtered HMDB annotation table defining the marker query (formula_adduct ions intersected with matrix rownames)
- Custom background universe of all detected HMDB annotations for Load_background

## How to apply

Download the SpaceM NASH single-cell metabolomics matrix (NASH_scm_dataset.rds) and METASPACE annotation results (SpaceM_metaspace_ds_annotations.rds) from Zenodo record 13318721. If exporting directly from METASPACE, use the metaspace-converter Python package to write an AnnData object and import it into R via anndataR. Transpose the matrix so rows are metabolites and columns are cells, and attach per-cell condition labels (e.g. 'U' untreated vs 'F' fatty-acid-treated HepaRG cells). Filter the METASPACE annotations at FDR <= 0.1 for the HMDB database within each condition; use FDR-passing formula_adduct ions intersected with the matrix rownames as the marker query (input matrix) and all detected HMDB annotations as the custom universe loaded via Load_background. The rationale is that S2IsoMEr requires the input matrix in 'scmatrix' with unambiguous row identity so isomeric/isobaric annotations can later be resampled during bootstrapping.

## Related tools

- **S2IsoMEr** (R package whose initEnrichment object consumes the prepared matrix ('scmatrix'); this skill produces its required input) — https://github.com/alexandrovteam/S2IsoMEr
- **metaspace-converter** (Python package to download METASPACE datasets and convert them to AnnData objects for import into R) — https://github.com/metaspace2020/metaspace-converter
- **anndataR** (R package to import the AnnData object exported by metaspace-converter into R) — https://github.com/scverse/anndataR
- **dplyr** (Used for filtering annotation rows (FDR <= 0.1, HMDB database, per-condition) and reshaping metadata)
- **METASPACE** (Source knowledge base for FDR-controlled metabolite annotations used to build the marker query and background universe)

## Examples

```
download.file("https://zenodo.org/records/13318721/files/NASH_scm_dataset.rds", destfile = NASH_scm_tmp)
```

## Evaluation signals

- Output matrix orientation check: rows correspond to metabolites and columns to cells, with every cell carrying a condition label ('U' or 'F')
- All marker-query formula_adduct ions pass FDR <= 0.1 and are present in the matrix rownames after intersection
- The custom universe equals the set of all detected HMDB annotations in the dataset, not the full HMDB v4 database
- Transposed matrix is loadable into an S2IsoMEr initEnrichment object without errors and preserves rownames
- Downstream per-term summaries (min_intersection = 3, alpha_cutoff = 0.05, q.val_cutoff = 0.2, boot_fract_cutoff = 0.5) yield non-empty enrichment rows including lipid-class terms

## Limitations

- MS1-based identification cannot resolve isomeric/isobaric molecules, so row identity is formula/adduct level and ambiguity must be propagated downstream by S2IsoMEr bootstrapping
- Filtering is bound to the HMDB v4 annotation database in the demonstrated workflow; other databases (SwissLipids, LipidMaps, CoreMetabolome) would require corresponding annotation availability
- The metaspace-converter route requires Python >= 3.11 and an extra AnnData/AnnDataR conversion step relative to directly loading the .rds files

## Evidence

- [methods] download the SpaceM NASH dataset: "download.file("https://zenodo.org/records/13318721/files/NASH_scm_dataset.rds", destfile = NASH_scm_tmp)"
- [methods] METASPACE converter to transform METASPACE datasets: "use the [METASPACE converter](https://github.com/metaspace2020/metaspace-converter) to transform [METASPACE](https://metaspace2020.eu/) datasets"
- [methods] AnnData import into R: "import it into R using the [anndataR](https://github.com/scverse/anndataR) package"
- [methods] FDR filter and HMDB database in demonstrated workflow: "FDR_level = 0.1
DB = ("HMDB", "v4")"
- [methods] S2IsoMEr object contains input matrix in scmatrix: "The first step is creating a `S2IsoMEr` object which contains the input matrix in `scmatrix`"
- [readme] Bootstrapping propagates molecular ambiguity into enrichment results: [evidence span withheld — anchor status: external_doc]
