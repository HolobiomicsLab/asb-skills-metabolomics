---
name: qc-run-reference-calibration
description: Use when targeted LC-MS runs show drift in retention time and expected
  RT windows for targets need adjusting before peak integration, using housekeeping/QC
  compounds as anchors.
when_to_use_negative:
- Data are already a feature/peak table with aligned retention times rather than raw
  LC-MS runs
- No housekeeping compounds are present in the target list or no QC runs exist in
  the batch, leaving no anchor points for RT deviation modeling
- The goal is untargeted preprocessing (e.g., xcms/MZmine/OpenMS-style feature finding)
  rather than targeted peak adjustment
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TARDIS
  role: R package that implements the RT alignment via xcms' PeakGroupsParam inside
    tardisPeaks() and emits the rt_alignment.csv table
  repo: https://github.com/UGent-LIMET/TARDIS
- name: xcms
  role: Provides PeakGroupsParam, the alignment engine whose housekeeping-compound
    RT deviation models are used by TARDIS
- name: R
  role: Runtime environment (version >= 4.4.0 required) in which tardisPeaks() and
    the RT alignment are executed
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:03d669deb213eb860169bc94490861ee0a8dd80047a704338d44b5d05e3e6a10
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
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# qc-run-reference-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retention-time alignment of targeted LC-MS runs by fitting a linear or polynomial model to the RT deviations of user-selected housekeeping peaks measured across QC runs, then applying that model to adjust each target's expected RT in every run — the TARDIS approach built on xcms' PeakGroupsParam.

## When to use

Use when targeted LC-MS runs show drift in retention time and expected RT windows for targets need adjusting before peak integration, using housekeeping/QC compounds as anchors.

## When NOT to use

- Data are already a feature/peak table with aligned retention times rather than raw LC-MS runs
- No housekeeping compounds are present in the target list or no QC runs exist in the batch, leaving no anchor points for RT deviation modeling
- The goal is untargeted preprocessing (e.g., xcms/MZmine/OpenMS-style feature finding) rather than targeted peak adjustment

## Inputs

- Raw LC-MS runs (e.g., the TARDIS quick-start vignette LC-MS runs)
- Target list including user-selected housekeeping peaks
- Identification of QC runs within the run sequence

## Outputs

- rt_alignment.csv containing housekeeping RT deviations and per-target adjusted RTs for every run and target
- Adjusted expected RTs used for subsequent targeted peak integration

## How to apply

Run tardisPeaks() with the target list containing user-selected housekeeping peaks, which extracts the retention time of the housekeeping compounds in each QC run via xcms' PeakGroupsParam and computes their RT deviations relative to the reference run. Choose a linear versus polynomial model based on the number of housekeeping compounds available — the article states the model is 'chosen by the number of housekeeping compounds'. Fit the RT-deviation model to the housekeeping peak deviations across QC runs, then apply the fitted model to adjust each target's expected RT in every run. Write the housekeeping RT deviations and per-target adjusted RTs to an rt_alignment.csv table for inspection. The rationale is that chromatographic shifts between injections would otherwise shift targets out of integration windows, and stable endogenous housekeeping peaks track that shift sample-by-sample.

## Related tools

- **TARDIS** (R package that implements the RT alignment via xcms' PeakGroupsParam inside tardisPeaks() and emits the rt_alignment.csv table) — https://github.com/UGent-LIMET/TARDIS
- **xcms** (Provides PeakGroupsParam, the alignment engine whose housekeeping-compound RT deviation models are used by TARDIS)
- **R** (Runtime environment (version >= 4.4.0 required) in which tardisPeaks() and the RT alignment are executed) — https://cloud.r-project.org/index.html

## Examples

```
BiocManager::install("pablovgd/TARDIS",build_vignettes = FALSE, dependencies=TRUE)
```

## Evaluation signals

- rt_alignment.csv exists and contains deviation and adjusted-RT fields for every run and every target
- Adjusted RTs are plausibly close to reference RTs (expert review of chromatographic stability — corrections should be small)
- The number of housekeeping peaks used matches the chosen model family (linear vs polynomial) as selected by housekeeping compound count
- Downstream peak integration succeeds with the adjusted RTs, yielding per-target AUC in the results data.frame

## Limitations

- Model choice (linear vs polynomial) is constrained by the number of housekeeping compounds available; too few anchors limits flexibility
- Alignment quality depends on the chromatographic stability of the chosen housekeeping peaks; if those peaks themselves shift unpredictably, per-target adjustments become unreliable
- The article does not report explicit limits of the alignment, only that rigorous testing on biological data (FAME, ENVIRONAGE, FGFP cohorts) confirmed applicability at large scale (n = 1786)

## Evidence

- [methods] TARDIS aligns RTs via xcms' PeakGroupsParam: [evidence span withheld — anchor status: not_found]
- [methods] these models are subsequently used to adjust the RT of each target: [evidence span withheld — anchor status: not_found]
- [discussion] Rigorous testing using biological data confirmed that TARDIS is applicable in large-scale LC-MS experiments: [evidence span withheld — anchor status: not_found]
