---
name: retention-time-scheduled-method-export
description: Use when a second-round targeted MS method (SRM/MRM) must be built from
  measured retention times observed in a first-pass Skyline analysis, e.g. refining
  an arginine/polyamine pathway assay after initial unscheduled acquisition.
when_to_use_negative:
- The input is untargeted MS1 feature data needing peak finding and alignment (use
  XCMS, MZmine, or MS-DIAL instead of Skyline's targeted extraction paradigm)
- The assay is being developed for the first time with no prior measured retention
  times — an unscheduled method must be run before scheduling is possible
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Skyline
  role: GUI tool for importing the transition list, enabling collision energy optimization
    under Settings->Transition Settings->Prediction, and exporting the retention-time-scheduled
    method from measured RTs
  repo: github.com/ProteoWizard/pwiz
- name: Panorama
  role: Web-based repository for sharing the resulting Skyline method and document
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:59dc0f5d537564d7ec9e6e34235d69fc7ccea0bedbe2146bc0df6dba978962b9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Panorama
  - Skyline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Retention-time-scheduled method export

## Summary

Export a Skyline targeted small-molecule acquisition method with retention time scheduling based on measured retention times from a prior data analysis, using a 1-minute scheduling window centered on those times to concentrate duty cycle on expected elution windows.

## When to use

Use when a second-round targeted MS method (SRM/MRM) must be built from measured retention times observed in a first-pass Skyline analysis, e.g. refining an arginine/polyamine pathway assay after initial unscheduled acquisition.

## When NOT to use

- The input is untargeted MS1 feature data needing peak finding and alignment (use XCMS, MZmine, or MS-DIAL instead of Skyline's targeted extraction paradigm)
- The assay is being developed for the first time with no prior measured retention times — an unscheduled method must be run before scheduling is possible

## Inputs

- Skyline small-molecule document with imported transition list (m/z, formula, adducts) and heavy labeled internal standards
- Prior-round Skyline analysis results with measured retention times for each target (e.g. ADNI p180 vendor raw files)
- Confirmed peak integration decisions (automatic plus manual boundary adjustments)

## Outputs

- Retention-time-scheduled transition list or native instrument method (1-minute window centered on measured RTs)
- Assembled Skyline document (e.g. small_molecule_workflow_template.sky)

## How to apply

First import the transition list of targets (with m/z, formula, adducts) plus heavy labeled internal standards into a Skyline small-molecule document and acquire/analyze an initial run, inspecting each target as an extracted ion chromatogram and adjusting integration boundaries where automatic peak picking is unsatisfactory. Read the measured retention times of the confirmed peaks from that prior analysis. Optionally enable collision energy optimization for the transitions via Settings -> Transition Settings -> Prediction before exporting. Then export the method with retention time scheduling based on the measured retention times and a time window setting of 1-minute centered on these times, so the instrument only monitors each transition near its expected elution. Save the assembled document (e.g. small_molecule_workflow_template.sky) and share it via Panorama if desired; the output is a vendor-transition-list or native method ready for the scheduled acquisition round.

## Related tools

- **Skyline** (GUI tool for importing the transition list, enabling collision energy optimization under Settings->Transition Settings->Prediction, and exporting the retention-time-scheduled method from measured RTs) — github.com/ProteoWizard/pwiz
- **Panorama** (Web-based repository for sharing the resulting Skyline method and document) — https://panoramaweb.org/

## Evaluation signals

- Every target in the exported method has a scheduled window of 1 minute centered on its measured retention time from the prior analysis
- The exported method contains the same transitions (quantifier plus heavy internal standards) as the imported transition list
- Collision energy values are optimized/enabled when the Prediction tab option was selected before export
- The saved .sky document re-opens with peak boundaries matching the manual integration adjustments made on the ADNI p180 chromatograms

## Limitations

- Skyline may lag behind commercial vendor software for latest-generation or vendor-specific workflows until support is implemented
- No single tool will be best for all workflows; GC-MS and ion mobility small molecule data support are still ongoing expansions
- Fragmentation library workflows for METLIN, MassBank, HMDB, and NIST were not yet implemented at the time of the paper

## Evidence
