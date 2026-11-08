---
name: compound-redundancy-quantification
description: Use when multiple OMSL files (e.g., GNPS .MGF, MoNA .MSP, MassBank, MS-Dial-VS17)
  have been integrated or are being compared and you need the fraction of compounds
  covered by more than one library versus single-library exclusives.
when_to_use_negative:
- You need spectral-level deduplication rather than compound-level overlap — FragHub's
  SPLASH-key-based 'update.json' repository is the mechanism for keeping only new
  spectra, not this compound-membership analysis
- The libraries have not been harmonized to consistent chemical identifiers; raw SMILES/InChI
  fields across sources are not directly comparable without RDKit standardization
  as done in FragHub
- The goal is annotating unknown spectra rather than characterizing library overlap/redundancy
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: FragHub
  role: Integrates the four OMSLs into a unified format and harmonizes metadata (SMILES,
    InChI, InChIKey via RDKit), yielding the identifier set whose cross-library overlap
    is quantified
  repo: github.com/eMetaboHUB/FragHub
- name: Python
  role: Language used to parse the .MGF/.MSP library files, build the InChIKey membership
    mapping, and compute the shared-identifier fraction
- name: RDKit
  role: Standardizes chemical identifiers (SMILES, InChI, InChIKey) so cross-library
    compound identity comparisons are consistent
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:aaa0af51094a5884c2cfa54de555ed174ba2efcccb675c188219fbb614f04d8b
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# compound-redundancy-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how many unique chemical identifiers (harmonized InChIKeys) across integrated open mass spectral libraries (OMSLs) are shared between two or more libraries versus exclusive to one, exposing cross-library redundancy before or after FragHub integration.

## When to use

Use when multiple OMSL files (e.g., GNPS .MGF, MoNA .MSP, MassBank, MS-Dial-VS17) have been integrated or are being compared and you need the fraction of compounds covered by more than one library versus single-library exclusives.

## When NOT to use

- You need spectral-level deduplication rather than compound-level overlap — FragHub's SPLASH-key-based 'update.json' repository is the mechanism for keeping only new spectra, not this compound-membership analysis
- The libraries have not been harmonized to consistent chemical identifiers; raw SMILES/InChI fields across sources are not directly comparable without RDKit standardization as done in FragHub
- The goal is annotating unknown spectra rather than characterizing library overlap/redundancy

## Inputs

- GNPS-tagged library in .MGF format (13,507 compounds, 63,935 spectra)
- MoNA library in .MSP/.SDF/.JSON formats (21,839 compounds, 190,359 spectra)
- MassBank records (8,358 compounds, 164,261 spectra)
- MS-Dial-VS17 library, presplit into positive/negative ionization modes (22,282 compounds, 376,430 spectra)
- FragHub-processed library files with harmonized InChIKey metadata

## Outputs

- Per-library unique compound counts and spectrum counts
- InChIKey-to-library membership table (number of libraries each identifier appears in: 1, 2, 3, or 4)
- Fraction of unique identifiers shared between two or more OMSLs (~45% of 35,673)
- Count of single-library-exclusive compounds (19,419)

## How to apply

Parse each OMSL file (.MGF, .MSP, etc.) in Python and extract the harmonized InChIKey of every compound, counting unique identifiers per source (e.g., 13,507 GNPS, 21,839 MoNA, 8,358 MassBank, 22,282 MS-Dial-VS17). Build an InChIKey-to-library membership mapping over the full integrated identifier set (35,673 unique identifiers across 794,985 spectra in the FragHub study) and tabulate identifiers appearing in exactly one, two, three, or all four libraries. Optionally count spectral duplicates per InChIKey within each library (e.g., MassBank's 164,261 spectra for 8,358 compounds) to quantify within-library redundancy alongside cross-library sharing. Report the fraction of identifiers shared by two or more libraries; in the FragHub integration this was ~45% (with 19,419 single-library exclusives per Figure 3). Validate by checking the computed fraction reproduces the published value and per-library compound/spectrum counts match the Results section.

## Related tools

- **FragHub** (Integrates the four OMSLs into a unified format and harmonizes metadata (SMILES, InChI, InChIKey via RDKit), yielding the identifier set whose cross-library overlap is quantified) — github.com/eMetaboHUB/FragHub
- **Python** (Language used to parse the .MGF/.MSP library files, build the InChIKey membership mapping, and compute the shared-identifier fraction)
- **RDKit** (Standardizes chemical identifiers (SMILES, InChI, InChIKey) so cross-library compound identity comparisons are consistent)

## Examples

```
keys = {lib: set(s['inchikey'] for s in parse(lib)) for lib in libs}; shared = sum(1 for k in set().union(*keys.values()) if sum(k in s for s in keys.values()) >= 2); frac = shared / len(set().union(*keys.values()))
```

## Evaluation signals

- Per-library unique compound counts match published values: 13,507 (GNPS), 21,839 (MoNA), 8,358 (MassBank), 22,282 (MS-Dial-VS17)
- Total unique chemical identifiers across the integrated set equals 35,673 over 794,985 spectra
- Computed shared-identifier fraction reproduces the reported ~45% of 35,673 unique chemicals
- Single-library-exclusive count equals 19,419 compounds, consistent with Figure 3
- Within-library spectral redundancy checks reproduce reported ratios, e.g., MassBank 164,261 spectra / 8,358 compounds and MoNA 190,359 spectra / 21,839 compounds

## Limitations

- After FragHub filtering the identifier set shrinks to 602,744 spectra for 32,193 unique chemicals, so overlap statistics differ depending on whether pre- or post-filtering data are used
- Redundancy across OMSLs may lead to inconsistencies with dot product and reverse dot product scoring systems that are highly sensitive to fragment number and intensity
- FragHub removes spectra lacking SMILES and InChI, precursor m/z, and adduct information, which reduces the identifier pool before overlap can be computed

## Evidence

- [results] combined total of 794,985 spectra for 35,673 unique chemical identifiers. The FragHub data integration workflow refines this further to 602,744 spectra for 32,193 unique chemicals: "combined total of 794,985 spectra for 35,673 unique chemical identifiers. The FragHub data integration workflow refines this further to 602,744 spectra for 32,193 unique chemicals"
- [results] we specifically utilized GNPS-tagged databases in the .MGF format, comprising 13,507 compounds and 63,935 spectra: "we specifically utilized GNPS-tagged databases in the .MGF format, comprising 13,507 compounds and 63,935 spectra"
