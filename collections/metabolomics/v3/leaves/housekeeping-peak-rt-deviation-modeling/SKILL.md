---
name: housekeeping-peak-rt-deviation-modeling
description: Use when targeted LC–MS peak extraction shows run-to-run retention time
  drift and QC runs with annotated housekeeping compounds are available to model per-run
  RT deviations.
when_to_use_negative:
- The dataset has no QC runs or no annotated housekeeping compounds, so no RT-deviation
  model can be fit
- The input is already an RT-aligned feature table and only downstream statistics
  are needed
- The goal is untargeted peak correspondence across samples (TARDIS is designed for
  targeted peak extraction and integration of LC–MS data)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TARDIS
  role: R package performing targeted raw data integration; extracts housekeeping
    peak RTs in QC runs and applies the fitted RT-deviation models to adjust target
    RTs
  repo: https://github.com/UGent-LIMET/TARDIS
- name: xcms
  role: Supplies PeakGroupsParam, used to model RT deviation of housekeeping compounds
    (linear or polynomial) and correct RTs
- name: R
  role: Runtime environment (>= 4.4.0) in which TARDIS and xcms are executed; outputs
    rt_alignment.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:a3763491a9b427161286edf39a549e06e0727002984ed12de831a045b839e07c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# housekeeping-peak-rt-deviation-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Corrects run-to-run retention time (RT) shifts during targeted LC–MS preprocessing by fitting linear or polynomial RT-deviation models to user-selected housekeeping compounds across QC runs (via xcms' PeakGroupsParam, as used inside TARDIS' tardisPeaks()), then applying the fitted models to adjust each target's expected RT in every run.

## When to use

Use when targeted LC–MS peak extraction shows run-to-run retention time drift and QC runs with annotated housekeeping compounds are available to model per-run RT deviations.

## When NOT to use

- The dataset has no QC runs or no annotated housekeeping compounds, so no RT-deviation model can be fit
- The input is already an RT-aligned feature table and only downstream statistics are needed
- The goal is untargeted peak correspondence across samples (TARDIS is designed for targeted peak extraction and integration of LC–MS data)

## Inputs

- Raw LC–MS runs (e.g., the TARDIS quick-start vignette runs) including QC runs
- Target list annotating target compounds and user-selected housekeeping peaks
- QC run annotations identifying which runs serve as QC

## Outputs

- rt_alignment.csv containing housekeeping RT deviations and per-target adjusted RTs for every run
- TARDIS results list containing a data.frame with the AUC of each target in each run and a tibble feature table with average metrics per target in QC runs

## How to apply

Within the TARDIS quick-start workflow (R >= 4.4.0, package installed via BiocManager from GitHub), load the LC–MS runs and the target list containing housekeeping peaks, then run tardisPeaks() to extract the retention times of the selected housekeeping peaks in the QC runs and compute their RT deviations relative to the reference. Fit an RT-deviation model to the housekeeping peak deviations across QC runs using xcms' PeakGroupsParam: choose a linear or polynomial model based on the number of housekeeping compounds available. Apply the fitted model to adjust each target's expected RT in every run, then write a table of housekeeping RT deviations and per-target adjusted RTs to rt_alignment.csv. Because TARDIS also produces a feature table with average metrics per target in the QC runs, use it to sanity-check that housekeeping peaks were picked consistently before trusting the fitted deviation model.

## Related tools

- **TARDIS** (R package performing targeted raw data integration; extracts housekeeping peak RTs in QC runs and applies the fitted RT-deviation models to adjust target RTs) — https://github.com/UGent-LIMET/TARDIS
- **xcms** (Supplies PeakGroupsParam, used to model RT deviation of housekeeping compounds (linear or polynomial) and correct RTs)
- **R** (Runtime environment (>= 4.4.0) in which TARDIS and xcms are executed; outputs rt_alignment.csv)

## Examples

```
RShowDoc("quick_start",package = "TARDIS")
```

## Evaluation signals

- rt_alignment.csv exists after the tardisPeaks() run and contains deviation and adjusted-RT fields for every run and every target
- RT corrections are plausibly small — expert review of chromatographic stability should show housekeeping RT deviations drifting smoothly across QC runs
- The linear vs. polynomial model choice is consistent with the number of housekeeping compounds available
- Adjusted RTs place extracted EIC windows on real peaks, evidenced by the AUC data.frame in the results list containing non-zero values for targets in each run
- The QC-run feature table shows consistent average metrics for housekeeping targets, confirming the deviations were computed from reliably picked peaks

## Limitations

- Model complexity (linear vs. polynomial) is constrained by the number of user-selected housekeeping compounds, so few housekeepers force simpler models and may under-correct nonlinear drift
- Correction quality depends on reliable picking of housekeeping peaks in QC runs; poor housekeeping peak quality will propagate into all adjusted target RTs
- The paper reports no explicit version number for the TARDIS R package, so exact behavior may differ across package versions

## Evidence

- [readme] Quick-start vignette is the documented entry point: [evidence span withheld — anchor status: external_doc]
