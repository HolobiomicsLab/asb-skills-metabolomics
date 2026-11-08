---
name: lipidomics-identification-count-tabulation
description: Use when you have aligned, peak-picked IM-MS lipidomics feature tables
  (one per ESI mode) and need lipid annotations ranked by confidence tier, with per-tier
  identification counts comparable to the paper's Figure 6J pattern.
when_to_use_negative:
- Input is raw vendor or profile-mode MS data rather than an aligned, peak-picked
  feature table — LiPydomics identification operates on Dataset objects built from
  processed features
- Data were not acquired on a HILIC-IM-MS platform (e.g., no CCS dimension or non-HILIC
  retention times), since the RT and CCS matching tiers and prediction models are
  class-specific to HILIC separation
- The goal is putative assignment of novel lipids absent from the 45-class experimental
  database and 145,388-entry predicted database
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: LiPydomics
  role: Python package providing the data loaders, identification module (tiered matching
    against predicted and experimental lipid databases), and stats/plotting used to
    annotate features and tabulate per-tier identification counts
  repo: https://github.com/dylanhross/lipydomics
- name: Python
  role: Runtime environment for scripting the tiered identification workflow and writing
    identifications.csv
- name: LipidCCS
  role: Prior CCS prediction work underpinning the machine-learning CCS prediction
    models used in the predicted lipid database
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:ade21cb4fe41f1f321d60bcc4d0578b6faf3ec5b2d374203a0fd712a16183fb3
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
  - LipidCCS
  - Python
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# lipidomics-identification-count-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies LiPydomics' tiered lipid identification to HILIC-IM-MS feature tables (e.g., MRSA datasets in positive and negative ESI) and tabulates the number of features annotated at each confidence level, from m/z-only matching up to m/z + retention time + CCS agreement.

## When to use

Use when you have aligned, peak-picked IM-MS lipidomics feature tables (one per ESI mode) and need lipid annotations ranked by confidence tier, with per-tier identification counts comparable to the paper's Figure 6J pattern.

## When NOT to use

- Input is raw vendor or profile-mode MS data rather than an aligned, peak-picked feature table — LiPydomics identification operates on Dataset objects built from processed features
- Data were not acquired on a HILIC-IM-MS platform (e.g., no CCS dimension or non-HILIC retention times), since the RT and CCS matching tiers and prediction models are class-specific to HILIC separation
- The goal is putative assignment of novel lipids absent from the 45-class experimental database and 145,388-entry predicted database

## Inputs

- Aligned, peak-picked feature table CSV for positive ESI mode (m/z, RT, CCS per feature)
- Aligned, peak-picked feature table CSV for negative ESI mode (m/z, RT, CCS per feature)
- LiPydomics predicted lipid database (145,388 entries)
- Experimental m/z-RT-CCS lipid database (45 lipid classes, 23 classes with HILIC retention times)

## Outputs

- identifications.csv: per-tier lipid identification counts for each ESI mode and confidence level
- Feature-level confidence annotations mapping each feature to its best available lipid identity tier

## How to apply

Load each aligned, peak-picked feature table (positive and negative ESI mode) into a lipydomics Dataset via the data module's CSV/pandas loaders. Instantiate the predicted lipid database (145,388 entries) alongside the experimental m/z-RT-CCS database from the identification module. Run tiered identification starting at the highest-confidence tier requiring agreement of predicted m/z + HILIC retention time + predicted CCS within tolerances of 0.02 Da (m/z), 0.2 min (RT), and 3.0% (CCS), then relax through successively lower tiers (dropping matching dimensions, e.g. m/z-only) so every feature receives its best available confidence-level annotation. Count annotated features per confidence level per ESI mode and write the per-tier table to identifications.csv. Rationale: matching on more orthogonal dimensions (m/z, RT, CCS) raises identification confidence, so counts must decrease monotonically as stringency increases.

## Related tools

- **LiPydomics** (Python package providing the data loaders, identification module (tiered matching against predicted and experimental lipid databases), and stats/plotting used to annotate features and tabulate per-tier identification counts) — https://github.com/dylanhross/lipydomics
- **Python** (Runtime environment for scripting the tiered identification workflow and writing identifications.csv)
- **LipidCCS** (Prior CCS prediction work underpinning the machine-learning CCS prediction models used in the predicted lipid database)

## Examples

```
python example_standard_analysis.py
```

## Evaluation signals

- Per-tier identification counts decrease monotonically as matching stringency increases from m/z-only to m/z + RT + CCS (reproduces the Figure 6J pattern); flag any mode-tier combination violating the monotonic decrease
- Counts are reported separately for positive and negative ESI modes with no mode-tier cell left empty
- Tolerances used are 0.02 Da (m/z), 0.2 min (RT), and 3.0% (CCS); verify these were applied at the highest-confidence tier
- Every feature in each input Dataset receives a confidence-level annotation after relaxing through all lower tiers
- Output file identifications.csv contains one row per confidence level per ESI mode

## Limitations

- Prediction models cover only 22 lipid classes for CCS and 23 for HILIC retention time, so features outside those classes cannot receive the highest-confidence tier
- The experimental database covers 45 lipid classes with only 23 classes containing HILIC retention times, limiting RT-based matching
- Previously available databases mostly cover mammalian lipid species with limited coverage of bacterial lipids; identifications depend on the coverage of the built-in databases
- The package (including lipid database and prediction models) is extensible, but coverage gaps persist as more data becomes available

## Evidence

- [abstract] identifying lipid species at different confidence levels: [evidence span withheld — anchor status: not_found]
