---
name: retention-time-confirmation-against-standards
description: Use when assigning highest-confidence (MSI level 1) compound identities
  from LC-MS/MS peak lists by matching precursor mass, retention time, and MS/MS fragments
  against an in-house authentic-standard spectral library.
when_to_use_negative:
- The spectrum has no authentic standard measured under identical chromatographic
  conditions — retention-time confirmation is impossible and the compound should be
  annotated at MSI level 2/3 instead.
- You only need putative class-level annotation of the remaining spectra (MSI level
  3) — use CSI:FingerID/NIST hybrid search rather than the standard-based level 1
  workflow.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: HILIC-Q Exactive HF mass spectrometry
  role: Untargeted LC-MS/MS acquisition providing the urine peak lists whose spectra
    are matched against authentic standards
- name: CSH-Q Exactive HF mass spectrometry
  role: Second untargeted assay (lipid-oriented) whose spectra are pooled with HILIC
    data for standard matching
- name: CSI:FingerID
  role: In silico fragmentation tool applied only to compounds left after level 1/2
    standard-based annotation (MSI level 3)
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:8ce5d4c3c203b43e0c889718140975dc071785b587c34fdf1592a0b751fd7632
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSH-Q Exactive HF mass spectrometry
  - CSI:FingerID
  - CarniBlast
  - HILIC-Q Exactive HF mass spectrometry
  - NIST17
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Retention-time confirmation against standards

## Summary

Assigns MSI level 1 identifications by requiring an unknown's precursor accurate mass, retention time, and MS/MS fragment ions to all agree with a measured authentic-standard spectrum, as done in the urine HILIC/CSH-Q Exactive HF workflow (175 compounds at level 1).

## When to use

Use when assigning highest-confidence (MSI level 1) compound identities from LC-MS/MS peak lists by matching precursor mass, retention time, and MS/MS fragments against an in-house authentic-standard spectral library.

## When NOT to use

- The spectrum has no authentic standard measured under identical chromatographic conditions — retention-time confirmation is impossible and the compound should be annotated at MSI level 2/3 instead.
- You only need putative class-level annotation of the remaining spectra (MSI level 3) — use CSI:FingerID/NIST hybrid search rather than the standard-based level 1 workflow.

## Inputs

- Merged urine MS/MS peak lists from untargeted HILIC-Q Exactive HF and CSH-Q Exactive HF LC-MS/MS runs
- In-house authentic-standard spectral library entries with precursor mass, retention time, and MS/MS fragment ions

## Outputs

- Table of MSI level 1 compound identifications (target 175 compounds) supported by precursor mass, retention time, and MS/MS matching
- Set of unmatched spectra forwarded to MSI level 2 library search (MoNA, NIST17, CarniBlast)

## How to apply

Load the merged urine MS/MS peak lists from the two untargeted assays (HILIC-Q Exactive HF and CSH-Q Exactive HF) and search each spectrum against the in-house authentic-standard spectral library, requiring agreement in precursor accurate mass, retention time, and MS/MS fragment ions simultaneously. Only hits meeting all three criteria are promoted to MSI level 1 (the article identified 175 such compounds); spectra failing retention-time or fragment agreement fall through to MSI level 2 library search (MoNA/NIST17, CarniBlast). The rationale is that MSI level 1 demands measurement of a reference compound under identical conditions, so retention time must be part of the match and not just mass and spectral similarity. Combine the level 1 results with downstream level 2/3 annotations in a single table and tally counts per MSI confidence level. Validate by confirming the level 1 tally equals 175 and that every level 1 hit is supported by all three matching criteria.

## Related tools

- **HILIC-Q Exactive HF mass spectrometry** (Untargeted LC-MS/MS acquisition providing the urine peak lists whose spectra are matched against authentic standards)
- **CSH-Q Exactive HF mass spectrometry** (Second untargeted assay (lipid-oriented) whose spectra are pooled with HILIC data for standard matching)
- **CSI:FingerID** (In silico fragmentation tool applied only to compounds left after level 1/2 standard-based annotation (MSI level 3)) — https://github.com/sirius-ms/sirius

## Evaluation signals

- Level 1 tally equals 175 compounds, matching the article's reported count.
- Every level 1 hit is supported by agreement in precursor mass, retention time, AND MS/MS fragment ions — no hit rests on mass alone.
- The combined level 1 + level 2 tally equals 753 (175 + 578), with level 2 comprising 39 lipids, 85 acylcarnitines, and 454 hydrophilic compounds.
- Spectra not matching standards are correctly routed to MoNA/NIST17 (480 compounds) and CarniBlast (67 acylcarnitines) searches rather than being counted at level 1.

## Limitations

- Only compounds with available authentic standards can reach MSI level 1; the majority of acquired MS/MS spectra cannot be annotated with certainty to specific chemicals.
- Structure assignments from tools like Sirius/CSI:Finger ID and NIST17 still require validation with authentic chemicals.
- The field lacks generally accepted, validated automated calculations of identification confidence levels with false-discovery rate assessments in metabolomics MS/MS.

## Evidence

- [readme] SIRIUS handles the downstream in silico annotation after standard-based ID: [evidence span withheld — anchor status: external_doc]
