---
name: mz-rt-ccs-tolerance-filtering
description: Use when analyzing aligned, peak-picked HILIC-IM-MS feature tables (positive/negative
  ESI) and every feature must receive a lipid annotation at the highest confidence
  level supported by m/z, retention time, and CCS agreement.
when_to_use_negative:
- The input data lacks an ion-mobility CCS dimension (e.g. conventional LC-MS only)
  — the highest-confidence RT+CCS tiers cannot be evaluated as designed
- The data is raw spectra rather than an aligned, peak-picked feature table — this
  skill starts from features already picked and aligned per ESI mode
- The goal is statistical/multivariate analysis or plotting of an already-identified
  lipid table rather than assigning confidence-level identifications
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LiPydomics
  role: Provides the identification module for tiered lipid species matching at different
    confidence levels and the data module for loading feature datasets
  repo: https://github.com/dylanhross/lipydomics
- name: Python
  role: Runtime environment in which the LiPydomics API is executed for the tiered
    identification workflow
  repo: https://github.com/dylanhross/lipydomics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:16325a256012c9bc1f543496f4209f1325f0d162c9862f90ebf9c8799d4e7c26
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# mz-rt-ccs-tolerance-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tiered lipid identification on HILIC-IM-MS lipidomics features using LiPydomics, matching each feature against a predicted lipid database (145,388 entries) and an experimental m/z-RT-CCS database under multi-dimensional tolerances (0.02 Da m/z, 0.2 min RT, 3.0% CCS), and assigning confidence levels based on how many dimensions agree.

## When to use

Use when analyzing aligned, peak-picked HILIC-IM-MS feature tables (positive/negative ESI) and every feature must receive a lipid annotation at the highest confidence level supported by m/z, retention time, and CCS agreement.

## When NOT to use

- The input data lacks an ion-mobility CCS dimension (e.g. conventional LC-MS only) — the highest-confidence RT+CCS tiers cannot be evaluated as designed
- The data is raw spectra rather than an aligned, peak-picked feature table — this skill starts from features already picked and aligned per ESI mode
- The goal is statistical/multivariate analysis or plotting of an already-identified lipid table rather than assigning confidence-level identifications

## Inputs

- Aligned, peak-picked HILIC-IM-MS feature data (CSV/pandas) for positive and negative ESI modes, loadable as lipydomics Dataset objects
- LiPydomics predicted lipid database (145,388 entries) with predicted CCS and HILIC retention time models
- Experimental m/z-RT-CCS database (45 lipid classes, 23 classes with HILIC retention times) from the identification module

## Outputs

- Per-feature confidence-level lipid annotations for each ESI mode (tiered from m/z-only to m/z + RT + CCS)
- Per-tier identification count table written to identifications.csv

## How to apply

Load the aligned, peak-picked MRSA feature data for each ESI mode into lipydomics Dataset objects via the data module's CSV/pandas loaders, then instantiate the 145,388-entry predicted lipid database alongside the experimental m/z-RT-CCS database from the identification module. Run tiered identification starting at the highest-confidence tier requiring predicted m/z + HILIC retention time + predicted CCS agreement within tolerances of 0.02 Da (m/z), 0.2 min (RT), and 3.0% (CCS); repeat for the negative-mode dataset with the same settings. Relax successively through lower-confidence tiers (fewer matching dimensions, e.g. m/z only) so every feature receives its best available confidence-level annotation. Count annotated features per confidence level per ESI mode and write the per-tier table to identifications.csv.

## Related tools

- **LiPydomics** (Provides the identification module for tiered lipid species matching at different confidence levels and the data module for loading feature datasets) — https://github.com/dylanhross/lipydomics
- **Python** (Runtime environment in which the LiPydomics API is executed for the tiered identification workflow) — https://github.com/dylanhross/lipydomics

## Examples

```
from lipydomics.data import Dataset; from lipydomics import identification  # tiered matching at 0.02 Da m/z, 0.2 min RT, 3.0% CCS
```

## Evaluation signals

- Tabulated identification counts per confidence level decrease steadily (monotonically) as matching stringency increases from m/z-only to m/z + RT + CCS, reproducing the Figure 6J pattern
- Any mode-tier combination violating the monotonic decrease is flagged for expert review
- Both positive and negative ESI mode datasets are processed with identical tolerance settings (0.02 Da m/z, 0.2 min RT, 3.0% CCS)
- Every feature in each Dataset receives a best-available confidence-level annotation (no feature left unannotated after the lowest tier)

## Limitations

- Predicted CCS databases mostly cover mammalian lipid species with limited coverage of bacterial lipids, so identifications may be sparse for bacterial features
- HILIC-IM-MS identification generally reaches only Level 3 confidence (lipid class/subclass and fatty acid sum composition), not molecular-species-level IDs
- The database and prediction models cover 22 and 23 lipid classes respectively for CCS/RT prediction; lipids outside trained classes fall back to lower-confidence tiers

## Evidence

- [abstract] tiered confidence-level identification module: [evidence span withheld — anchor status: not_found]
