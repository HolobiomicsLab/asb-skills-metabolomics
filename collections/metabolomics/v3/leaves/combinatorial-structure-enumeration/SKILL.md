---
name: combinatorial-structure-enumeration
description: Use when building an in silico structure search database by enumerating
  scaffold × building-block conjugates — e.g., bile acid cores × amino-acid/acyl fragments
  — to feed downstream structure annotation.
when_to_use_negative:
- The target structures are already covered by reference spectral databases (MassBank,
  GNPS, NIST 2017) or the HMDB structure database — COSMIC's novelty pipeline explicitly
  excluded hits with structures in HMDB.
- Exact stereochemistry must be resolved — fragmentation spectra cannot establish
  stereochemistry, so only the two-dimensional structure is considered during enumeration.
- The task is annotation/ranking of existing spectra rather than generating candidate
  structures — use CSI:FingerID ranking and the COSMIC confidence-score workflow instead
  of enumeration.
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0092
tools:
- name: SmiLib v2.0
  role: 'Exhaustive combinatorial enumeration engine: attaches amino-acid/acyl building
    blocks at conjugation positions and phase-2 modification sites of the 322 bile
    acid scaffolds'
- name: COSMIC
  role: 'Downstream consumer: the enumerated 28,630-structure database serves as the
    in silico structure database for COSMIC structure annotation and confidence scoring'
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: Framework integrating COSMIC/CSI:FingerID annotation of LC–MS/MS runs against
    the enumerated candidate database
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:8e1c25c7ccf9705713b2de480c6c255138777316bc3616c67dcb1736a9d2083e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - SIRIUS 4
  - SmiLib v2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# combinatorial-structure-enumeration

## Summary

Exhaustively enumerate all plausible conjugate structures by combinatorially attaching building blocks (91 amino-acid/acyl fragments) onto curated core scaffolds (322 bile acid cores) with SmiLib v2.0, producing the 28,630-structure in silico search database used by COSMIC.

## When to use

Use when building an in silico structure search database by enumerating scaffold × building-block conjugates — e.g., bile acid cores × amino-acid/acyl fragments — to feed downstream structure annotation.

## When NOT to use

- The target structures are already covered by reference spectral databases (MassBank, GNPS, NIST 2017) or the HMDB structure database — COSMIC's novelty pipeline explicitly excluded hits with structures in HMDB.
- Exact stereochemistry must be resolved — fragmentation spectra cannot establish stereochemistry, so only the two-dimensional structure is considered during enumeration.
- The task is annotation/ranking of existing spectra rather than generating candidate structures — use CSI:FingerID ranking and the COSMIC confidence-score workflow instead of enumeration.

## Inputs

- Curated bile acid scaffold SMILES set (322 scaffolds: cholic acid, deoxycholic acid, hyocholic acid, chenodeoxycholic acid cores with phase-2 modifications)
- Amino-acid/acyl building-block SMILES list (91 fragments)
- SmiLib v2.0 scaffold and building-block input files and combinatorial enumeration configuration

## Outputs

- bile_acid_conjugates.smi — 28,630 enumerated conjugate structures as canonical SMILES, one per line
- Validated in silico structure search database (exactly 28,630 records) for COSMIC annotation

## How to apply

Load the curated bile acid scaffold set — 322 scaffolds spanning cholic acid, deoxycholic acid, hyocholic acid and chenodeoxycholic acid cores with phase-2 modifications — plus the 91 amino-acid/acyl building-block SMILES into SmiLib v2.0 scaffold and building-block input files. Configure the combinatorial enumeration so fragments attach at the conjugation positions of each scaffold core and at the phase-2 modification sites, then run the exhaustive enumeration to generate all scaffold–building-block conjugates. Count the enumerated records and reject the run unless the database contains exactly 28,630 structures, matching the database size COSMIC used for its in silico structure database generation. Write the output to bile_acid_conjugates.smi as canonical SMILES, one structure per line, suitable for downstream COSMIC/CSI:FingerID candidate ranking.

## Related tools

- **SmiLib v2.0** (Exhaustive combinatorial enumeration engine: attaches amino-acid/acyl building blocks at conjugation positions and phase-2 modification sites of the 322 bile acid scaffolds)
- **COSMIC** (Downstream consumer: the enumerated 28,630-structure database serves as the in silico structure database for COSMIC structure annotation and confidence scoring) — https://github.com/sirius-ms/sirius
- **SIRIUS 4** (Framework integrating COSMIC/CSI:FingerID annotation of LC–MS/MS runs against the enumerated candidate database) — https://github.com/sirius-ms/sirius

## Examples

```
smililib -c config.ini && wc -l bile_acid_conjugates.smi  # must print 28630
```

## Evaluation signals

- Output file bile_acid_conjugates.smi exists and its structure row count equals exactly 28,630; reject the run if the count differs.
- Every record is a canonical SMILES on its own line, each combining exactly one scaffold core with allowed building blocks attached at conjugation/phase-2 positions.
- Compounds beyond 700 Da mass were discarded before search-database use, so enumerated candidates entering COSMIC should respect the ≤700 Da window.
- Downstream sanity check: COSMIC annotations of the top 12 bile acid conjugates from this database were >90% correct, and 12 natural bile acids were annotated (9 confirmed by manual evaluation, 2 by synthetic standards).

## Limitations

- Enumeration covers only the curated chemical space of the 322 scaffolds and 91 building blocks; structures outside that scaffold/fragment set are not represented.
- Stereochemistry is not resolved — consistent with COSMIC's design, only the two-dimensional structure is considered since stereochemistry cannot be established from fragmentation spectra.
- The structure count is a strict invariant (28,630): any change to scaffolds, building blocks, or attachment-site configuration invalidates comparability with the published COSMIC database.

## Evidence

- [abstract] COSMIC in silico database generation: "Here we introduce the COSMIC workflow that combines in silico structure database generation and annotation with a confidence score"
- [other] Enumeration engine: [evidence span withheld — anchor status: not_found]
- [other] Validation invariant: [evidence span withheld — anchor status: not_found]
- [discussion] Stereochemistry limitation: "Establishing the stereochemistry from fragmentation spectra is highly challenging and beyond the power of automated search engines; hence, only the two-dimensional structure is considered"
