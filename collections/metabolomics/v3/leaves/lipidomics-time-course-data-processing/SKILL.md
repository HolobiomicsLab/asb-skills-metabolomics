---
name: lipidomics-time-course-data-processing
description: Use when analyzing LC-MS/MS lipidomics feature tables from a stable-isotope
  labeling time course (e.g., [U-13C6]glucose 0-32 h) and you need isotopologue distributions,
  natural abundance correction, or clustering of labeling kinetics.
when_to_use_negative:
- Input is raw LC-MS vendor data that has not been pre-processed with XCMS (SIMPEL
  expects the XCMS feature table)
- Data are not from a transient stable-isotope labeling experiment, so isotopologue
  distributions and labeling-kinetics clustering are not applicable
- The goal is metabolic flux estimation (INST-MFA) rather than post-processing and
  clustering of isotope-enriched lipidomics tables — SIMPEL exports tables for INCA
  in that case
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: SIMPEL
  role: 'R package performing post-processing: compound matching, isotopologue distribution
    generation (MIDsTableNoScale), natural abundance correction (NACorrectionFxnII),
    and k-means clustering of labeling profiles (getClustersAndPlots)'
  repo: github.com/SIMPELmetabolism/SIMPEL
- name: XCMS
  role: Pre-processing of acquired HRMS LC-MS data into the feature table that is
    imported into SIMPEL
- name: IsoCorrectoR
  role: Cited in the source card as the natural abundance correction backend used
    in the Camelina PC/DAG workflow
- name: tidyverse
  role: Data manipulation of feature and abundance tables during the R workflow
- name: ggplot2
  role: Plotting of cluster plots/heatmap outputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:1c5838c6e76d6f894164a6178bccd71f6fa32f9abba031a281e5dae2e4ec3ba7
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
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# lipidomics-time-course-data-processing

## Summary

Process XCMS-pre-processed HRMS lipidomics time-course data through SIMPEL to convert isotope-enriched feature tables into natural-abundance-corrected isotopologue distributions and nmol 13C product abundances, then group lipid species (e.g., PC and DAG) by labeling kinetics via k-means clustering.

## When to use

Use when analyzing LC-MS/MS lipidomics feature tables from a stable-isotope labeling time course (e.g., [U-13C6]glucose 0-32 h) and you need isotopologue distributions, natural abundance correction, or clustering of labeling kinetics.

## When NOT to use

- Input is raw LC-MS vendor data that has not been pre-processed with XCMS (SIMPEL expects the XCMS feature table)
- Data are not from a transient stable-isotope labeling experiment, so isotopologue distributions and labeling-kinetics clustering are not applicable
- The goal is metabolic flux estimation (INST-MFA) rather than post-processing and clustering of isotope-enriched lipidomics tables — SIMPEL exports tables for INCA in that case

## Inputs

- XCMS-pre-processed HRMS lipidomics feature table (e.g., packaged Camelina [U-13C6]glucose XCMS data)
- PC/DAG compound database (exact mass and retention time) for compound matching
- label_scheme describing the isotope tracer (e.g., [U-13C6]glucose)
- Time-course sample metadata (0-32 h labeling time points)

## Outputs

- Isotopologue distribution table from MIDsTableNoScale
- Natural-abundance-corrected isotopologue distributions from NACorrectionFxnII
- nmol 13C product abundance table for 2-32 h time-course samples
- k-means cluster assignments, cluster plots, and heatmap (saved via outputName)

## How to apply

Start from an XCMS-pre-processed HRMS feature table and a compound database (e.g., PC/DAG species); match compounds by exact mass with a ppm tolerance and by retention time (rt_tolerance) using SIMPEL's get_comp_mz_lookup/get_table_objects_NA_corrected. Generate isotopologue distributions with MIDsTableNoScale, supplying a label_scheme that describes the tracer ([U-13C6]glucose), then apply natural abundance correction with NACorrectionFxnII. Convert the corrected isotopologue distributions to nmol of 13C-labeled product (pool-size-normalized quantities) for each time-course sample so that lipid species are compared by amount of labeled molecules formed rather than fractional enrichment. Finally, run getClustersAndPlots with k-means (e.g., nClust = 3) on the nmol 13C abundance profiles to group species by labeling kinetics, and save cluster plots/heatmap via the outputName argument.

## Related tools

- **SIMPEL** (R package performing post-processing: compound matching, isotopologue distribution generation (MIDsTableNoScale), natural abundance correction (NACorrectionFxnII), and k-means clustering of labeling profiles (getClustersAndPlots)) — github.com/SIMPELmetabolism/SIMPEL
- **XCMS** (Pre-processing of acquired HRMS LC-MS data into the feature table that is imported into SIMPEL)
- **IsoCorrectoR** (Cited in the source card as the natural abundance correction backend used in the Camelina PC/DAG workflow)
- **tidyverse** (Data manipulation of feature and abundance tables during the R workflow)
- **ggplot2** (Plotting of cluster plots/heatmap outputs)

## Examples

```
getClustersAndPlots(nmol_13C_abundances, nClust = 3, outputName = "PC_DAG_kmeans_clusters")
```

## Evaluation signals

- k-means with nClust = 3 recovers three distinct clusters from the nmol 13C abundance profiles
- The fastest-labeled cluster consists only of two PC species (cluster 1 in the Camelina PC/DAG analysis)
- Compound matching succeeds within the specified ppm mass tolerance and rt_tolerance for PC and DAG species
- Natural abundance correction produces corrected isotopologue distributions consistent with the [U-13C6]glucose label_scheme
- Time-course plots show monotonic labeling accumulation in nmol 13C-labeled molecules over the 2-32 h window

## Limitations

- No software versions reported for SIMPEL, XCMS, MZmine2, IsoCorrectoR, or INCA
- No repository or accession identifiers provided in the results section
- Requires a defined label_scheme matching the tracer; incorrect tracer specification will corrupt the natural abundance correction

## Evidence

- [intro] Acquired data were first pre-processed using XCMS: "Acquired data were first pre-processed using XCMS, and post-processing data analysis was performed using SIMPEL."
- [intro] SIMPEL automates isotopologue generation and natural abundance correction: "SIMPEL automates post-processing data analyses of isotope enriched metabolomics datasets, generates isotopologue distributions, performs natural abundance corrections"
- [abstract] SIMPEL paired with HRMS lipidomics: "SIMPEL was paired with HRMS-based lipidomics data to describe lipid metabolism based on a single labeling experiment"
- [results] Camelina [U-13C6]glucose time course: [evidence span withheld — anchor status: not_found]
- [methods] MIDsTableNoScale and NACorrectionFxnII signatures: [evidence span withheld — anchor status: not_found]
