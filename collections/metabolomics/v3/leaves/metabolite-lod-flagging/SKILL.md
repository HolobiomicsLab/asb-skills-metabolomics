---
name: metabolite-lod-flagging
description: Use when pre-processing targeted full-scan MSI-CE-MS data in PeakMeister
  and you need to distinguish true analyte peaks from near-noise signals, particularly
  for low-abundance serum metabolites whose signal sits near the method detection
  limit.
when_to_use_negative:
- The analyte peaks are strong, well-resolved reference-standard signals — snr.threshold
  is meant for borderline analyte signals, not landmark peaks used for migration-index
  calculation, which require reliable and always-the-most-intense peak areas.
- Input is not targeted full-scan MSI-CE-MS data (e.g. HILIC LC-MS or non-multiplexed
  CE-MS runs); PeakMeister's peak-space and migration-index logic does not apply.
- The mzML files have not yet been peak-picked and annotated — <LOD flagging operates
  on identified analyte peaks, not raw extracted-ion traces.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakMeister
  role: Computes per-peak S/N on smoothed EIEs and applies snr.threshold to record
    analytes as "<LOD" in the peak-area output
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Runtime environment in which PeakMeister (and thus the LOD-flagging logic
    via ksmooth-based smoothing) executes
  repo: https://stat.ethz.ch/R-manual/R-devel/library/stats/html/ksmooth.html
- name: mzML
  role: Input file format that PeakMeister directly reads to extract EIEs for S/N-based
    LOD flagging
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:d2d49382404558978b25ad65e6a96b3176f59164f9840545b1a2890b4d4c1225
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
  - mzML
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metabolite-lod-flagging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Flag metabolite signals as below the limit of detection ("<LOD") within PeakMeister by computing the signal-to-noise ratio of each annotated analyte peak and applying the snr.threshold parameter, ensuring low-abundance MSI-CE-MS serum metabolites are not spuriously integrated or quantified.

## When to use

Use when pre-processing targeted full-scan MSI-CE-MS data in PeakMeister and you need to distinguish true analyte peaks from near-noise signals, particularly for low-abundance serum metabolites whose signal sits near the method detection limit.

## When NOT to use

- The analyte peaks are strong, well-resolved reference-standard signals — snr.threshold is meant for borderline analyte signals, not landmark peaks used for migration-index calculation, which require reliable and always-the-most-intense peak areas.
- Input is not targeted full-scan MSI-CE-MS data (e.g. HILIC LC-MS or non-multiplexed CE-MS runs); PeakMeister's peak-space and migration-index logic does not apply.
- The mzML files have not yet been peak-picked and annotated — <LOD flagging operates on identified analyte peaks, not raw extracted-ion traces.

## Inputs

- Annotated MSI-CE-MS extracted ion electropherograms (EIEs) from PeakMeister peak picking
- Mass List and Parameters.xlsx (Sheet 1) with per-analyte mz, extraction.window.ppm, and snr.threshold
- mzML files in the 'mzML Files' folder (already converted and loaded into the PeakMeister R script)

## Outputs

- Peak-area table in which sub-threshold analyte signals are encoded as "<LOD" instead of numeric area
- Plots of each extracted ion electropherogram showing which peaks were flagged <LOD

## How to apply

After PeakMeister has identified which extracted ion electropherogram signals correspond to the analyte peaks, compute each peak's signal-to-noise ratio using the smoothing parameters already applied to the EIE (smoothing.kernal via the stats::ksmooth function, with a chosen smoothing.strength bandwidth). Compare the computed S/N against the user-defined snr.threshold from the 'Mass List and Parameters.xlsx' Sheet 1; any analyte whose S/N falls below this threshold is recorded as "<LOD" in the peak-area output table rather than being integrated as a real measurement. This flagging step works in tandem with minimum.peak.width.seconds (which filters noise during peak detection before annotation) — LOD flagging catches the residual low-abundance peaks that survive width filtering but are too weak to quantify. The flag propagates downstream, e.g. excluding such samples when computing metabolite reference intervals from peak-area distributions in R.

## Related tools

- **PeakMeister** (Computes per-peak S/N on smoothed EIEs and applies snr.threshold to record analytes as "<LOD" in the peak-area output) — https://github.com/PBM-Group/PeakMeister
- **R** (Runtime environment in which PeakMeister (and thus the LOD-flagging logic via ksmooth-based smoothing) executes) — https://stat.ethz.ch/R-manual/R-devel/library/stats/html/ksmooth.html
- **mzML** (Input file format that PeakMeister directly reads to extract EIEs for S/N-based LOD flagging)

## Examples

```
Rscript code  # open the PeakMeister project file; snr.threshold is set in 'Mass List and Parameters.xlsx' Sheet 1
```

## Evaluation signals

- Peak-area table contains no numeric area for analytes whose computed S/N is below the user-set snr.threshold — they must appear as the literal string "<LOD"
- Reference-standard / landmark peaks (Sheet 2 'Reference' class) are never flagged <LOD; their peak areas must remain numeric
- Spot-check flagged peaks against the saved extracted ion electropherogram plots — the flagged peak should be visibly at or below the noise floor of the smoothed EIE
- Downstream metabolite quantification (e.g. NIST SRM-1950 16-plasma-metabolite comparison, or ENANI-2019 reference intervals) must exclude <LOD entries rather than treating them as zeros or NAs
- The snr.threshold value used matches what was recorded in the saved copy of processing parameters (PeakMeister output artifact #2)

## Limitations

- Low abundance metabolites with signals near the method detection limit are inherently prone to peak misannotation, so <LOD flagging cannot recover peaks that were mis-annotated in the first place.
- The snr.threshold is a user-set parameter in Sheet 1 of the Mass List and Parameters.xlsx template — an inappropriate threshold will either flag true peaks as <LOD or integrate noise.
- LOD flagging is applied per-analyte peak, not globally; it does not substitute for the minimum.peak.width.seconds noise filter during peak detection.

## Evidence
