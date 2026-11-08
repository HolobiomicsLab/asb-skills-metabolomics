---
name: bile-acid-conjugate-chemistry
description: Use when constructing an in silico structure database of bile acid conjugates
  from scaffold cores plus amino-acid/acyl building blocks, requiring an exact combinatorial
  enumeration count (28,630 structures) as validation.
when_to_use_negative:
- The task is annotating spectra against an already-built structure database with
  SIRIUS 4/CSI:FingerID/COSMIC scoring, not constructing the bile acid conjugate database
  itself.
- The building blocks are not amino-acid/acyl fragments attachable at bile acid conjugation
  positions (e.g. arbitrary natural-product fragments), so the 28,630 combinatorial
  closure does not apply.
- The target output requires resolved 3D stereochemistry; the enumeration yields two-dimensional
  structures only.
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0153
tools:
- name: SmiLib v2.0
  role: 'Combinatorial enumeration engine: attaches the 91 amino-acid/acyl building-block
    SMILES to the 322 bile acid scaffolds at conjugation positions and phase-2 modification
    sites, then writes enumerated records as canonical SMILES.'
- name: COSMIC
  role: 'Downstream consumer of the 28,630-structure conjugate database: the enumerated
    structures form the in silico structure database used for COSMIC confidence-scored
    bile acid annotation.'
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:9d12c03adf22630600b74d65d29be8146f98341b0714c8e552be8517be40aa3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - SmiLib v2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bile-acid-conjugate-chemistry

## Summary

Exhaustively enumerates bile acid amino-acid/acyl conjugate structures by combinatorially attaching 91 building-block SMILES onto 322 curated bile acid scaffolds with SmiLib v2.0, producing the 28,630-structure in silico search database used by COSMIC for bile acid annotation.

## When to use

Use when constructing an in silico structure database of bile acid conjugates from scaffold cores plus amino-acid/acyl building blocks, requiring an exact combinatorial enumeration count (28,630 structures) as validation.

## When NOT to use

- The task is annotating spectra against an already-built structure database with SIRIUS 4/CSI:FingerID/COSMIC scoring, not constructing the bile acid conjugate database itself.
- The building blocks are not amino-acid/acyl fragments attachable at bile acid conjugation positions (e.g. arbitrary natural-product fragments), so the 28,630 combinatorial closure does not apply.
- The target output requires resolved 3D stereochemistry; the enumeration yields two-dimensional structures only.

## Inputs

- Curated bile acid scaffold set: 322 scaffolds covering cholic acid, deoxycholic acid, hyocholic acid and chenodeoxycholic acid cores with phase-2 modifications (SmiLib v2.0 scaffold input file)
- Amino-acid/acyl building-block set: 91 fragment SMILES (SmiLib v2.0 building-block input file)
- SmiLib v2.0 enumeration configuration specifying attachment at scaffold conjugation positions and phase-2 modification sites

## Outputs

- bile_acid_conjugates.smi: 28,630 enumerated bile acid conjugate structures as canonical SMILES, one structure per line
- In silico structure search database used downstream by COSMIC for bile acid conjugate annotation

## How to apply

Load the curated bile acid scaffold set (322 scaffolds spanning cholic acid, deoxycholic acid, hyocholic acid and chenodeoxycholic acid cores with phase-2 modifications) and the 91 amino-acid/acyl building-block SMILES into SmiLib v2.0 scaffold and building-block input files. Configure SmiLib v2.0 so amino-acid/acyl fragments attach at the conjugation positions of each scaffold core and at the phase-2 modification sites, then run the exhaustive combinatorial enumeration to generate all scaffold-building-block conjugate structures. Count the enumerated records and reject the run if the database does not contain exactly 28,630 structures. Write the enumerated conjugates as canonical SMILES, one structure per line, to bile_acid_conjugates.smi; this file is the structure database size used for COSMIC's in silico structure database generation, against which 12 natural bile acids were subsequently annotated.

## Related tools

- **SmiLib v2.0** (Combinatorial enumeration engine: attaches the 91 amino-acid/acyl building-block SMILES to the 322 bile acid scaffolds at conjugation positions and phase-2 modification sites, then writes enumerated records as canonical SMILES.)
- **COSMIC** (Downstream consumer of the 28,630-structure conjugate database: the enumerated structures form the in silico structure database used for COSMIC confidence-scored bile acid annotation.) — https://github.com/sirius-ms/sirius

## Evaluation signals

- Output file bile_acid_conjugates.smi exists and its structure row count equals exactly 28,630; any other count rejects the run.
- Every line of the .smi file parses as a valid canonical SMILES (one structure per line, no malformed records).
- Enumeration covers all scaffold-building-block combinations: 322 scaffolds with fragments attached at conjugation positions and phase-2 modification sites.
- The generated database matches the structure database size used for COSMIC's in silico structure database generation.
- Downstream sanity check: COSMIC's top 12 bile acid conjugate annotations over this database achieved >90% correctness when validated.

## Limitations

- Only two-dimensional structures are enumerated; establishing stereochemistry from fragmentation spectra is beyond automated search engines, so stereoisomeric bile acid conjugates are not distinguished.
- The structure count is a hard invariant: any enumeration configuration drift (scaffold or building-block set changes) breaks the exact 28,630-structure validation and the run must be rejected.
- Small structure modifications among the enumerated conjugates can be hard or impossible to tell apart using MS/MS data alone, limiting distinguishability of closely related conjugates during downstream COSMIC annotation.

## Evidence

- [results] Enumeration yield: [evidence span withheld — anchor status: not_found]
- [discussion] Annotation quality of enumerated conjugates: "more than 90% of the annotations for the top 12 bile acid conjugates suggested by COSMIC turned out to be correct."
- [abstract] Natural bile acids annotated with the database: "we annotated 12 natural bile acids. The annotation of nine structures was confirmed by manual evaluation and two structures using synthetic standards"
- [readme] COSMIC integration in SIRIUS: "SIRIUS integrates a collection of our tools, including CSI:FingerID (with COSMIC)"
