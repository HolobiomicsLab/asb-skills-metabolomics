---
name: autoq-integration-table-processing
description: Use when you already hold the isoSCAN autoQ integration table from high-resolution
  GC-CI-MS (GC-(q)TOF or GC-Orbitrap) stable-isotope-labelling runs and need per-metabolite
  isotopologue summaries or a barplot of a single compound such as Gly across sample
  groups.
when_to_use_negative:
- Low-resolution autoQ/SIM/NS integration tables — sumIsotopologues is stated as applicable
  only to high-resolution autoQ results
- Input is raw GC-CI-MS vendor data rather than an already-produced isoSCAN autoQ
  integration table
- Visualising many metabolites at once rather than a single targeted compound per
  metBarPlot call
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: isoSCAN
  role: R package whose sumIsotopologues function aggregates high-resolution autoQ
    isotope-integration results into per-metabolite isotopologue tables and whose
    metBarPlot function renders the per-compound barplot across sample groups.
  repo: https://github.com/jcapelladesto/isoSCAN
- name: R
  role: Execution environment in which the isoSCAN package functions (sumIsotopologues,
    metBarPlot) are called.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:bfc8a9e910c9bd93e872ed17e0a64d3c654b0be609f66186ef268ae48b870332
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# autoq-integration-table-processing

## Summary

Process isoSCAN's high-resolution autoQ isotope-integration outputs from GC-CI-HRMS data: aggregate them into summed per-metabolite isotopologue tables (sumIsotopologues) and visualise a targeted compound across sample groups (metBarPlot).

## When to use

Use when you already hold the isoSCAN autoQ integration table from high-resolution GC-CI-MS (GC-(q)TOF or GC-Orbitrap) stable-isotope-labelling runs and need per-metabolite isotopologue summaries or a barplot of a single compound such as Gly across sample groups.

## When NOT to use

- Low-resolution autoQ/SIM/NS integration tables — sumIsotopologues is stated as applicable only to high-resolution autoQ results
- Input is raw GC-CI-MS vendor data rather than an already-produced isoSCAN autoQ integration table
- Visualising many metabolites at once rather than a single targeted compound per metBarPlot call

## Inputs

- isoSCAN autoQ isotope-integration table from high-resolution GC-CI-MS (GC-(q)TOF or GC-Orbitrap) stable-isotope-labelling experiments
- Compound name of interest to filter on (e.g. 'Gly')

## Outputs

- Summed per-metabolite isotopologue table produced by sumIsotopologues
- Barplot image (e.g. gly_metBarPlot.png) of the target compound's isotopologue abundances across sample groups

## How to apply

Start from the high-resolution autoQ integration table that isoSCAN emits after processing GC-CI-HRMS data (Agilent 7890B GC coupled to a 7250 QTOF mass spectrometer); the vignette notes sumIsotopologues applies only to high-res autoQ results, not low-resolution SIM/NS data. Apply sumIsotopologues to the table to aggregate individual autoQ isotope integrations into per-metabolite isotopologue summary tables, mapping each metabolite's isotopologue abundance onto the summed table. From that summed table, extract rows for the target compound, e.g. where CompoundName equals 'Gly', to isolate its isotopologue profile across sample groups. Pass the extracted Gly rows to isoSCAN's metBarPlot function to draw a barplot of isotopologue abundances across sample groups; the function may emit 'Not quantified' warnings for groups lacking quantified values, which is expected behaviour matching the vignette's example figure. Save the rendered plot (e.g. gly_metBarPlot.png) and verify the figure file was created as a completion check.

## Related tools

- **isoSCAN** (R package whose sumIsotopologues function aggregates high-resolution autoQ isotope-integration results into per-metabolite isotopologue tables and whose metBarPlot function renders the per-compound barplot across sample groups.) — https://github.com/jcapelladesto/isoSCAN
- **R** (Execution environment in which the isoSCAN package functions (sumIsotopologues, metBarPlot) are called.)

## Examples

```
install_github("jcapelladesto/isoSCAN"); library(isoSCAN); gly_rows <- summed_iso_table[summed_iso_table$CompoundName == "Gly", ]; metBarPlot(gly_rows)
```

## Evaluation signals

- sumIsotopologues returns one aggregated isotopologue summary per metabolite, with abundances summing consistently from the underlying autoQ isotope-integration rows
- Only high-resolution autoQ tables are fed in — confirm the source instrument was GC-(q)TOF or GC-Orbitrap, not the Agilent 7890A/7000 QqQ SIM/NS setup
- The filtered rows (e.g. CompoundName == 'Gly') exist in the summed table and carry all sample groups expected in the study
- metBarPlot may warn 'Not quantified' for groups without quantified values; presence of these warnings matches the vignette's documented example figure and is not an error
- The saved figure file (e.g. gly_metBarPlot.png) exists and visually shows isotopologue abundance bar heights per sample group

## Limitations

- sumIsotopologues is only applicable to high-resolution autoQ results; low-resolution (LRMS) data are not supported for this aggregation step
- metBarPlot emits 'Not quantified' warnings for sample groups lacking quantified values — treat these as expected, not as processing failures
- The tool processes GC-CI-MS data specifically; it is not a general-purpose MS isotope quantification package for other acquisition types

## Evidence

- [other] vignette demonstrates applying sumIsotopologues to the autoQ integrations (noted as only for high-res data) and then calling metBarPlot: [evidence span withheld — anchor status: not_found]
- [intro] we have developed an R-package called isoSCAN capable of processing GC-CI-MS data from LRMS and HRMS stable-isotope labeling studies: [evidence span withheld — anchor status: not_found]
- [methods] A 7890B GC system coupled to a 7250 QTOF mass spectrometer (Agilent Technologies) was used: [evidence span withheld — anchor status: not_found]
- [results] isoSCAN, an open-source R-based computational tool: [evidence span withheld — anchor status: not_found]
