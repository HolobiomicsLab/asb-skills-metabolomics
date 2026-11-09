---
name: multiplexed-injection-peak-spacing-analysis
description: Use when annotating peaks in multiplexed MSI-CE-MS data where serial
  injections produce evenly spaced peak segments, and MTI-predicted migration times
  need confirmation or reanalysis of mis-annotated segments.
when_to_use_negative:
- Data are single-injection (non-multiplexed) CE-MS runs, where there is no consistent
  inter-peak spacing to exploit
- Input is already a processed feature/peak table rather than raw mzML electropherograms
  needing annotation
- Separations rely on time-warping or effective mobility alignment approaches, which
  PeakMeister deliberately avoids
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: PeakMeister
  role: R software that applies the peak-space tolerance check to confirm/adjust multiplexed
    segment annotations after MTI-based migration time prediction
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which PeakMeister (and its spacing-based reanalysis)
    is executed
- name: MSI-CE-MS
  role: Acquisition platform whose serial injections create the equally spaced peak
    segments this skill exploits
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:397680a6f876fccc53f77bf83d7737220b989280fba58ee7f67509a335df27cf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  repo_url: https://github.com/PBM-Group/PeakMeister
  tools:
  - MSI-CE-MS
  - PeakMeister
  - R
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# multiplexed-injection-peak-spacing-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Uses the characteristic, approximately equal spacing between analytical peaks in multiplexed multisegment injection-CE-MS electropherograms to confirm and adjust automated peak annotations, as implemented via the peak.space.tolerance.percent parameter in PeakMeister.

## When to use

Use when annotating peaks in multiplexed MSI-CE-MS data where serial injections produce evenly spaced peak segments, and MTI-predicted migration times need confirmation or reanalysis of mis-annotated segments.

## When NOT to use

- Data are single-injection (non-multiplexed) CE-MS runs, where there is no consistent inter-peak spacing to exploit
- Input is already a processed feature/peak table rather than raw mzML electropherograms needing annotation
- Separations rely on time-warping or effective mobility alignment approaches, which PeakMeister deliberately avoids

## Inputs

- Extracted ion electropherograms (EIEs) from mzML files of MSI-CE-MS runs
- MTI table of predicted/expected peak migration times for the 47 serum metabolites
- Mass List and Parameters.xlsx settings including peak.space.tolerance.percent, number.of.injections, and migration.window.seconds

## Outputs

- Confirmed or adjusted peak annotations for each multiplexed segment
- Table of migration times of peaks or expected peak positions
- Plots of each extracted ion electropherogram for visual verification of annotation and integration

## How to apply

After PeakMeister computes migration time indexes from the 11 internal standards and predicts analyte migration times within the migration.window.seconds window, check whether detected peaks deviate from the median inter-peak spacing by no more than peak.space.tolerance.percent. Segments whose spacing exceeds this tolerance are flagged and reanalyzed by PeakMeister to correct the multiplexed segment annotation. This spacing check exploits the fact that gaps between peaks in MSI-CE-MS experiments are typically consistent because serial injections occur at fixed intervals. Tune the tolerance to the expected peak widths and injection schedule so genuine drift is tolerated but mis-segmentation triggers reanalysis.

## Related tools

- **PeakMeister** (R software that applies the peak-space tolerance check to confirm/adjust multiplexed segment annotations after MTI-based migration time prediction) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which PeakMeister (and its spacing-based reanalysis) is executed)
- **MSI-CE-MS** (Acquisition platform whose serial injections create the equally spaced peak segments this skill exploits)

## Examples

```
Rscript code  # open the PeakMeister project file, set peak.space.tolerance.percent in 'Mass List and Parameters.xlsx', then execute the 'code' script
```

## Evaluation signals

- Median peak annotation accuracy across the 47 serum metabolites exceeds 99.9% after spacing-based confirmation
- Flagged segments (spacing deviation > peak.space.tolerance.percent) are successfully reanalyzed and re-annotated by PeakMeister
- Predicted vs. observed migration times give a median MAE of ~2.9 s versus ~105 s without MT correction
- Peak spacing between multiplexed segments deviates from the median by no more than the configured tolerance percent in correctly annotated runs
- Peak-area outputs for 16 NIST SRM-1950 plasma metabolites agree with manual preprocessing with a mean bias of -1.3%

## Limitations

- Tolerance must be tuned to peak widths; too tight a value triggers unnecessary reanalysis, too loose a value misses mis-segmentation
- Late-migrating serum metabolites lacking two flanking or a stable-isotope internal standard were prone to peak misannotation even with spacing checks
- Low-abundance metabolites with signals near the method detection limit remain prone to misannotation
- PeakMeister is validated only for serum/plasma matrices, not urine, saliva, or stool

## Evidence

- [other] Use the inter-peak spacing check (peak.space.tolerance.percent) to confirm/adjust annotations of the multiplexed segments, then export the MTI table and the table of predicted/expected peak migration times.: [evidence span withheld — anchor status: not_found]
