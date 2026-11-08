---
name: differential-abundance-analysis
description: Use when comparing taxonomic (species-level) or molecular abundances
  across two or more condition groups — such as 3xTg versus WT fecal shotgun metagenomic
  profiles — and you need log-fold changes with FDR-adjusted significance.
when_to_use_negative:
- When the input is already a raw read-count matrix without species-level profiling
  or when sample size per group is too small for a compositional model to be meaningful
- When the goal is functional pathway inference rather than taxonomic or molecular
  feature differential abundance (e.g. pathway-level enrichment)
- When the contrast is multi-factor with strong confounding (e.g. cage or batch effects
  unaccounted for) and no design matrix to adjust for them
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0625
tools:
- name: ANCOM-BC2
  role: 'Primary statistical engine: bias-corrected compositional model estimating
    species-level differential abundance log-fold changes and adjusted p-values'
- name: tissueMASST
  role: Supporting search tool for cross-referencing molecular features linked to
    the same AD animal models in public metabolomics repositories, complementing species-level
    DA results
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:b4a9784ad46455ca1ee40ef26010a01e5ba7dde7c777bb3cfcdf83c061e233f5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANCOM-BC2
  - tissueMASST
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# differential-abundance-analysis

## Summary

Species- or feature-level differential abundance testing (e.g. ANCOM-BC2) that estimates compositionally-aware log-fold changes and adjusted p-values between biological groups, applied here to fecal shotgun metagenomic species profiles contrasting 3xTg AD model mice versus WT controls.

## When to use

Use when comparing taxonomic (species-level) or molecular abundances across two or more condition groups — such as 3xTg versus WT fecal shotgun metagenomic profiles — and you need log-fold changes with FDR-adjusted significance.

## When NOT to use

- When the input is already a raw read-count matrix without species-level profiling or when sample size per group is too small for a compositional model to be meaningful
- When the goal is functional pathway inference rather than taxonomic or molecular feature differential abundance (e.g. pathway-level enrichment)
- When the contrast is multi-factor with strong confounding (e.g. cage or batch effects unaccounted for) and no design matrix to adjust for them

## Inputs

- Species-level taxonomic abundance table derived from fecal shallow shotgun metagenomics data (e.g. 666 paired SPF Sacrifice cohort samples from 3xTg and WT C57BL/6J animals)
- Sample metadata specifying the primary contrast (3xTg vs WT genotype), plus cohort/colcolonization status information

## Outputs

- ancom_bc2_results.csv containing species-level log-fold changes, adjusted p-values, and significance calls for every tested species
- Interpreted direction-of-change summary (enriched vs depleted species) aligned with published findings for expert review

## How to apply

Start from a species-level taxonomic abundance table (generated from fecal shallow shotgun metagenomics reads, e.g. the 666-sample SPF Sacrifice cohort) joined with genotype metadata (3xTg vs WT). Run ANCOM-BC2 with genotype as the primary contrast; ANCOM-BC2 handles compositionality through its bias-corrected log-linear model, yielding per-species log-fold changes and adjusted p-values. Extract significant species, then verify the direction of effects against known biology: enrichment of M. schaedleri and Helicobacter/Desulfovibrio/Escherichia/Shigella species, and depletion of Akkermansia muciniphila and Bifidobacterium species in 3xTg animals. Write the full result table with log-fold changes and adjusted p-values to a CSV (e.g. ancom_bc2_results.csv) for downstream expert review and cross-checking against published figures (Figure 5D).

## Related tools

- **ANCOM-BC2** (Primary statistical engine: bias-corrected compositional model estimating species-level differential abundance log-fold changes and adjusted p-values)
- **tissueMASST** (Supporting search tool for cross-referencing molecular features linked to the same AD animal models in public metabolomics repositories, complementing species-level DA results)

## Examples

```
R: library(ANCOMBC); res <- ancombc2(data = spe_table, fix_formula = 'genotype', group = 'genotype', zero_cut = 0.90, output = 'ancom_bc2_results.csv')
```

## Evaluation signals

- Output CSV contains per-species log-fold change, standard error, and adjusted p-value columns with no missing values for tested species
- Direction of significant species matches published Figure 5D: M. schaedleri and Helicobacter/Desulfovibrio/Escherichia/Shigella enriched, Akkermansia muciniphila and Bifidobacterium depleted in 3xTg
- Adjusted p-values are FDR-corrected (e.g. Benjamini–Hochberg) and lie in [0,1], with significant calls typically < 0.05
- Sample count and group balance match the source cohort (n=666 paired SPF Sacrifice samples across 3xTg and WT)
- Full species table, not a filtered subset, is retained in the output CSV to allow downstream re-ranking or threshold changes

## Limitations

- ANCOM-BC2 is compositional: relative abundance changes do not directly imply absolute biomass differences; interpretation should be framed in compositional terms
- Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed and can be attributed to differences in sample collection, so cohort identity must be modelled or separated
- 3xTg and WT mice were bred and housed separately before the start of the experiment, a potential confound for any differential abundance conclusion
- Longitudinal fecal samples lacked animal IDs (collected randomly per cage), preventing single-animal paired analysis if longitudinal designs are attempted
- Hundreds of differentially abundant molecules remain non-annotated, so a purely feature-level DA call list can be biologically incomplete without follow-up annotation

## Evidence

- [methods] Run ANCOM-BC2 at the species level, using genotype (3xTg vs WT) as the primary contrast, to estimate differential abundance log-fold changes and adjusted p-values: [evidence span withheld — anchor status: not_found]
- [results] 3xTg mice harbored increased abundance of M. schaedleri and species of the Helicobacter, Desulfovibrio, Escherichia, and Shigella genera, and decreased abundance of A. muciniphila strains.: [evidence span withheld — anchor status: not_found]
- [intro] we generate and share a large centralized untargeted metabolomics dataset comprising 2,266 unique biological samples: "we generate and share a large centralized untargeted metabolomics dataset comprising 2,266 unique biological samples"
- [discussion] Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data: "Differences between experimental cohorts (Longitudinal and Sacrifice) within each study were observed for both microbiome and metabolomics data"
- [discussion] 3xTg and WT mice were bred and housed separately before the start of the experiment: "3xTg and WT mice were bred and housed separately before the start of the experiment"
