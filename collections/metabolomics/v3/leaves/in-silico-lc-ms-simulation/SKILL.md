---
name: in-silico-lc-ms-simulation
description: Use when developing or comparing MS2 acquisition/DDA prioritization strategies
  on complex metabolite mixtures and you need to evaluate a scan schedule (coverage
  of picked peaks) in silico from a full-scan mzML before instrument time.
when_to_use_negative:
- The input is already an MS2 feature table or peak list — this skill simulates acquisition
  from full-scan MS1 data, not downstream annotation
- You need validated fragment spectra or structural identifications (e.g., SIRIUS4/CFM-ID
  searches) rather than acquisition-strategy comparison
- The question is DIA vs DDA benchmarking on already-collected real-data files; this
  skill is for prototyping new DDA controllers in silico
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer that consumes the full-scan mzML and
    executes the DDA controller (e.g., SmartROI) to emit a simulated mzML scan schedule
- name: MZmine 2
  role: Peak picking of bounding boxes on the simulated mzML to compute fragmentation
    coverage against the TopN baseline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:8710528ae8f346d705b9e9e3e4e67747cbf8b5b316a160d79c0081d45bd5e40b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine 2
  - ViMMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# in-silico-lc-ms-simulation

## Summary

Simulates LC-MS/MS acquisitions in the ViMMS virtual metabolomics mass spectrometer so that data-dependent acquisition (DDA) controller strategies (e.g., SmartROI with min-intensity, alpha/beta re-fragmentation rules) can be prototyped and validated on mzML full-scan input before touching real hardware.

## When to use

Use when developing or comparing MS2 acquisition/DDA prioritization strategies on complex metabolite mixtures and you need to evaluate a scan schedule (coverage of picked peaks) in silico from a full-scan mzML before instrument time.

## When NOT to use

- The input is already an MS2 feature table or peak list — this skill simulates acquisition from full-scan MS1 data, not downstream annotation
- You need validated fragment spectra or structural identifications (e.g., SIRIUS4/CFM-ID searches) rather than acquisition-strategy comparison
- The question is DIA vs DDA benchmarking on already-collected real-data files; this skill is for prototyping new DDA controllers in silico

## Inputs

- Full-scan mzML file of a complex metabolite sample extract (used to seed ViMMS)
- Controller specification/parameters (e.g., SmartROI: min_fragmentation_intensity, alpha = 1000, beta = 0.1, N MS2 scans per MS1)

## Outputs

- Simulated scan schedule exported as mzML (scan start times, MS1/MS2 events)
- Coverage table: MZmine 2 picked-peak bounding boxes annotated with whether they contain a fragmentation event, compared to the TopN baseline

## How to apply

Load the full-scan mzML of the sample extract to seed the ViMMS virtual metabolomics mass spectrometer, then implement the DDA controller (e.g., SmartROI) so that each simulated MS1 survey scan updates the regions of interest (ROIs) via the ROI algorithm and ROIs not extended by the scan are discarded as inactive. Apply availability rules: an ROI must have intensity >= the minimum fragmentation intensity in the latest survey scan; unfragmented ROIs are available, while previously fragmented ROIs re-qualify only if intensity rose by alpha = 1000 since last fragmentation or dropped by beta = 0.1 from its highest value since last fragmentation. Prioritize active, available ROIs by intensity and schedule the duty cycle of one MS1 scan followed by up to N MS2 fragmentation scans. Export the resulting schedule (scan start times, MS1/MS2 events) as mzML and evaluate by picking peaks with MZmine 2 bounding boxes, computing coverage (picked peaks containing a fragmentation event) against a TopN baseline.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer that consumes the full-scan mzML and executes the DDA controller (e.g., SmartROI) to emit a simulated mzML scan schedule)
- **MZmine 2** (Peak picking of bounding boxes on the simulated mzML to compute fragmentation coverage against the TopN baseline)

## Evaluation signals

- Coverage of MZmine 2 picked peaks (peaks whose bounding box contains a fragmentation event) exceeds the TopN controller's coverage on the same sample
- SmartROI produces fewer, more precisely targeted MS2 scans than TopN while fragmenting more unique peaks
- ROI lifecycle behaves per the rules: ROIs not extended by the incoming MS1 scan are discarded as inactive; re-fragmentation only when intensity rises by alpha = 1000 or falls to beta = 0.1 of its highest value since last fragmentation
- Duty cycle respected: one MS1 scan followed by up to N MS2 fragmentation scans
- Exported mzML contains scan start times and alternating MS1/MS2 events consistent with the controller decisions

## Limitations

- Results are in silico; the article notes validation on real hardware via an IAPI is needed before deployment on instruments
- DDA inherently 'typically only fragments a small subset of the ions present', so gains are bounded — the paper computes a theoretically optimal per-injection ceiling that no controller can exceed
- Coverage depends on MZmine 2 peak-picking quality; noisy or poorly eluting compounds can distort picked-peak bounding boxes and thus the comparison

## Evidence

- [abstract] Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans"
- [abstract] an in silico framework for fast and cost-efficient development of new DDA strategies: "an in silico framework for fast and cost-efficient development of new DDA strategies"
- [intro] new strategies can be prototyped, implemented, optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS): "new strategies can be prototyped, implemented, optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS)"
- [abstract] we first provide theoretical results that show how much improvement is possible over current DDA strategies: "we first provide theoretical results that show how much improvement is possible over current DDA strategies"
