---
name: esi-mode-feature-annotation
description: Use when peak-picked, aligned HILIC-IM-MS feature tables from positive
  and negative ESI modes need lipid species annotation at graded confidence levels
  (m/z-only up to m/z + RT + CCS).
when_to_use_negative:
- Input is raw IM-MS spectra rather than an aligned, peak-picked feature table — the
  skill operates on already peak-picked Dataset objects
- The analysis requires statistical or multivariate testing (e.g. PCA, t-tests) rather
  than lipid identification — the `stats` module, not this identification workflow,
  is the right fit
- Data were acquired with a separation or acquisition mode other than HILIC-IM-MS,
  since the CCS and retention time prediction models and databases were trained for
  HILIC-IM-MS lipidomics
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: LiPydomics
  role: Provides the `data` module (Dataset CSV/pandas loaders), the `identification`
    module (experimental and predicted lipid databases, tiered matching), and the
    scriptable API used to count and export per-tier identifications
  repo: https://github.com/dylanhross/lipydomics
- name: Python
  role: Execution environment for the LiPydomics API and the tiered identification
    script
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:078ddfe513f7baa98968b4490bf261aaab052534e226b66275f43c128b8b5ae4
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
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# esi-mode-feature-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates peak-picked HILIC-IM-MS features with lipid identifications at tiered confidence levels using LiPydomics, matching each feature's m/z, HILIC retention time, and predicted CCS against the experimental and 145,388-entry predicted lipid databases separately per ESI mode.

## When to use

Use when peak-picked, aligned HILIC-IM-MS feature tables from positive and negative ESI modes need lipid species annotation at graded confidence levels (m/z-only up to m/z + RT + CCS).

## When NOT to use

- Input is raw IM-MS spectra rather than an aligned, peak-picked feature table — the skill operates on already peak-picked Dataset objects
- The analysis requires statistical or multivariate testing (e.g. PCA, t-tests) rather than lipid identification — the `stats` module, not this identification workflow, is the right fit
- Data were acquired with a separation or acquisition mode other than HILIC-IM-MS, since the CCS and retention time prediction models and databases were trained for HILIC-IM-MS lipidomics

## Inputs

- Aligned, peak-picked HILIC-IM-MS feature table (CSV) for the positive ESI mode dataset
- Aligned, peak-picked HILIC-IM-MS feature table (CSV) for the negative ESI mode dataset
- Experimental m/z-RT-CCS database covering 45 lipid classes (23 with HILIC retention times)
- Predicted lipid database with 145,388 entries

## Outputs

- Per-tier lipid identification counts for positive and negative ESI modes
- identifications.csv containing the tabulated confidence-level annotation results

## How to apply

Load the aligned, peak-picked feature data for the positive and negative ESI mode datasets into LiPydomics Dataset objects via the `data` module's CSV/pandas loaders. Instantiate the experimental database of m/z and CCS for 45 lipid classes (23 with HILIC retention times) alongside the predicted lipid database of 145,388 entries from the `identification` module. Run tiered identification on each ESI mode dataset, starting at the highest-confidence tier requiring agreement within tolerances of 0.02 Da (m/z), 0.2 min (RT), and 3.0% (CCS), then relax through successively lower tiers (fewer matching dimensions) so every feature receives its best available confidence-level annotation. The rationale is that requiring more matching dimensions (m/z + RT + CCS) yields fewer but higher-confidence identifications, reproducing a monotonic decrease in identifications as stringency increases. Count annotated features per confidence level for each ESI mode and write the per-tier identification table to identifications.csv.

## Related tools

- **LiPydomics** (Provides the `data` module (Dataset CSV/pandas loaders), the `identification` module (experimental and predicted lipid databases, tiered matching), and the scriptable API used to count and export per-tier identifications) — https://github.com/dylanhross/lipydomics
- **Python** (Execution environment for the LiPydomics API and the tiered identification script)

## Examples

```
from lipydomics.data import Dataset; from lipydomics.identification import identify_features; ds = Dataset.load_csv('positive_mode.csv'); identify_features(ds)
```

## Evaluation signals

- The number of identified features must decrease monotonically as the required confidence level increases from m/z-only to m/z + RT + CCS, reproducing the Figure 6J pattern
- Annotated counts must be tabulated separately for positive and negative ESI modes
- Every feature in the input Dataset should carry an annotation at its best available confidence level after relaxing through lower tiers
- The output identifications.csv must contain the per-tier identification count table
- Matching must respect the article's tolerances: 0.02 Da for m/z, 0.2 min for retention time, and 3.0% for CCS

## Limitations

- The predicted CCS database and RT models mostly cover mammalian lipid species and have limited coverage of bacterial lipids, which can reduce annotation coverage for bacterial lipidomics data
- Identification is limited to Level 3 (lipid class/subclass and fatty acid sum composition) unless additional matching dimensions are available
- The package (including the lipid database and prediction models) is extensible, so newly acquired experimental CCS/RT data must be added manually; any mode-tier combination that violates the monotonic decrease in identifications should be flagged for expert review

## Evidence

- [methods] Tiered identification uses tolerances 0.02 Da (m/z), 0.2 min (RT), 3.0% (CCS): [evidence span withheld — anchor status: not_found]
- [results] Monotonic decrease of identifications with stringency: [evidence span withheld — anchor status: not_found]
