---
name: dd-fragmentation-strategy-prototyping
description: Use when designing or tuning an MS2 fragmentation strategy (ROI-based
  DDA controllers, TopN baselines, duty cycles) for LC-MS/MS metabolomics and you
  need in silico simulation of MS1/MS2 schedules on full-scan mzML data.
when_to_use_negative:
- Input is already acquired MS2 data or a feature table — this skill simulates and
  schedules acquisitions, it does not analyze existing spectra
- The target strategy is DIA rather than DDA — the article addresses improving DDA
  coverage (proportion of ions fragmented) for a single injection, not DIA method
  design
- You need validated biological identifications rather than acquisition-schedule development;
  this skill evaluates coverage of picked peaks, not compound annotation
edam_operation: http://edamontology.org/operation_0438
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to simulate MS1/MS2 scans, seed
    ROI tracking, and execute the DDA controller to produce a simulated mzML schedule
- name: IAPI
  role: Instrument Application Programming Interface used to apply strategies developed
    in silico to real samples on actual hardware
- name: MZmine 2
  role: Peak picking with bounding boxes on the simulated output to compute fragmentation
    coverage against the TopN baseline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:afa0195d012c4291005687877be12c56735c8e4573015596c866eb0d4cfc9117
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IAPI
  - MZmine 2
  - ViMMS
  edam_operation: http://edamontology.org/operation_0438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# dd-fragmentation-strategy-prototyping

## Summary

Prototype, optimize, and validate data-dependent acquisition (DDA) ion prioritization strategies (e.g., SmartROI) in silico using the ViMMS virtual metabolomics mass spectrometer before committing to expensive instrument time on real samples.

## When to use

Use when designing or tuning an MS2 fragmentation strategy (ROI-based DDA controllers, TopN baselines, duty cycles) for LC-MS/MS metabolomics and you need in silico simulation of MS1/MS2 schedules on full-scan mzML data.

## When NOT to use

- Input is already acquired MS2 data or a feature table — this skill simulates and schedules acquisitions, it does not analyze existing spectra
- The target strategy is DIA rather than DDA — the article addresses improving DDA coverage (proportion of ions fragmented) for a single injection, not DIA method design
- You need validated biological identifications rather than acquisition-schedule development; this skill evaluates coverage of picked peaks, not compound annotation

## Inputs

- Full-scan mzML file of a complex metabolite mixture sample extract (MS1 survey scans)
- Controller specification (e.g., SmartROI parameters: min fragmentation intensity, alpha = 1000, beta = 0.1, N MS2 scans per MS1 duty cycle)
- Baseline controller definition (standard TopN) for comparison

## Outputs

- Simulated acquisition schedule exported as an mzML file (scan start times, MS1/MS2 events)
- Fragmentation coverage statistics: picked peaks (MZmine 2 bounding boxes) containing a fragmentation event, unique peaks fragmented, MS2 scan count vs. TopN baseline

## How to apply

Load a full-scan mzML file of the sample extract to seed the ViMMS virtual metabolomics mass spectrometer, then implement the DDA controller (e.g., SmartROI) so that as each simulated MS1 survey scan arrives the region-of-interest (ROI) set is updated and ROIs not extended by the scan are discarded as inactive. Apply ROI availability rules before scheduling fragmentation: intensity >= the minimum fragmentation intensity in the latest survey scan; unfragmented ROIs are always available, whereas previously fragmented ROIs are available only if intensity has risen by a factor alpha = 1000 since last fragmentation or dropped by a factor beta = 0.1 from its highest value since last fragmentation. Prioritize active, available ROIs by intensity and schedule the duty cycle of one MS1 scan followed by up to N MS2 fragmentation scans, then export the resulting scan schedule (scan start times, MS1/MS2 events) as an mzML file. Rationale: DDA based on MS1 survey intensities typically fragments only a small subset of ions present, and the paper shows ROI-based prioritization fragments more unique peaks with fewer, more precisely targeted MS2 scans.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to simulate MS1/MS2 scans, seed ROI tracking, and execute the DDA controller to produce a simulated mzML schedule)
- **IAPI** (Instrument Application Programming Interface used to apply strategies developed in silico to real samples on actual hardware)
- **MZmine 2** (Peak picking with bounding boxes on the simulated output to compute fragmentation coverage against the TopN baseline)

## Evaluation signals

- SmartROI simulated schedule's coverage of picked peaks (peaks whose MZmine 2 bounding box contains a fragmentation event) exceeds the TopN controller's on the same sample
- SmartROI fragments more unique peaks than TopN while issuing fewer, more precisely targeted MS2 scans
- Inactive ROIs are correctly discarded: every scheduled fragmentation ROI was extended by the latest survey scans and met the availability rules (min intensity, alpha = 1000 rise or beta = 0.1 drop)
- Exported mzML preserves the duty-cycle structure: one MS1 scan followed by up to N MS2 fragmentation scans
- Optimized strategy transfers: when deployed on real samples via IAPI, behavior matches the in silico simulation rather than degrading

## Limitations

- Simulated results depend on the fidelity of the seeded mzML sample extract; strategies may behave differently on hardware, so real-sample validation via IAPI is needed
- ROI availability thresholds (alpha = 1000, beta = 0.1, minimum fragmentation intensity) are strategy parameters that require optimization per sample; defaults may not generalize
- The theoretical optimum is sample-dependent, taking into account the uneven elution distribution of the ions, so improvement margins vary between injections

## Evidence

- [abstract] an in silico framework for fast and cost-efficient development of new DDA strategies: "an in silico framework for fast and cost-efficient development of new DDA strategies"
- [abstract] Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans"
- [abstract] using a previously developed virtual metabolomics mass spectrometer (ViMMS): "using a previously developed virtual metabolomics mass spectrometer (ViMMS)"
- [abstract] on real samples via an Instrument Application Programming Interface (IAPI): "on real samples via an Instrument Application Programming Interface (IAPI)"
- [other] an ROI must have intensity >= the minimum fragmentation intensity in the latest survey scan: [evidence span withheld — anchor status: not_found]
- [other] Evaluate the schedule by picking peaks with MZmine 2 bounding boxes and computing coverage (picked peaks containing a fragmentation event) against the TopN baseline: [evidence span withheld — anchor status: not_found]
