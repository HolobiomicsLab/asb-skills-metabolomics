---
name: pqn-normalization-of-lipidomics-intensities
description: Use when Skyline-derived transition-level lipid intensities show sample-to-sample
  total-signal variation that must be corrected before uni/multivariate analysis or
  differential expression (limma-based de_analysis).
when_to_use_negative:
- Input is already a normalized feature/matrix table where a prior normalization step
  has been applied — re-applying PQN would distort quotients
- When sample-specific internal-standard-based normalization is available and preferred,
  since lipidr supports normalization with internal standards that may be more appropriate
  for targeted assays
- Before attaching sample annotation or summarizing transitions, when the data are
  still raw transition-level Skyline output
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: lipidr
  role: R/Bioconductor package providing PQN and internal-standard normalization of
    LipidomicsExperiment objects prior to differential analysis
  repo: github.com/ahmohamed/lipidr
- name: limma
  role: downstream linear modeling (de_analysis) that consumes the PQN-normalized
    intensities
- name: Skyline
  role: source of the exported CSV intensity measurements that PQN normalizes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:e22853443fd0082cb596c772b91f025a83f414cd8c92fd17ab9b8b98dcf7ced9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Skyline
  - limma
  - lipidr
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# PQN normalization of lipidomics intensities

## Summary

Apply probabilistic quotient normalization (PQN) to lipidomics intensity data via lipidr to correct sample-to-sample differences before differential analysis, as in the mouse serum diet dataset (HighFat_water vs NormalDiet_water) workflow.

## When to use

Use when Skyline-derived transition-level lipid intensities show sample-to-sample total-signal variation that must be corrected before uni/multivariate analysis or differential expression (limma-based de_analysis).

## When NOT to use

- Input is already a normalized feature/matrix table where a prior normalization step has been applied — re-applying PQN would distort quotients
- When sample-specific internal-standard-based normalization is available and preferred, since lipidr supports normalization with internal standards that may be more appropriate for targeted assays
- Before attaching sample annotation or summarizing transitions, when the data are still raw transition-level Skyline output

## Inputs

- LipidomicsExperiment object produced by read_skyline() from a Skyline-exported CSV (transition-level 'Area' measurements)
- Sample annotation CSV (e.g. clin.csv) attached via add_sample_annotation, containing group variables such as diet
- Optionally, lipid molecules summarized from transition level (summarize_transitions)

## Outputs

- Normalized LipidomicsExperiment with PQN-corrected per-sample lipid intensities
- QC plots (PCA score plots, box plots) showing normalization effect
- Normalized intensities feeding de_analysis (HighFat_water - NormalDiet_water, measure='Area') and volcano plots

## How to apply

After importing the Skyline export with read_skyline, attaching sample annotation with add_sample_annotation (e.g. clin.csv carrying diet-group variables), and summarizing transitions to the lipid-molecule level, apply PQN normalization with lipidr's normalize_pqn. PQN rescales each sample's lipid intensities by a sample-specific quotient (median of per-lipid intensity ratios to a reference), correcting global dilution/loading differences without requiring internal standards; lipidr additionally supports normalization with internal standards when those are present in the Skyline export. Decide whether filtering lipids by %CV is needed before or after normalization using lipidr's QC plots (box plots, PCA). Run PQN before de_analysis (measure='Area') so that the limma linear model compares comparable, normalized lipid abundances; success is judged by PCA/box plots showing reduced between-sample spread and a de_analysis result set (e.g. many significant lipids in the HighFat_water vs NormalDiet_water contrast).

## Related tools

- **lipidr** (R/Bioconductor package providing PQN and internal-standard normalization of LipidomicsExperiment objects prior to differential analysis) — github.com/ahmohamed/lipidr
- **limma** (downstream linear modeling (de_analysis) that consumes the PQN-normalized intensities)
- **Skyline** (source of the exported CSV intensity measurements that PQN normalizes)

## Examples

```
d <- read_skyline("Skyline_export.csv"); d <- add_sample_annotation(d, "clin.csv"); d <- summarize_transitions(d, method="average"); d <- normalize_pqn(d)
```

## Evaluation signals

- PCA score plot of normalized data shows reduced between-sample spread compared with pre-normalization data
- Box plots of per-sample intensity distributions are more comparable across samples after PQN
- Sample names in the annotation file (e.g. clin.csv) match exactly those in the normalized object — 'sample names must be identical in the two files'
- de_analysis on measure='Area' after PQN yields a large number of significantly altered lipids for the HighFat_water vs NormalDiet_water contrast, consistent with the published vignette finding
- Lipids failing QC (%CV filtering) are excluded before interpreting differential results

## Limitations

- PQN assumes the majority of lipids are not changing between conditions; strong global metabolic shifts may bias the quotients
- Lipids with high %CV should be filtered; lipidr supports %CV filtering for QC of measured lipids
- No changelog found in the extracted article metadata, so version-specific normalization behavior changes cannot be traced

## Evidence

- [readme] Lipids can be filtered by their %CV. Normalization methods with and without internal standards are also supported.: [evidence span withheld — anchor status: external_doc]
