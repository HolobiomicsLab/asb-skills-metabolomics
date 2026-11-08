---
name: isotopologue-to-nmol-13c-unit-conversion
description: Use when isotopologue distributions from a [U-13C6]glucose or similar
  tracer HRMS time course have already been natural-abundance corrected (NACorrectionFxnII)
  and the goal is to express enrichment as nmol of 13C-labeled molecules formed per
  time point for kinetic clustering or precursor-product.
when_to_use_negative:
- Input is a raw XCMS feature table or uncorrected isotopologue fractions — natural
  abundance correction (NACorrectionFxnII) must precede the nmol-13C conversion or
  amounts will be biased by unlabeled isotopologue spillover
- The analysis goal is flux estimation via INST-MFA, where SIMPEL tables are exported
  to INCA rather than converted to nmol product abundances
- Data come from nominal-resolution MS where dual-labeled isotopologues are indistinguishable;
  HRMS m/z resolution is assumed by the isotopologue model
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: SIMPEL
  role: R package providing MIDsTableNoScale, NACorrectionFxnII, and getClustersAndPlots
    used to generate, correct, and cluster the nmol-13C abundance profiles
  repo: github.com/SIMPELmetabolism/SIMPEL
- name: R
  role: Runtime environment for the SIMPEL package and the tidyverse/ggplot2-based
    plotting of converted abundance profiles
- name: IsoCorrectoR
  role: Natural abundance correction support in the isotope-correction step preceding
    the nmol-13C conversion
- name: XCMS
  role: Pre-processing of HRMS lipidomics acquisitions into the feature table that
    SIMPEL imports for isotopologue quantification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:56d9a02945cdfa365b098defc8bd775a2efad3183c22b252df3127a72a70f527
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# isotopologue-to-nmol-13c-unit-conversion

## Summary

Converts natural-abundance-corrected isotopologue distributions from SIMPEL-processed HRMS stable-isotope labeling data into absolute nmol abundances of 13C-labeled product molecules, so that labeling kinetics can be compared across lipid species of different pool sizes (e.g., Camelina PC and DAG over a [U-13C6]glucose 0-32 h time course).

## When to use

Use when isotopologue distributions from a [U-13C6]glucose or similar tracer HRMS time course have already been natural-abundance corrected (NACorrectionFxnII) and the goal is to express enrichment as nmol of 13C-labeled molecules formed per time point for kinetic clustering or precursor-product analysis.

## When NOT to use

- Input is a raw XCMS feature table or uncorrected isotopologue fractions — natural abundance correction (NACorrectionFxnII) must precede the nmol-13C conversion or amounts will be biased by unlabeled isotopologue spillover
- The analysis goal is flux estimation via INST-MFA, where SIMPEL tables are exported to INCA rather than converted to nmol product abundances
- Data come from nominal-resolution MS where dual-labeled isotopologues are indistinguishable; HRMS m/z resolution is assumed by the isotopologue model

## Inputs

- Natural-abundance-corrected isotopologue distribution table from NACorrectionFxnII (SIMPEL, R)
- XCMS-preprocessed HRMS lipidomics feature table (Camelina PC/DAG, [U-13C6]glucose time course)
- [U-13C6]glucose label_scheme definition
- PC/DAG compound database (exact mass, retention time) matched via ppm and rt_tolerance

## Outputs

- Matrix of nmol 13C-labeled product abundances per lipid species per time point (2-32 h)
- Labeled k-means cluster plots/heatmap (via getClustersAndPlots outputName argument)

## How to apply

In SIMPEL, first match compounds from the XCMS feature table to a PC/DAG compound database with get_comp_mz_lookup/get_table_objects_NA_corrected using exact-mass ppm tolerance and retention-time tolerance. Generate isotopologue distributions with MIDsTableNoScale(XCMS_data, label_scheme) and correct them for natural abundance with NACorrectionFxnII using the [U-13C6]glucose label_scheme; this yields fractional enrichment per isotopologue. The conversion step then multiplies corrected fractional abundances by species pool size to obtain nmol of 13C-labeled product molecules for each 2-32 h time-point sample, normalizing away differences in PC/DAG pool size. The resulting nmol-13C abundance matrix (species x time) is the correct input for k-means labeling-kinetics clustering via getClustersAndPlots (nClust = 3), rather than raw fractional MIDs which confound enrichment with abundance.

## Related tools

- **SIMPEL** (R package providing MIDsTableNoScale, NACorrectionFxnII, and getClustersAndPlots used to generate, correct, and cluster the nmol-13C abundance profiles) — github.com/SIMPELmetabolism/SIMPEL
- **R** (Runtime environment for the SIMPEL package and the tidyverse/ggplot2-based plotting of converted abundance profiles)
- **IsoCorrectoR** (Natural abundance correction support in the isotope-correction step preceding the nmol-13C conversion)
- **XCMS** (Pre-processing of HRMS lipidomics acquisitions into the feature table that SIMPEL imports for isotopologue quantification)

## Examples

```
mids <- MIDsTableNoScale(XCMS_data, label_scheme); na_corr <- NACorrectionFxnII(mids, label_scheme, "na_corrected"); getClustersAndPlots(nmol_13C_abundances, nClust = 3, outputName = "PC_DAG_13C_clusters")
```

## Evaluation signals

- k-means (nClust = 3) on the nmol-13C profiles recovers three distinct clusters, with cluster 1 containing only two PC species and being the fastest labeled group
- Cluster assignments differ from what raw fractional MIDs would give, confirming pool-size normalization was applied
- Corrected isotopologue fractions per species sum to 1 after NACorrectionFxnII before the nmol conversion
- All 2-32 h time-course samples are represented for each PC/DAG species in the output matrix (no NA rows after get_table_objects_NA_corrected)
- Output plots/heatmap files are written under the getClustersAndPlots outputName argument

## Limitations

- No software versions reported for SIMPEL, XCMS, IsoCorrectoR, or INCA, so exact-version reproducibility is not documented
- Conversion quality depends on correct compound matching by exact mass (ppm tolerance) and retention time; mis-assignment propagates into nmol amounts
- Quantitative outputs depend on the label_scheme correctly specifying the [U-13C6]glucose tracer

## Evidence

- [intro] SIMPEL automates post-processing data analyses of isotope enriched metabolomics datasets, generates isotopologue distributions, performs natural abundance corrections: "SIMPEL automates post-processing data analyses of isotope enriched metabolomics datasets, generates isotopologue distributions, performs natural abundance corrections"
- [methods] NACorrectionFxnII(MIDs_table, label_scheme, output): "NACorrectionFxnII(MIDs_table, label_scheme, output)"
- [results] Developing Camelina sativa seeds... were labeled with [U-13C6] glucose in a time-course experiment (0–32 h): [evidence span withheld — anchor status: not_found]
- [results] k-means clustering of the quantities of 13C PC and DAG species formed over the labeling time course revealed three distinct clusters: [evidence span withheld — anchor status: not_found]
