---
name: r-package-example-data-processing
description: Use when a stable-isotope HRMS feature table (e.g., XCMS output) must
  be converted into isotopologue distributions with natural abundance correction for
  downstream INST-MFA or labeling time-course interpretation.
when_to_use_negative:
- Input is raw HRMS vendor files that have not yet been pre-processed with XCMS (or
  MZmine2); SIMPEL post-processing expects an untargeted metabolomics feature table.
- Data are nominal-resolution (unit-mass) MS where dual-labeled isotopologues of heteroatoms
  are indistinguishable; HRMS resolving m/z differences <0.01 Da is assumed.
- The goal is targeted quantification of metabolites without tracer labeling, where
  isotopologue distributions and natural abundance correction are not needed.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: SIMPEL
  role: 'R package that automates post-processing: isotopologue distribution generation
    (MIDsTableNoScale), natural abundance correction (NACorrectionFxnII), and MID
    plotting (MIDplot) from HRMS data'
  repo: https://github.com/SIMPELmetabolism/SIMPEL
- name: XCMS
  role: Pre-processing of HRMS acquisition data into the untargeted metabolomics feature
    table imported into SIMPEL
- name: IsoCorrectoR
  role: Dependency used by SIMPEL for natural abundance correction of isotopologue
    distributions
- name: INCA
  role: Consumes SIMPEL-exported isotopologue distribution tables for isotopically
    nonstationary metabolic flux analysis (INST-MFA)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:fa65668c18ac5267a3e82552a7daecc0ff38d0dc5d78e3974244dc76e86ef4ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SIMPELmetabolism/SIMPEL
  tools:
  - INCA
  - IsoCorrectoR
  - R
  - SIMPEL
  - XCMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# r-package-example-data-processing

## Summary

Post-process XCMS-pre-processed HRMS isotope-labeling data with the SIMPEL R package to generate isotopologue distributions and natural-abundance-corrected MIDs and average labeling values, as demonstrated on the Arabidopsis [13C5,15N2]glutamine dual-isotope example dataset.

## When to use

Use when a stable-isotope HRMS feature table (e.g., XCMS output) must be converted into isotopologue distributions with natural abundance correction for downstream INST-MFA or labeling time-course interpretation.

## When NOT to use

- Input is raw HRMS vendor files that have not yet been pre-processed with XCMS (or MZmine2); SIMPEL post-processing expects an untargeted metabolomics feature table.
- Data are nominal-resolution (unit-mass) MS where dual-labeled isotopologues of heteroatoms are indistinguishable; HRMS resolving m/z differences <0.01 Da is assumed.
- The goal is targeted quantification of metabolites without tracer labeling, where isotopologue distributions and natural abundance correction are not needed.

## Inputs

- XCMS-pre-processed feature table from HRMS isotope-labeling acquisition (e.g., [13C5,15N2]glutamine root-labeling time course)
- SIMPEL packaged compound database (compound formulas, retention times, polarity)
- SIMPEL label_scheme for the tracer (e.g., [13C5,15N2]glutamine labeling time course 0-8 h)
- ppm mass tolerance and retention-time tolerance parameters

## Outputs

- Natural-abundance-corrected isotopologue distribution (MIDs) table (e.g., NA_corrected_MIDs.csv)
- Average labeling values per compound over the labeling time course
- SIMPEL post-processed data table of isotopologue distributions for metabolic intermediates suitable for INST-MFA with INCA

## How to apply

Load the SIMPEL R package in R and explicitly attach dependencies (e.g., IsoCorrectoR) with library(), since only Depends packages auto-attach. Build the compound m/z lookup with get_comp_mz_lookup using the packaged compound formulas, retention times, polarity, and label_scheme so candidate features are matched by exact mass within the ppm tolerance. Match XCMS features to compounds with get_comp_stage using the comp_lookup_table, r_time, and rt_tolerance to assign labeled features to the correct compound isotopologue groups. Call get_table_objects_NA_corrected(XCMS_data, compounds_data, ppm, rt_tolerance, output, label_scheme), which internally generates isotopologue distributions via MIDsTableNoScale and applies natural abundance correction via NACorrectionFxnII. Save the corrected isotopologue distribution table and average labeling values (e.g., as NA_corrected_MIDs.csv) for INST-MFA with INCA.

## Related tools

- **SIMPEL** (R package that automates post-processing: isotopologue distribution generation (MIDsTableNoScale), natural abundance correction (NACorrectionFxnII), and MID plotting (MIDplot) from HRMS data) — https://github.com/SIMPELmetabolism/SIMPEL
- **XCMS** (Pre-processing of HRMS acquisition data into the untargeted metabolomics feature table imported into SIMPEL)
- **IsoCorrectoR** (Dependency used by SIMPEL for natural abundance correction of isotopologue distributions)
- **INCA** (Consumes SIMPEL-exported isotopologue distribution tables for isotopically nonstationary metabolic flux analysis (INST-MFA))

## Examples

```
library(SIMPEL); library(IsoCorrectoR); tab <- get_table_objects_NA_corrected(XCMS_data, compounds_data, ppm, rt_tolerance, output, label_scheme)
```

## Evaluation signals

- Natural-abundance contributions are removed from unlabeled (0 h) samples in the corrected MIDs table.
- Label enrichment increases monotonically over the 0-8 h [13C5,15N2]glutamine feeding time course.
- Corrected isotopologue distributions and average labeling values qualitatively reproduce the patterns reported in Fig. 1/2 of the paper for the Arabidopsis example dataset.
- XCMS features are correctly assigned to compound isotopologue groups within the specified ppm and rt_tolerance thresholds.
- Output tables have the schema expected by INCA for INST-MFA (isotopologue distributions per compound per time point).

## Limitations

- No software versions reported for SIMPEL, XCMS, MZmine2, IsoCorrectoR, or INCA, so exact reproducibility may depend on package versions.
- Only Depends packages of SIMPEL are auto-attached in R; required dependencies such as IsoCorrectoR must be loaded explicitly with library().
- Dual-isotope resolution requires HRMS; nominal-resolution m/z cannot resolve the multivariate isotopologues of dual-labeled tracers.

## Evidence

- [abstract] Available as an R package, SIMPEL extends metabolomics analyses to include isotopologue signatures necessary to quantify metabolic flux.: "Available as an R package, SIMPEL extends metabolomics analyses to include isotopologue signatures necessary to quantify metabolic flux."
- [intro] Acquired data were first pre-processed using XCMS, and post-processing data analysis was performed using SIMPEL.: "Acquired data were first pre-processed using XCMS, and post-processing data analysis was performed using SIMPEL."
- [results] pre-processed with XCMS was imported into SIMPEL (Fig. 1b) resulting in a post-processed data table containing isotopologue distributions for metabolic intermediates: "pre-processed with XCMS was imported into SIMPEL (Fig. 1b) resulting in a post-processed data table containing isotopologue distributions for metabolic intermediates"
- [methods] NACorrectionFxnII(MIDs_table, label_scheme, output): "NACorrectionFxnII(MIDs_table, label_scheme, output)"
- [methods] MIDsTableNoScale(XCMS_data, label_scheme): "MIDsTableNoScale(XCMS_data, label_scheme)"
