---
name: predicted-lipid-database-matching
description: Use when aligned, peak-picked HILIC-IM-MS feature tables (positive and/or
  negative ESI) must be annotated with lipid identities at graded confidence levels
  using m/z, retention time, and CCS matching.
when_to_use_negative:
- Data lacks ion-mobility CCS values or HILIC retention times, so the higher-confidence
  tiers (m/z + RT + CCS) cannot be evaluated and only m/z-only matching is possible
- Input is raw vendor spectra rather than an aligned, peak-picked feature table suitable
  for loading as a lipydomics Dataset
- Analytes are non-lipid metabolites outside the 45 lipid classes covered by LiPydomics'
  experimental and predicted databases
tools:
- name: LiPydomics
  role: Provides the identification module, predicted lipid database, experimental
    m/z-RT-CCS database, and Dataset loaders used for tiered lipid matching
  repo: https://github.com/dylanhross/lipydomics
- name: Python
  role: Implementation language in which the lipydomics identification workflow is
    scripted (e.g. via example_standard_analysis.py)
  repo: https://github.com/dylanhross/lipydomics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:dd323b99744ae9cd9c6e122ce3eb7eaad0631bad2a3a368ec2fb2c4666dad35c
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# predicted-lipid-database-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tiered lipid identification by matching HILIC-IM-MS features against LiPydomics' predicted lipid database (145,388 entries) and its experimental m/z-RT-CCS database, annotating each feature at the highest confidence level (m/z, RT, CCS) supported by the data.

## When to use

Use when aligned, peak-picked HILIC-IM-MS feature tables (positive and/or negative ESI) must be annotated with lipid identities at graded confidence levels using m/z, retention time, and CCS matching.

## When NOT to use

- Data lacks ion-mobility CCS values or HILIC retention times, so the higher-confidence tiers (m/z + RT + CCS) cannot be evaluated and only m/z-only matching is possible
- Input is raw vendor spectra rather than an aligned, peak-picked feature table suitable for loading as a lipydomics Dataset
- Analytes are non-lipid metabolites outside the 45 lipid classes covered by LiPydomics' experimental and predicted databases

## Inputs

- Aligned, peak-picked MRSA HILIC-IM-MS feature table for positive ESI mode (CSV/pandas-loadable)
- Aligned, peak-picked MRSA HILIC-IM-MS feature table for negative ESI mode (CSV/pandas-loadable)
- LiPydomics predicted lipid database (145,388 entries)
- Experimental m/z-RT-CCS lipid database (45 lipid classes; 23 with HILIC retention times)

## Outputs

- Per-feature lipid annotations with confidence level (m/z-only up to m/z + RT + CCS) for each ESI mode
- Per-tier identification count table written to identifications.csv, tabulated per positive and negative mode

## How to apply

Load the aligned, peak-picked feature data for each ESI mode into lipydomics Dataset objects using the data module's CSV/pandas loaders. Instantiate the predicted lipid database (145,388 entries) and the experimental database (m/z and CCS for 45 lipid classes, 23 with HILIC retention times) from the identification module. Run tiered identification starting at the highest-confidence tier requiring agreement on predicted m/z + HILIC retention time + predicted CCS within tolerances of 0.02 Da (m/z), 0.2 min (RT), and 3.0% (CCS), then relax through successively lower tiers (fewer matching dimensions, e.g. m/z only) so every feature receives its best available confidence-level annotation. Repeat separately for positive- and negative-mode datasets, count annotated features per confidence level per mode, and write the per-tier table to identifications.csv. The rationale is that predicted CCS and RT values fill gaps in experimental CCS collections, expanding lipid class coverage beyond curated experimental entries while keeping the strictest tier anchored to multi-dimensional agreement.

## Related tools

- **LiPydomics** (Provides the identification module, predicted lipid database, experimental m/z-RT-CCS database, and Dataset loaders used for tiered lipid matching) — https://github.com/dylanhross/lipydomics
- **Python** (Implementation language in which the lipydomics identification workflow is scripted (e.g. via example_standard_analysis.py)) — https://github.com/dylanhross/lipydomics

## Examples

```
python example_standard_analysis.py  # scripted lipydomics analysis incl. the identification module
```

## Evaluation signals

- Annotated feature counts decrease steadily (monotonically) as matching stringency increases from m/z-only to m/z + RT + CCS, reproducing the Figure 6J pattern; flag any mode-tier combination that violates monotonic decrease
- Match tolerances respect the specified thresholds: 0.02 Da (m/z), 0.2 min (RT), 3.0% (CCS)
- identifications.csv contains counts for every confidence level for both positive and negative ESI modes
- Every feature in each dataset receives an annotation at its best available confidence level after relaxing through all tiers
- Predicted database instantiated with the expected 145,388 entries before matching

## Limitations

- HILIC-IM-MS matching generally supports Level 3 lipid identifications (lipid class/subclass and fatty acid sum composition), not fully resolved species-level IDs
- Experimental CCS/RT coverage is incomplete: only 23 of 45 lipid classes have HILIC retention times, so predicted values must fill the gaps
- Any mode-tier combination violating the monotonic decrease in identifications should be flagged for expert review

## Evidence
