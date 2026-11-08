---
name: kegg-metabolite-database-searching
description: Use when you have preprocessed LC/MS feature tables (m/z per feature,
  e.g. from apLCMS cdf.to.ftr or XCMS centWave) and need to quantify how many features
  can be matched to known metabolites via KEGG.
when_to_use_negative:
- Input is raw vendor/NetCDF spectra rather than a preprocessed feature table — peak
  detection and alignment (e.g. apLCMS cdf.to.ftr) must be done first
- Goal is correcting between-batch intensity differences — post-hoc batch effect correction
  only adjusts signal intensity and cannot fix feature misalignment, so annotation
  matching is not a substitute for batch-aware preprocessing
- The study used positive-mode ionization — the specified adduct list ([M-H]-, [M+Cl]-,
  etc.) applies only to negative mode
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: xMSAnnotator
  role: Performs the m/z-to-KEGG metabolite annotation using the configured negative-mode
    adduct list on each feature table
- name: apLCMS
  role: Produces the two-stage (batch-aware) and traditional single-batch feature
    tables from ST000868 raw data that are annotated in this skill
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Environment in which xMSAnnotator is run and the annotation proportions are
    computed, tabulated, and plotted; apLCMS dependencies must be attached explicitly
    with library(<dep>)
- name: XCMS
  role: Provides the centWave peak detection feature table used as the comparison
    baseline in the Fig. 3b annotation-rate comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:1809ed3f74f5523b6aefc04649131a5a6d3d16eeee48b81121fd43758cd9c90d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - XCMS
  - apLCMS
  - xMSAnnotator
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# kegg-metabolite-database-searching

## Summary

Annotates LC/MS feature m/z values against KEGG compound masses using xMSAnnotator with a negative-mode adduct ion list, computing the proportion of annotated features per preprocessing method (Fig. 3b-style comparison on ST000868).

## When to use

Use when you have preprocessed LC/MS feature tables (m/z per feature, e.g. from apLCMS cdf.to.ftr or XCMS centWave) and need to quantify how many features can be matched to known metabolites via KEGG.

## When NOT to use

- Input is raw vendor/NetCDF spectra rather than a preprocessed feature table — peak detection and alignment (e.g. apLCMS cdf.to.ftr) must be done first
- Goal is correcting between-batch intensity differences — post-hoc batch effect correction only adjusts signal intensity and cannot fix feature misalignment, so annotation matching is not a substitute for batch-aware preprocessing
- The study used positive-mode ionization — the specified adduct list ([M-H]-, [M+Cl]-, etc.) applies only to negative mode

## Inputs

- Feature table from two-stage apLCMS preprocessing of ST000868 (m/z, RT, per-sample intensities)
- Feature table from traditional single-batch apLCMS preprocessing of ST000868
- Feature table from XCMS with centWave peak detection on ST000868
- KEGG reference metabolite database
- Negative-mode adduct ion list ([M-H]- through [M+2Cl]2-)

## Outputs

- fig3b_annotation_proportions.csv with one annotation proportion (0–1) per preprocessing setting
- Fig. 3b-style bar chart comparing annotation rates across the three preprocessing methods
- Per-method counts of KEGG-annotated vs. total features

## How to apply

Load each ST000868 feature table (from the two-stage apLCMS hybrid, traditional single-batch apLCMS, and XCMS centWave preprocessing) and run xMSAnnotator separately on each, with KEGG as the reference metabolite database and the paper's negative-mode adduct list: [M-H]-, [M-2H]2-, [M-2H+Na]-, [M-2H+K]-, [M-2H+NH4]-, [M-H2O-H]-, [M-H+Cl]2-, [M+Cl]-, [M+2Cl]2-. xMSAnnotator matches each feature m/z to a KEGG compound mass under the allowed adduct transformations, so the comparison is fair only if the identical adduct list and database are used across all three tables. For each setting, compute the annotation proportion as annotated features divided by total features. Assemble the proportions into fig3b_annotation_proportions.csv and draw a Fig. 3b-style bar chart; expect the three methods to perform similarly, with XCMS centWave slightly higher. The rationale (from the source paper) is that this annotation rate serves as a downstream consistency check of whether batch-aware preprocessing preserves more interpretable features than traditional preprocessing.

## Related tools

- **xMSAnnotator** (Performs the m/z-to-KEGG metabolite annotation using the configured negative-mode adduct list on each feature table)
- **apLCMS** (Produces the two-stage (batch-aware) and traditional single-batch feature tables from ST000868 raw data that are annotated in this skill) — https://github.com/tianwei-yu/apLCMS
- **R** (Environment in which xMSAnnotator is run and the annotation proportions are computed, tabulated, and plotted; apLCMS dependencies must be attached explicitly with library(<dep>))
- **XCMS** (Provides the centWave peak detection feature table used as the comparison baseline in the Fig. 3b annotation-rate comparison)

## Evaluation signals

- fig3b_annotation_proportions.csv exists and contains exactly one proportion per preprocessing setting (two-stage apLCMS, traditional apLCMS, XCMS centWave)
- Every annotation proportion lies between 0 and 1 (annotated features / total features)
- The same negative-mode adduct list and KEGG database were used for all three feature tables — annotation runs differ only in the input table
- The bar chart's relative ordering matches Fig. 3b: the three methods are similar, with XCMS centWave slightly highest
- Each xMSAnnotator run reports a non-zero number of matched features; zero matches indicates a misconfigured adduct list or m/z unit mismatch

## Limitations

- The comparison was run on the ST000868 dataset with negative-mode adducts; results may not transfer to positive mode or other datasets
- Annotation proportions depend on the chosen adduct list — adding/removing adducts (e.g. [M+Cl]-, [M-2H]2-) changes denominators and matches
- The three methods 'performed similarly', so the annotation rate is a weak discriminator between preprocessing pipelines
- The article does not state an apLCMS package version, and no apLCMS package limitations for this comparison are discussed

## Evidence

- [abstract] the new method resulted in feature tables with better consistency, as well as better down-stream analysis results: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
- [other] the methods performed similarly, with XCMS with centWave peak detection yielding a slightly higher rate of matching (Fig. 3b): [evidence span withheld — anchor status: not_found]
- [other] Configure xMSAnnotator with KEGG as the reference metabolite database and the paper's negative-mode adduct ion list: [M-H]-, [M-2H]2-, [M-2H+Na]-, [M-2H+K]-, [M-2H+NH4]-, [M-H2O-H]-, [M-H+Cl]2-, [M+Cl]-, [M+2Cl]2-: [evidence span withheld — anchor status: not_found]
- [discussion] The package can be downloaded at https://github.com/tianwei-yu/apLCMS.: "The package can be downloaded at https://github.com/tianwei-yu/apLCMS."
