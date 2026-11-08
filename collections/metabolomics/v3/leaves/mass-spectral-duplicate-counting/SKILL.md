---
name: mass-spectral-duplicate-counting
description: 'Use when integrating multiple OMSLs (e.g., GNPS .MGF, MoNA .MSP, MassBank,
  MS-Dial-VS17) and you need to measure cross-library redundancy: the fraction of
  unique chemicals shared between two or more libraries and per-library duplicate
  spectra counts.'
when_to_use_negative:
- When the input is a single library with no cross-source integration question — overlap
  and redundancy statistics are only meaningful across multiple sources
- When chemical identifiers are not harmonized (raw SMILES/InChI as submitted by submitters);
  without RDKit-standardized InChIKeys, cross-library matching will be inflated by
  inconsistent identifier representations
- When the analysis goal is spectral similarity scoring or identification rather than
  library coverage/redundancy assessment
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FragHub
  role: Integrates and harmonizes the OMSLs into a unified format with standardized
    InChIKeys, producing the identifier-to-library mapping that the overlap analysis
    consumes
  repo: https://github.com/eMetaboHUB/FragHub
- name: Python
  role: Parsing library files, building InChIKey membership sets, and computing overlap/redundancy
    tabulations
- name: RDKit
  role: Standardizes chemical identifiers (SMILES, InChI, InChIKey) so cross-library
    identifier comparison is on harmonized keys
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:3461376ace55af618d8fc41ca932c8358a5780e605d0b9b84f524fe1b634cebd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  repo_url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mass-spectral-duplicate-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies chemical identifier overlap and spectral redundancy across Open Mass Spectral Libraries (OMSLs) by extracting harmonized InChIKeys from each source and tabulating how many of the integrated unique identifiers appear in one, two, three, or all four libraries.

## When to use

Use when integrating multiple OMSLs (e.g., GNPS .MGF, MoNA .MSP, MassBank, MS-Dial-VS17) and you need to measure cross-library redundancy: the fraction of unique chemicals shared between two or more libraries and per-library duplicate spectra counts.

## When NOT to use

- When the input is a single library with no cross-source integration question — overlap and redundancy statistics are only meaningful across multiple sources
- When chemical identifiers are not harmonized (raw SMILES/InChI as submitted by submitters); without RDKit-standardized InChIKeys, cross-library matching will be inflated by inconsistent identifier representations
- When the analysis goal is spectral similarity scoring or identification rather than library coverage/redundancy assessment

## Inputs

- Processed OMSL library files: GNPS .MGF, MoNA .MSP, MassBank records, MS-Dial-VS17 (or any FragHub-integrated output in .MSP/.MGF/.JSON/.CSV/.XML)
- Harmonized chemical identifiers (InChIKey) per spectrum, as standardized by FragHub via RDKit
- Per-library spectrum and compound counts (e.g., 63,935 GNPS spectra; 376,430 MS-Dial-VS17 spectra)

## Outputs

- Tabulation of unique InChIKeys by library membership count (1, 2, 3, or 4 libraries) out of 35,673 total unique identifiers
- Fraction of unique chemical identifiers shared between two or more OMSLs (~45%) and count exclusive to one library (19,419)
- Per-library spectral duplicates per InChIKey (spectra-to-compound redundancy ratios)

## How to apply

Parse each processed OMSL file with Python and extract the harmonized InChIKey of every compound per source (expected: 13,507 GNPS, 21,839 MoNA, 8,358 MassBank, 22,282 MS-Dial-VS17 compounds). Build an InChIKey-to-library membership mapping across the 35,673 unique chemical identifiers in the 794,985-spectra integrated set, then tabulate how many identifiers appear in exactly one, two, three, or four libraries. Separately count spectral duplicates per InChIKey within each library (e.g., MassBank's 164,261 spectra over 8,358 compounds; MoNA's 190,359 spectra over 21,839 compounds) to quantify within-library redundancy. Report the shared-identifier fraction alongside per-library compound and spectrum counts; in the FragHub study this yielded ~45% shared and 19,419 compounds exclusive to a single OMSL (Figure 3).

## Related tools

- **FragHub** (Integrates and harmonizes the OMSLs into a unified format with standardized InChIKeys, producing the identifier-to-library mapping that the overlap analysis consumes) — https://github.com/eMetaboHUB/FragHub
- **Python** (Parsing library files, building InChIKey membership sets, and computing overlap/redundancy tabulations)
- **RDKit** (Standardizes chemical identifiers (SMILES, InChI, InChIKey) so cross-library identifier comparison is on harmonized keys)

## Examples

```
# tabulate InChIKey membership across the four OMSLs
keys = [set(parse_inchikeys(f)) for f in ['gnps.mgf','mona.msp','massbank_records.xml','msdial_vs17.csv']]; from collections import Counter; bins = Counter(len({k for k in keys if k in union} for k in set().union(*keys)))
```

## Evaluation signals

- Shared-identifier fraction reproduces the paper's reported ~45% of the 35,673 unique chemicals
- Count of identifiers exclusive to a single OMSL equals 19,419 (per Figure 3)
- Per-library unique compound counts match Results values: 13,507 GNPS, 21,839 MoNA, 8,358 MassBank, 22,282 MS-Dial-VS17
- Sum of unique identifiers across membership bins equals 35,673, and total spectra across libraries equals 794,985
- Within-library redundancy ratios are consistent with reported values (e.g., MassBank: 164,261 spectra / 8,358 compounds; MoNA: 190,359 / 21,839)

## Limitations

- Identifier overlap depends on FragHub's deduplication and harmonization: the workflow refines the 794,985-spectrum / 35,673-identifier integrated set to 602,744 spectra for 32,193 unique chemicals, so overlap statistics differ before vs. after filtering
- Redundancy across OMSLs may lead to inconsistencies in downstream dot-product and reverse dot-product scoring, which are highly sensitive to fragment number and intensity
- Spectra lacking SMILES/InChI, precursor m/z, and adduct information are removed by FragHub and therefore absent from the integrated identifier set

## Evidence
