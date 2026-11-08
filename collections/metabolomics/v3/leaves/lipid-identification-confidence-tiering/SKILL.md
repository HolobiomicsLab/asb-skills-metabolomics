---
name: lipid-identification-confidence-tiering
description: Use when a peak-picked, aligned HILIC-IM-MS feature table must be annotated
  with lipid identifications whose confidence is graded by how many dimensions (m/z,
  retention time, CCS) agree within defined tolerances.
when_to_use_negative:
- Data were acquired without ion mobility, so no CCS values exist for the RT-only
  or m/z-only lower tiers to outperform — matching cannot be stratified by CCS
- The goal is de novo lipid structure elucidation rather than database-level annotation
  of features at graded confidence levels
- Features are not peak-picked/aligned across samples (e.g. raw profile-mode mzML),
  since the loaders expect processed feature tables
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: LiPydomics
  role: Provides the Dataset loaders (data module), the predicted and experimental
    lipid databases, and the tiered-confidence identification functions applied to
    positive- and negative-ESI HILIC-IM-MS feature tables
  repo: https://github.com/dylanhross/lipydomics
- name: Python
  role: Language in which the lipydomics API is invoked to run the tiered identification
    and tabulate per-tier counts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:3c1bf4add9bdf261f6b9d8fc24d61b36d1759eb8fd343f7b028232b8e6eee97e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dylanhross/lipydomics
  repo_url: https://github.com/dylanhross/lipydomics
  tools:
  - LiPydomics
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# lipid-identification-confidence-tiering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns lipid annotations to HILIC-IM-MS features at multiple confidence levels by matching against LiPydomics' predicted (145,388-entry) and experimental m/z-RT-CCS databases, relaxing from m/z+RT+CCS agreement down to m/z-only. This reproduces the tiered-confidence identification scheme of LiPydomics, applied to MRSA positive- and negative-ESI datasets.

## When to use

Use when a peak-picked, aligned HILIC-IM-MS feature table must be annotated with lipid identifications whose confidence is graded by how many dimensions (m/z, retention time, CCS) agree within defined tolerances.

## When NOT to use

- Data were acquired without ion mobility, so no CCS values exist for the RT-only or m/z-only lower tiers to outperform — matching cannot be stratified by CCS
- The goal is de novo lipid structure elucidation rather than database-level annotation of features at graded confidence levels
- Features are not peak-picked/aligned across samples (e.g. raw profile-mode mzML), since the loaders expect processed feature tables

## Inputs

- Aligned, peak-picked feature table for the positive ESI HILIC-IM-MS dataset (CSV)
- Aligned, peak-picked feature table for the negative ESI HILIC-IM-MS dataset (CSV)
- LiPydomics predicted lipid database (145,388 entries)
- LiPydomics experimental m/z-RT-CCS lipid database (45 classes, 23 with HILIC RT)

## Outputs

- Per-feature lipid annotations with confidence-level tier assignments for each ESI mode
- Table of identification counts per confidence level per ESI mode (e.g. identifications.csv)
- Flags for mode-tier combinations that violate the expected monotonic decrease in identifications

## How to apply

Load the aligned, peak-picked positive- and negative-ESI feature data into lipydomics Dataset objects using the data module's CSV/pandas loaders. Instantiate the predicted lipid database (145,388 entries) alongside the experimental m/z-RT-CCS database (45 lipid classes, 23 with HILIC retention times) from the identification module. Run tiered identification per ESI mode, starting at the highest-confidence tier requiring agreement within 0.02 Da (m/z), 0.2 min (RT), and 3.0% (CCS), then relax through lower tiers (dropping RT and CCS matching) so each feature receives its best available annotation. Count annotated features per confidence level per mode and write the per-tier table (e.g. identifications.csv). Validate that counts decrease monotonically as matching stringency increases, mirroring the Figure 6J pattern; flag any mode-tier combination violating monotonic decrease for expert review.

## Related tools

- **LiPydomics** (Provides the Dataset loaders (data module), the predicted and experimental lipid databases, and the tiered-confidence identification functions applied to positive- and negative-ESI HILIC-IM-MS feature tables) — https://github.com/dylanhross/lipydomics
- **Python** (Language in which the lipydomics API is invoked to run the tiered identification and tabulate per-tier counts)

## Examples

```
from lipydomics.data import Dataset; from lipydomics import identification  # load ESI-mode CSVs, run tiered m/z+RT+CCS matching
```

## Evaluation signals

- The number of annotated features decreases steadily (monotonically) as matching stringency increases from m/z-only to m/z + RT + CCS in both positive and negative ESI modes
- Every feature in each dataset carries its best available confidence-level annotation after relaxing through all tiers
- Matching tolerances used are explicitly 0.02 Da (m/z), 0.2 min (RT), and 3.0% (CCS)
- The per-tier count table is written for both ESI modes and tabulates identifications per confidence level
- Identifications resolve to Level 3 annotations (lipid class/subclass and fatty acid sum composition) where the HILIC-IM-MS dimensions agree

## Limitations

- Predicted CCS/RT models cover only 22 and 23 lipid classes respectively, so features outside those classes fall back to lower-confidence tiers
- Experimental database coverage is uneven: 45 lipid classes have m/z and CCS but only 23 include HILIC retention times, limiting full three-dimensional matching
- Predicted databases mostly cover mammalian lipids with limited coverage of bacterial lipids, relevant for MRSA applications

## Evidence

- [abstract] identifying lipid species at different confidence levels: [evidence span withheld — anchor status: not_found]
