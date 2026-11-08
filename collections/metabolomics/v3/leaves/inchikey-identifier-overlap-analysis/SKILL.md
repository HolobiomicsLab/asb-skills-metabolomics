---
name: inchikey-identifier-overlap-analysis
description: Use when integrating two or more mass spectral libraries (e.g., .MGF,
  .MSP, .JSON exports) and you need the fraction of compounds present in multiple
  sources, per-library compound/spectrum counts, and single-library exclusivity.
when_to_use_negative:
- When deduplicating at the spectral level rather than the compound level — use SPLASH-key-based
  deduplication (update.json) instead of InChIKey membership
- When the input is a single already-merged library with no cross-library membership
  labels, since no overlap can be computed
- When the goal is spectral matching quality (dot product / reverse dot product scoring)
  rather than compound-level coverage overlap
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: FragHub
  role: Integrates and harmonizes the four OMSLs into a unified format and standardizes
    the chemical identifiers (InChIKey) used for the overlap computation
  repo: github.com/eMetaboHUB/FragHub
- name: Python
  role: Parses .MGF/.MSP library files and computes the InChIKey membership counts
    and shared-identifier fraction
- name: RDKit
  role: Standardizes SMILES, InChI, and InChIKey identifiers so cross-library compound
    identity comparison is consistent
- name: spectra-hash (SPLASH)
  role: Provides the SPLASH spectral identifier used by FragHub for spectral-level
    (not compound-level) deduplication
  repo: github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:f24a026595cc4f9cd597b572c91301e27f349f890a6382c63ffdeb052adf3cd5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - Python
  - RDKit
  - spectra-hash (SPLASH)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# inchikey-identifier-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how many unique chemical identifiers (harmonized InChIKeys) are shared between two or more open mass spectral libraries (OMSLs) versus exclusive to one, after FragHub integration of GNPS, MoNA, MassBank, and MS-Dial-VS17. It exposes cross-library redundancy that affects dereplication and spectral scoring.

## When to use

Use when integrating two or more mass spectral libraries (e.g., .MGF, .MSP, .JSON exports) and you need the fraction of compounds present in multiple sources, per-library compound/spectrum counts, and single-library exclusivity.

## When NOT to use

- When deduplicating at the spectral level rather than the compound level — use SPLASH-key-based deduplication (update.json) instead of InChIKey membership
- When the input is a single already-merged library with no cross-library membership labels, since no overlap can be computed
- When the goal is spectral matching quality (dot product / reverse dot product scoring) rather than compound-level coverage overlap

## Inputs

- GNPS-tagged library in .MGF format (13,507 compounds / 63,935 spectra)
- MoNA library in .MSP (with .SDF, .JSON metadata) (21,839 compounds / 190,359 spectra)
- MassBank records (8,358 compounds / 164,261 spectra)
- MS-Dial-VS17 library, pre-split into positive/negative ionization modes (22,282 compounds / 376,430 spectra)
- InChIKey-to-library membership table from the integrated spectra

## Outputs

- Membership counts of identifiers in exactly 1, 2, 3, or 4 libraries
- Fraction of unique identifiers shared between two or more OMSLs (~45% of 35,673)
- Count of single-library-exclusive compounds (19,419)
- Per-library compound and spectrum counts table
- Spectral duplicates per InChIKey within each library (redundancy metric)

## How to apply

First parse each library (GNPS .MGF, MoNA .MSP, MassBank records, MS-Dial-VS17) and extract the FragHub/RDKit-harmonized InChIKey for every compound, building an InChIKey-to-library membership map (expected per-library uniques: GNPS 13,507, MoNA 21,839, MassBank 8,358, MS-Dial-VS17 22,282 over a 794,985-spectra integrated set with 35,673 unique identifiers). Tabulate how many identifiers appear in exactly one, two, three, or four libraries. Additionally count spectral duplicates per InChIKey within each library (e.g., MassBank 164,261 spectra / 8,358 compounds) to gauge within-library redundancy. Report the shared fraction: about 45% of the 35,673 unique chemicals appear in two or more OMSLs, leaving 19,419 compounds exclusive to a single library. Rationale: harmonization via RDKit is essential because native SMILES/InChI/InChIKey fields differ across OMSL formats; without normalized identifiers the overlap is undercounted.

## Related tools

- **FragHub** (Integrates and harmonizes the four OMSLs into a unified format and standardizes the chemical identifiers (InChIKey) used for the overlap computation) — github.com/eMetaboHUB/FragHub
- **Python** (Parses .MGF/.MSP library files and computes the InChIKey membership counts and shared-identifier fraction)
- **RDKit** (Standardizes SMILES, InChI, and InChIKey identifiers so cross-library compound identity comparison is consistent)
- **spectra-hash (SPLASH)** (Provides the SPLASH spectral identifier used by FragHub for spectral-level (not compound-level) deduplication) — github.com/berlinguyinca/spectra-hash

## Examples

```
memb = df.groupby('InChIKey')['library'].nunique(); print((memb >= 2).sum() / len(memb))  # expect ~0.45 for 35,673 identifiers
```

## Evaluation signals

- Per-library unique InChIKey counts must equal 13,507 (GNPS), 21,839 (MoNA), 8,358 (MassBank), 22,282 (MS-Dial-VS17)
- Total unique chemical identifiers across the integrated 794,985-spectra set must equal 35,673
- Computed shared-identifier fraction should reproduce the reported ~45% shared between two or more OMSLs
- Single-library-exclusive compound count should equal 19,419 (Figure 3)
- Every identifier's membership count must fall between 1 and 4 (number of integrated libraries)

## Limitations

- Identifier harmonization depends on FragHub's cleaning step: spectra lacking SMILES/InChI, precursor m/z, or adduct information are removed, so overlap is computed only over the harmonized subset (602,744 spectra / 32,193 chemicals after refinement)
- Cross-OMSL redundancy can lead to inconsistencies with dot product and reverse dot product scoring, which are highly sensitive to fragment number and intensity
- Collision energy is not harmonized across libraries (~70 different formats), so shared InChIKeys may correspond to incomparable spectra

## Evidence
