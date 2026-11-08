---
name: ratio-of-change-curve-computation
description: Use when paired longitudinal DIA/DDA QC injections must be summarized
  as ratio-of-change curves around recorded maintenance-event timestamps to compare
  fault-detection sensitivity of DIA vs DDA metrics.
when_to_use_negative:
- Only a single file is available (no consecutive injections), so no ratio-of-change
  between injections can be computed
- Maintenance-event timestamps are unavailable or unrecorded for the instrument period,
  so curves cannot be aligned to fault events
- The goal is a qualified/unqualified AI classification of individual files rather
  than longitudinal sensitivity comparison of DIA vs DDA metrics
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: iDIA-QC
  role: GUI that exports the metric matrices and generates the HTML report plotting
    ratio-of-change curves around maintenance events
  repo: github.com/guomics-lab/iDIA-QC
- name: msConvert
  role: Converts vendor raw files (.raw/.d/.wiff) to mzXML from which numerical QC
    metrics are extracted
- name: DIA-NN
  role: Analyzes DIA files (v1.7.15) to derive peptide and protein quantification
    feeding the MS2-intensity-based DIA metrics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:fa040768dfa082941c1ee180f0de1ae79b474211d5b7dd051b95ebb21d7ef2fd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/guomics-lab/iDIA-QC
  tools:
  - DIA-NN
  - Orbitrap
  - iDIA-QC
  - msConvert
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# ratio-of-change-curve-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the ratio-of-change of LC-MS QC metrics (peptide number, protein number, MS1 area for DDA; MS2 intensity for DIA) between consecutive injections and aligns these values with maintenance-event timestamps to reveal whether DIA-based metrics detect system faults more sensitively than DDA-based metrics.

## When to use

Use when paired longitudinal DIA/DDA QC injections must be summarized as ratio-of-change curves around recorded maintenance-event timestamps to compare fault-detection sensitivity of DIA vs DDA metrics.

## When NOT to use

- Only a single file is available (no consecutive injections), so no ratio-of-change between injections can be computed
- Maintenance-event timestamps are unavailable or unrecorded for the instrument period, so curves cannot be aligned to fault events
- The goal is a qualified/unqualified AI classification of individual files rather than longitudinal sensitivity comparison of DIA vs DDA metrics

## Inputs

- Paired vendor raw MS files (.raw, .d, or .wiff) from consecutive DDA and DIA injections
- mzXML files converted from vendor raw files via msConvert
- DIA-NN (v1.7.15) peptide and protein quantification outputs (1% FDR at peptide precursor and protein levels)
- Instrument type and instrument ID assignments for each file
- Recorded maintenance-event timestamps for the monitored instrument (e.g., QE HF-X)

## Outputs

- Ratio-of-change values per QC metric (peptide number, protein number, MS1 area, MS2 intensity) between consecutive injections
- Metric matrices exported by iDIA-QC
- HTML report plotting ratio-of-change curves aligned with maintenance-event timestamps

## How to apply

Extract per-file numerical QC metrics — peptide number, protein number, MS1 area for DDA files, and MS2 intensity for DIA files — from mzXML conversions (via msConvert from .raw/.d/.wiff vendor files) and from DIA-NN (v1.7.15) peptide/protein quantification output filtered at 1% FDR at both peptide precursor and protein levels. For each metric, compute the ratio-of-change between consecutive injections to capture sudden shifts in LC-MS status. Align the ratio-of-change series with the recorded maintenance-event timestamps of the monitored instrument (e.g., the QE HF-X). Run iDIA-QC to export the metric matrices and generate the HTML report that plots the ratio-of-change curves around each maintenance event. The rationale is that a larger, sharper ratio-of-change excursion at a fault timestamp indicates higher detection sensitivity, so the reproduced curves should show DIA-based consensus QC metrics exceeding DDA-based metrics at the maintenance events.

## Related tools

- **iDIA-QC** (GUI that exports the metric matrices and generates the HTML report plotting ratio-of-change curves around maintenance events) — github.com/guomics-lab/iDIA-QC
- **msConvert** (Converts vendor raw files (.raw/.d/.wiff) to mzXML from which numerical QC metrics are extracted)
- **DIA-NN** (Analyzes DIA files (v1.7.15) to derive peptide and protein quantification feeding the MS2-intensity-based DIA metrics)

## Evaluation signals

- Ratio-of-change excursions coincide with the recorded maintenance-event timestamps when curves are aligned
- Reproduced curves show DIA-based consensus QC metrics (MS2 intensity, peptide/protein numbers) exhibiting higher sensitivity than DDA-based metrics (MS1 area) at the maintenance events, as reported in the paper
- DIA-NN identifications pass 1% FDR at both peptide precursor and protein levels before metric extraction
- iDIA-QC outputs include both the html report and the metric matrices, the latter usable without statistical software such as R/Python
- Per-file analysis completes in under 5 minutes as stated for iDIA-QC

## Limitations

- The study could not cover all LC and MS instruments, especially newly emerging ones
- Generalization of the workflow to other QC and experimental samples requires further investigation
- Sensitivity comparison depends on the accuracy of recorded maintenance-event timestamps; files must be assigned the correct instrument type and instrument ID

## Evidence
