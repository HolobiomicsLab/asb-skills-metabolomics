---
name: dda-duty-cycle-scheduling
description: 'Use when prototyping or simulating a DDA acquisition strategy for untargeted
  LC-MS/MS metabolomics: full-scan mzML input, a duty cycle to define, and a goal
  of improving coverage of picked peaks over a TopN baseline.'
when_to_use_negative:
- The input is already an acquired MS2 dataset or a feature table — this skill schedules
  acquisition, it does not re-analyze existing spectra.
- The goal is DIA (data-independent) acquisition rather than DDA ion prioritization;
  the article notes the DDA-vs-DIA question is out of scope for its DDA coverage objective.
- Real instrument timing without an instrument API is required — in silico ViMMS schedules
  must be validated separately on real hardware via IAPI.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to simulate MS1 survey scans,
    apply the SmartROI controller and duty-cycle rules, and export the resulting schedule
    as mzML.
- name: IAPI
  role: Instrument Application Programming Interface used to apply strategies developed
    in silico on real samples.
- name: MZmine 2
  role: Peak picking (bounding boxes) on the simulated schedule's output to compute
    coverage of picked peaks that contain a fragmentation event.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:bb842eebedecff7e27ede960fe3e080eb26e060ed975e1ede997fb15dfe4ca69
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IAPI
  - MZmine 2
  - ViMMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# dda-duty-cycle-scheduling

## Summary

Schedules data-dependent acquisition duty cycles in which one MS1 survey scan is followed by up to N MS2 fragmentation scans, using ROI-based ion prioritization (e.g., SmartROI availability rules) instead of naive TopN intensity ranking, so that more unique peaks get fragmented per injection.

## When to use

Use when prototyping or simulating a DDA acquisition strategy for untargeted LC-MS/MS metabolomics: full-scan mzML input, a duty cycle to define, and a goal of improving coverage of picked peaks over a TopN baseline.

## When NOT to use

- The input is already an acquired MS2 dataset or a feature table — this skill schedules acquisition, it does not re-analyze existing spectra.
- The goal is DIA (data-independent) acquisition rather than DDA ion prioritization; the article notes the DDA-vs-DIA question is out of scope for its DDA coverage objective.
- Real instrument timing without an instrument API is required — in silico ViMMS schedules must be validated separately on real hardware via IAPI.

## Inputs

- Full-scan mzML file of the sample extract (used to seed the ViMMS virtual metabolomics mass spectrometer)
- Duty-cycle parameters: number of MS2 scans per MS1 (N), minimum fragmentation intensity, alpha = 1000, beta = 0.1

## Outputs

- Simulated DDA scan schedule (scan start times, MS1/MS2 events) exported as an mzML file
- Fragmentation coverage comparison against the TopN controller baseline

## How to apply

Load the full-scan mzML of the sample extract into the ViMMS virtual metabolomics mass spectrometer. As each simulated MS1 survey scan arrives, update the regions of interest (ROIs) and discard ROIs not extended by the scan as inactive. Apply availability rules before targeting: an ROI must have intensity >= the minimum fragmentation intensity in the latest survey scan; unfragmented ROIs are always available, while previously fragmented ROIs are available only if intensity has risen by a factor alpha = 1000 since last fragmentation or dropped by a factor beta = 0.1 from its highest value since last fragmentation. Prioritize active, available ROIs by intensity and schedule the duty cycle of one MS1 scan followed by up to N MS2 fragmentation scans, then export the scan schedule (scan start times, MS1/MS2 events) as mzML.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to simulate MS1 survey scans, apply the SmartROI controller and duty-cycle rules, and export the resulting schedule as mzML.)
- **IAPI** (Instrument Application Programming Interface used to apply strategies developed in silico on real samples.)
- **MZmine 2** (Peak picking (bounding boxes) on the simulated schedule's output to compute coverage of picked peaks that contain a fragmentation event.)

## Evaluation signals

- Coverage of picked peaks (MZmine 2 bounding boxes containing a fragmentation event) exceeds the TopN controller's coverage on the same sample extract.
- The controller fragments more unique peaks while issuing fewer, more precisely targeted MS2 scans than TopN.
- Every scheduled MS2 event targets an active, available ROI whose intensity in the latest survey scan meets the minimum fragmentation intensity threshold.
- Previously fragmented ROIs are re-fragmented only when intensity rose by factor alpha = 1000 since last fragmentation or dropped by factor beta = 0.1 from its peak since last fragmentation.
- The exported mzML contains a valid duty cycle of one MS1 followed by up to N MS2 events.

## Limitations

- DDA typically only fragments a small subset of the ions present; ROI scheduling improves but does not guarantee full coverage, since many standard TopN MS2 scans target ions that would not be picked as peaks.
- In silico ViMMS results are simulation-based; strategies must be validated on real samples via IAPI before deployment.
- Availability thresholds (min intensity, alpha = 1000, beta = 0.1) are sample- and setup-dependent and may require re-optimization per dataset.

## Evidence

- [other] SmartROI tracks ROIs and availability rules: [evidence span withheld — anchor status: not_found]
- [abstract] DDA fragments only a subset of ions: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [abstract] ViMMS as the simulation framework: "using a previously developed virtual metabolomics mass spectrometer (ViMMS)"
- [abstract] In silico framework purpose: "an in silico framework for fast and cost-efficient development of new DDA strategies"
