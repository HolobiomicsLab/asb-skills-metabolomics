---
name: isotopic-artifact-aware-tolerance-handling
description: Use when a GC/EI-MS analyte spectrum matches no reference library and
  a large enumerated derivative database (e.g., ~422M cathinone-like compounds with
  precomputed exact masses) must be narrowed to candidates near the analyte's molecular
  ion M+ mass.
when_to_use_negative:
- The analyte spectrum already matches an established reference library (NIST, Wiley,
  SWGDRUG) — PS2MS is only invoked when no reference match is found in the conventional
  screening procedure
- The analyte's molecular ion (M+ peak) cannot be reliably assigned, since it is the
  sole reference mass for the tolerance window
- Candidates of interest differ from the analyte by more than the ±5 Da window (e.g.,
  large multi-substituent derivatives), as they would be filtered out before spectrum/fingerprint
  ranking
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: PS2MS
  role: Implements the rule-based enumeration and M+-referenced mass-weight filtering
    of enumerated compounds within a tolerance range
  repo: https://github.com/jhhung/PS2MS
- name: rdkit
  role: Provides exact-mass computation for enumerated derivative structures (SMILES)
    prior to tolerance filtering
  repo: https://www.rdkit.org/docs/Install.html
- name: python 3.6.9
  role: Runtime environment for running the deep learning tools and processing the
    filtered candidate CSVs
  repo: https://www.python.org/downloads/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:ff52d73b9de423300ee52966e0e48d6812be4a9efe2e993cadbb7b4a286b97f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - PS2MS
  - python
  - python 3.6.9
  - rdkit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# isotopic-artifact-aware-tolerance-handling

## Summary

A rule-based mass-weight filtering step from PS2MS that uses the analyte's M+ peak (derived from its SWGDRUG reference spectrum) as a reference mass and retains enumerated synthetic NPS derivatives only when their exact masses fall within tolerance windows (mwf ±1 or ±5 Da) of that reference.

## When to use

Use when a GC/EI-MS analyte spectrum matches no reference library and a large enumerated derivative database (e.g., ~422M cathinone-like compounds with precomputed exact masses) must be narrowed to candidates near the analyte's molecular ion M+ mass.

## When NOT to use

- The analyte spectrum already matches an established reference library (NIST, Wiley, SWGDRUG) — PS2MS is only invoked when no reference match is found in the conventional screening procedure
- The analyte's molecular ion (M+ peak) cannot be reliably assigned, since it is the sole reference mass for the tolerance window
- Candidates of interest differ from the analyte by more than the ±5 Da window (e.g., large multi-substituent derivatives), as they would be filtered out before spectrum/fingerprint ranking

## Inputs

- Analyte GC/EI-MS spectrum (70 eV) with an assigned M+ peak, cross-referenced against the SWGDRUG MS library
- Enumerated synthetic NPS derivative database with precomputed exact masses (from task_001, generated from a core drug structure such as cathinone)
- Tolerance settings: mwf ±1 Da and mwf ±5 Da

## Outputs

- CSV candidate list for the mwf ±1 Da subset
- CSV candidate list for the mwf ±5 Da subset
- Summary recording candidate counts per tolerance setting (row counts must match the CSVs)

## How to apply

First extract the molecular-weight reference from the M+ peak of the analyte's SWGDRUG reference spectrum. Load the enumerated derivative database (built by substituting hydrogen atoms in the core drug structure with functional groups) whose records carry precomputed exact masses. Scan the database and retain candidates whose exact mass falls within ±1 Da of the reference M+ mass (the mwf ±1 setting), then repeat with the ±5 Da window (mwf ±5) to obtain a broader subset. Write each subset to a CSV candidate list and record per-tolerance candidate counts in a summary. This M+-anchored tolerance filtering is one of PS2MS's rule-based filters alongside Messy rings (>7 smallest sets of smallest rings), ΔSCScore thresholds, and 57 drug-like criteria from Virshup et al., and it prevents spending downstream NEIMS/DeepEI similarity computation on derivatives with implausible molecular weights.

## Related tools

- **PS2MS** (Implements the rule-based enumeration and M+-referenced mass-weight filtering of enumerated compounds within a tolerance range) — https://github.com/jhhung/PS2MS
- **rdkit** (Provides exact-mass computation for enumerated derivative structures (SMILES) prior to tolerance filtering) — https://www.rdkit.org/docs/Install.html
- **python 3.6.9** (Runtime environment for running the deep learning tools and processing the filtered candidate CSVs) — https://www.python.org/downloads/

## Evaluation signals

- Every row in the mwf ±1 output CSV has an exact mass within ±1 Da of the reference M+ mass extracted from the analyte's SWGDRUG reference spectrum
- Every row in the mwf ±5 output CSV has an exact mass within ±5 Da of the same reference mass
- Reported candidate counts per tolerance setting exactly equal the row counts of the corresponding CSV files
- The mwf ±5 subset fully contains the mwf ±1 subset (monotone nested windows)
- The reference mass used equals the M+ peak mass of the analyte's SWGDRUG reference spectrum, not a rounded nominal mass

## Limitations

- The tolerance windows (±1 and ±5 Da) are fixed settings; derivatives outside these windows are excluded regardless of spectral similarity
- The number of enumeration levels affects the filtered database size, and no exact number of enumeration levels is specified in the discussion
- Mass filtering alone provides no identity information — candidates must subsequently pass NEIMS-predicted spectrum and DeepEI fingerprint similarity ranking to yield a top-100 ranked list

## Evidence

- [discussion] M+ peak reference for mass-weight filtering: "utilizes the M+ peak as a reference for the mass weight filtering of enumerated compounds within a certain tolerance range"
- [readme] Database construction by functional-group substitution: "PS<sup>2</sup>MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
- [methods] Scale of enumeration and filtering: "the process of enumeration and filtering resulted in a total of approximately 422 million compounds."
- [methods] Other rule-based filters applied alongside mass filtering: "The most effective rule was Messy rings (more than 7 smallest sets of smallest rings)"
- [intro] PS2MS invoked only when no library match exists: "When the mass spectrum of an analyte, retrieved via GC/EI-MS, does not match any established reference spectra during the conventional screening procedure, PS2MS builds a supplementary synthetic NPS"
