---
name: retention-time-drift-alignment
description: Use when targeted LC–MS peak integration shows RT shifts between runs
  and you have QC runs plus a target list containing housekeeping peaks whose RT deviations
  can anchor the alignment.
when_to_use_negative:
- Data are untargeted feature tables already preprocessed by tools like xcms, MZmine
  or OpenMS rather than raw targeted LC–MS runs
- No QC runs or no housekeeping peaks are present in the target list, leaving no anchors
  for the RT-deviation model
- The goal is untargeted RT alignment of features (e.g. MZmine/OpenMS alignment) rather
  than adjusting expected RTs of a fixed target list
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: TARDIS
  role: R package whose tardisPeaks() performs targeted peak preprocessing including
    RT alignment via xcms PeakGroupsParam; writes rt_alignment.csv
  repo: https://github.com/UGent-LIMET/TARDIS
- name: xcms
  role: Underlying preprocessing engine whose PeakGroupsParam fits linear/polynomial
    RT-deviation models from housekeeping compounds
- name: R
  role: Runtime environment (version >= 4.4.0) in which TARDIS and the vignette workflow
    are executed
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:54590b6f46d7350d37b54b7de24a08f264c59347c13d55d9667eb844560ea545
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/UGent-LIMET/TARDIS
  repo_url: https://github.com/UGent-LIMET/TARDIS
  tools:
  - R
  - TARDIS
  - xcms
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# retention-time-drift-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Corrects retention time (RT) drift between LC–MS runs during targeted peak preprocessing by fitting RT-deviation models of user-selected housekeeping compounds across QC runs with xcms' PeakGroupsParam, as implemented in TARDIS' tardisPeaks().

## When to use

Use when targeted LC–MS peak integration shows RT shifts between runs and you have QC runs plus a target list containing housekeeping peaks whose RT deviations can anchor the alignment.

## When NOT to use

- Data are untargeted feature tables already preprocessed by tools like xcms, MZmine or OpenMS rather than raw targeted LC–MS runs
- No QC runs or no housekeeping peaks are present in the target list, leaving no anchors for the RT-deviation model
- The goal is untargeted RT alignment of features (e.g. MZmine/OpenMS alignment) rather than adjusting expected RTs of a fixed target list

## Inputs

- Raw LC–MS runs loaded in R via TARDIS' quick-start vignette workflow (QC runs plus sample runs)
- Target list containing targets and user-selected housekeeping peaks
- xcms PeakGroupsParam settings (housekeeping compound set; linear vs polynomial model choice)

## Outputs

- rt_alignment.csv containing housekeeping RT deviations and per-target adjusted RTs for every run
- RT-corrected expected RTs used by tardisPeaks() for targeted peak extraction/integration

## How to apply

Within tardisPeaks(), extract the retention times of user-selected housekeeping peaks in the QC runs and compute their RT deviations relative to a reference run. Fit an RT-deviation model to these deviations using xcms' PeakGroupsParam: a linear model is chosen when few housekeeping compounds are available, while a polynomial model is used when more are available (model choice is governed by the number of housekeeping compounds). Apply the fitted model to adjust each target's expected RT in every run so integration windows track the drifted peaks. Write the housekeeping RT deviations and per-target adjusted RTs to rt_alignment.csv. Rationale: chromatographic drift between runs would otherwise cause TARDIS' targeted integration to miss or mis-integrate peaks; anchoring on stable housekeeping compounds measured in every QC run provides a run-specific RT correction curve.

## Related tools

- **TARDIS** (R package whose tardisPeaks() performs targeted peak preprocessing including RT alignment via xcms PeakGroupsParam; writes rt_alignment.csv) — https://github.com/UGent-LIMET/TARDIS
- **xcms** (Underlying preprocessing engine whose PeakGroupsParam fits linear/polynomial RT-deviation models from housekeeping compounds)
- **R** (Runtime environment (version >= 4.4.0) in which TARDIS and the vignette workflow are executed) — https://cloud.r-project.org/index.html

## Examples

```
R> BiocManager::install('pablovgd/TARDIS'); library(TARDIS); results <- tardisPeaks(...)  # see quick_start vignette: RShowDoc('quick_start', package='TARDIS')
```

## Evaluation signals

- rt_alignment.csv exists and contains deviation and adjusted-RT fields for every run and every target
- Fitted RT corrections are plausibly small, consistent with expert review of chromatographic stability
- The RT-deviation model type (linear vs polynomial) matches the number of available housekeeping compounds
- Housekeeping compound RT deviations across QC runs are computed relative to the reference and are stable across runs
- Post-alignment targeted integration recovers each target within the adjusted RT window in every run

## Limitations

- Model choice (linear vs polynomial) depends on the number of housekeeping compounds; too few anchors restrict alignment accuracy
- The skill is specific to TARDIS' targeted preprocessing of LC–MS data and requires QC runs with housekeeping peaks; it cannot compensate when the target list lacks stable anchors
- R >= 4.4.0 and BiocManager 3.20 are required, and example-data installation can hit connection timeouts (increase options(timeout = '300'))

## Evidence

- [results] RT alignment via PeakGroupsParam: [evidence span withheld — anchor status: not_found]
- [readme] Installation requirements from README: [evidence span withheld — anchor status: external_doc]
