---
name: stable-isotope-lipidomics-compound-annotation
description: Use when HRMS lipidomics feature tables from a stable-isotope ([U-13C6]
  tracer) time course must be matched to lipid species and converted to isotope-corrected
  labeled quantities, e.g., to cluster PC/DAG species by labeling kinetics.
when_to_use_negative:
- Data are raw vendor files or mzML that have not been feature-detected — pre-process
  with XCMS (or MZmine2) first, since SIMPEL consumes XCMS-style feature tables
- The experiment is a steady-state or single-pulse endpoint metabolomics run where
  pool-size-normalized labeling kinetics across a time course are not of interest
- Targeted low-resolution MS data where isotopologues of dual-labeled tracers cannot
  be resolved — HRMS is required to distinguish isotopologues differing by <0.01 Da
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SIMPEL
  role: R package providing compound matching (get_comp_mz_lookup/get_table_objects_NA_corrected),
    isotopologue distribution generation (MIDsTableNoScale), natural abundance correction
    (NACorrectionFxnII), and clustering/plots (getClustersAndPlots)
  repo: github.com/SIMPELmetabolism/SIMPEL
- name: XCMS
  role: Pre-processing of HRMS lipidomics acquisitions into the feature table that
    SIMPEL imports for compound annotation and isotope analysis
- name: R
  role: Runtime environment for the SIMPEL package and downstream clustering/plotting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:34d0282791abb4af132af2cf351e8eebdb7cf1f0cc6f81c1ab61fd00d9403768
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# stable-isotope-lipidomics-compound-annotation

## Summary

Annotate lipid compounds (e.g., PC and DAG species) in XCMS-pre-processed HRMS stable-isotope labeling data using the SIMPEL R package, matching features to a compound database by exact mass (ppm tolerance) and retention time, then generate and natural-abundance-correct isotopologue distributions so that 13C-labeled lipid abundances (nmol) can be quantified and grouped by labeling kinetics.

## When to use

Use when HRMS lipidomics feature tables from a stable-isotope ([U-13C6] tracer) time course must be matched to lipid species and converted to isotope-corrected labeled quantities, e.g., to cluster PC/DAG species by labeling kinetics.

## When NOT to use

- Data are raw vendor files or mzML that have not been feature-detected — pre-process with XCMS (or MZmine2) first, since SIMPEL consumes XCMS-style feature tables
- The experiment is a steady-state or single-pulse endpoint metabolomics run where pool-size-normalized labeling kinetics across a time course are not of interest
- Targeted low-resolution MS data where isotopologues of dual-labeled tracers cannot be resolved — HRMS is required to distinguish isotopologues differing by <0.01 Da

## Inputs

- XCMS-pre-processed untargeted HRMS lipidomics feature table (m/z, retention time, intensities across time-course samples)
- Compound database of lipid species (e.g., PC and DAG entries with exact masses and retention times)
- Label scheme defining the tracer (e.g., [U-13C6]glucose) and labeled atoms
- Sample time-course metadata (labeling time points, e.g., 0-32 h)

## Outputs

- Compound-annotated isotopologue distribution table per PC/DAG species per time point
- Natural-abundance-corrected isotopologue distributions
- Table of nmol 13C-labeled lipid abundances over the time course
- K-means cluster assignments of lipid species with labeled cluster plots/heatmap

## How to apply

Load the XCMS-pre-processed feature table and the PC/DAG compound database into SIMPEL, matching compounds to features by exact mass within a ppm tolerance and by retention time via rt_tolerance (get_comp_mz_lookup/get_table_objects_NA_corrected). Generate unscaled isotopologue distributions with MIDsTableNoScale(XCMS_data, label_scheme) using the tracer's label scheme (e.g., [U-13C6]glucose), then apply natural abundance correction with NACorrectionFxnII to remove isotope signal attributable to natural 13C/2H/15N/O contributions. Convert NA-corrected isotopologue distributions to nmol of 13C-labeled molecules formed for each time-course sample (e.g., 2-32 h in developing Camelina sativa embryos). Cluster the resulting labeled-abundance profiles with k-means (e.g., nClust = 3) via getClustersAndPlots to group lipid species by labeling kinetics, and save plots/heatmaps using the outputName argument. The rationale is that normalizing enrichment to pool size (nmol of labeled molecules) distinguishes species labeled rapidly in small pools from those accruing label slowly, revealing precursor-product relationships such as PC/DAG acyl editing.

## Related tools

- **SIMPEL** (R package providing compound matching (get_comp_mz_lookup/get_table_objects_NA_corrected), isotopologue distribution generation (MIDsTableNoScale), natural abundance correction (NACorrectionFxnII), and clustering/plots (getClustersAndPlots)) — github.com/SIMPELmetabolism/SIMPEL
- **XCMS** (Pre-processing of HRMS lipidomics acquisitions into the feature table that SIMPEL imports for compound annotation and isotope analysis)
- **R** (Runtime environment for the SIMPEL package and downstream clustering/plotting)

## Examples

```
mids <- MIDsTableNoScale(XCMS_data, label_scheme); na_corr <- NACorrectionFxnII(mids, label_scheme, output); getClustersAndPlots(na_corr, nClust = 3, outputName = "camelina_clusters")
```

## Evaluation signals

- K-means clustering (nClust = 3) recovers three distinct labeling-kinetic clusters, with the fastest-labeled cluster consisting only of two PC species
- Every matched PC/DAG feature is consistent with the compound's exact mass within the ppm tolerance and co-elutes within the rt_tolerance window
- NA-corrected isotopologue distributions sum appropriately and show near-zero label at the 0 h (unlabeled) control time point
- nmol 13C-labeled quantities increase monotonically over the 2-32 h [U-13C6]glucose time course for labeled species, consistent with accumulating label
- Cluster plots/heatmap are written to the path given via outputName without missing-value artifacts (NA-corrected tables)

## Limitations

- No software versions were reported for SIMPEL, XCMS, MZmine2, or IsoCorrectoR in the article, so exact parameter reproducibility across versions is unverified
- No repository or accession identifiers were provided in the results section for the packaged datasets, requiring the SIMPEL GitHub package data as a stand-in
- Compound matching depends on the quality of the compound database's exact masses and retention times; poor rt_tolerance settings can misassign lipid species

## Evidence

- [abstract] SIMPEL was paired with HRMS-based lipidomics data to describe lipid metabolism based on a single labeling experiment: "SIMPEL was paired with HRMS-based lipidomics data to describe lipid metabolism based on a single labeling experiment"
- [methods] MIDsTableNoScale function signature: "MIDsTableNoScale(XCMS_data, label_scheme)"
- [methods] NACorrectionFxnII function signature: "NACorrectionFxnII(MIDs_table, label_scheme, output)"
- [results] Camelina time course labeling: [evidence span withheld — anchor status: not_found]
- [intro] HRMS isotopologue resolution: "High resolution mass spectrometry (HRMS) can distinguish metabolites (and isotopologues of heteroatoms) through accurate measurement of m/z values that differ by <0.01 Da"
