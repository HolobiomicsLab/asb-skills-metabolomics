---
name: k-means-clustering-of-labeling-kinetics
description: Use when NA-corrected isotopologue distributions from a stable-isotope
  time course (e.g., [U-13C6]glucose 0–32 h lipidomics) have been converted to molar
  13C quantities and species must be grouped by labeling kinetics.
when_to_use_negative:
- The input is still a raw or natural-abundance-uncorrected XCMS feature table — clustering
  must be done on NA-corrected, pool-size-normalized nmol 13C quantities, not fractional
  isotopologue distributions
- The data lack a time-course dimension (single time point or steady-state labeling),
  where kinetic clustering is meaningless
- The goal is pathway-level flux resolution (INST-MFA with INCA) rather than descriptive
  grouping of labeling kinetics
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SIMPEL
  role: R package providing the pipeline from XCMS feature table to nmol 13C quantities
    and the k-means clustering/plots (MIDsTableNoScale, NACorrectionFxnII, getClustersAndPlots)
  repo: github.com/SIMPELmetabolism/SIMPEL
- name: R
  role: Language in which SIMPEL is distributed and the clustering analysis is executed
  repo: github.com/SIMPELmetabolism/SIMPEL
- name: XCMS
  role: Pre-processing of acquired HRMS lipidomics data into the feature table that
    SIMPEL imports before clustering
- name: ggplot2
  role: Rendering of cluster line plots and heatmaps via the outputName argument
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:cd4baf0a3211de4eb3c83cd505926523ee90cf25bd3e18fed51b4ba6e4bb89dd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoCorrectoR
  - R
  - SIMPEL
  - XCMS
  - ggplot2
  - tidyverse
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# k-means-clustering-of-labeling-kinetics

## Summary

Groups isotope-labeled lipid or metabolite species by labeling kinetics using k-means clustering of molar 13C-enrichment time courses. In the SIMPEL Camelina [U-13C6]glucose lipidomics workflow, three clusters recovered PC and DAG species whose 13C quantities (nmol labeled molecules formed) accumulate at distinct rates.

## When to use

Use when NA-corrected isotopologue distributions from a stable-isotope time course (e.g., [U-13C6]glucose 0–32 h lipidomics) have been converted to molar 13C quantities and species must be grouped by labeling kinetics.

## When NOT to use

- The input is still a raw or natural-abundance-uncorrected XCMS feature table — clustering must be done on NA-corrected, pool-size-normalized nmol 13C quantities, not fractional isotopologue distributions
- The data lack a time-course dimension (single time point or steady-state labeling), where kinetic clustering is meaningless
- The goal is pathway-level flux resolution (INST-MFA with INCA) rather than descriptive grouping of labeling kinetics

## Inputs

- NA-corrected isotopologue distribution table for PC/DAG species (output of NACorrectionFxnII)
- Lipid pool-size (nmol) measurements per time point for converting fractional enrichment to nmol 13C product
- Time-course sample metadata (e.g., [U-13C6]glucose 2–32 h Camelina embryo samples)
- Compound database (PC/DAG exact masses and retention times) for feature-to-compound matching

## Outputs

- k-means cluster assignments for each labeled lipid species (e.g., three kinetic clusters)
- Cluster line plots of nmol 13C abundance over the labeling time course
- Cluster heatmap saved via the outputName argument

## How to apply

First process the XCMS-pre-processed HRMS lipidomics feature table through SIMPEL: match PC/DAG compounds to features with get_comp_mz_lookup/get_table_objects_NA_corrected using exact mass (ppm tolerance) and retention-time matching, generate isotopologue distributions with MIDsTableNoScale, and apply natural abundance correction with NACorrectionFxnII against the [U-13C6]glucose label_scheme. Critically, convert the corrected isotopologue fractions to nmol of 13C-labeled molecules formed by normalizing enrichment to the measured pool size — clustering on raw fractional enrichment instead would confound kinetics with pool size. Then run k-means (e.g., nClust = 3 as in the Camelina PC/DAG analysis) on the nmol 13C abundance profiles across the 2–32 h time points via getClustersAndPlots, and save the cluster plots/heatmap with the outputName argument. Judge success by whether the clustering recovers kinetically coherent groups — in the published analysis, three clusters with the fastest-labeled cluster containing only two PC species.

## Related tools

- **SIMPEL** (R package providing the pipeline from XCMS feature table to nmol 13C quantities and the k-means clustering/plots (MIDsTableNoScale, NACorrectionFxnII, getClustersAndPlots)) — github.com/SIMPELmetabolism/SIMPEL
- **R** (Language in which SIMPEL is distributed and the clustering analysis is executed) — github.com/SIMPELmetabolism/SIMPEL
- **XCMS** (Pre-processing of acquired HRMS lipidomics data into the feature table that SIMPEL imports before clustering)
- **ggplot2** (Rendering of cluster line plots and heatmaps via the outputName argument)

## Examples

```
clusters <- getClustersAndPlots(nmol_13C_abundances, nClust = 3, outputName = "Camelina_PC_DAG_kinetic_clusters")
```

## Evaluation signals

- The expected number of kinetically distinct clusters (three in the published Camelina PC/DAG analysis) is recovered with k-means nClust = 3
- The fastest-labeled cluster consists only of two PC species, matching the published finding
- Clustering is performed on nmol of 13C-labeled molecules (pool-size-normalized), not on fractional enrichment, so cluster ordering reflects labeling kinetics rather than pool size
- Every PC and DAG species in the 2–32 h time course receives exactly one cluster assignment with no NA handling errors
- Cluster plots and heatmap are written to files specified by the outputName argument

## Limitations

- No software versions were reported for SIMPEL or the downstream tooling in the article
- Cluster count (k) is user-chosen (nClust = 3 in the exemplar); no objective selection criterion is described
- Interpretation depends on matching compounds by exact mass and retention time; mis-matched features propagate into cluster assignments

## Evidence

- [abstract] SIMPEL was paired with HRMS-based lipidomics data to describe lipid metabolism based on a single labeling experiment: "SIMPEL was paired with HRMS-based lipidomics data to describe lipid metabolism based on a single labeling experiment"
- [results] Label developing Camelina sativa embryos with [U-13C6] glucose in a time-course experiment (0-32 h): [evidence span withheld — anchor status: not_found]
- [methods] MIDsTableNoScale(XCMS_data, label_scheme): "MIDsTableNoScale(XCMS_data, label_scheme)"
- [methods] NACorrectionFxnII(MIDs_table, label_scheme, output): "NACorrectionFxnII(MIDs_table, label_scheme, output)"
