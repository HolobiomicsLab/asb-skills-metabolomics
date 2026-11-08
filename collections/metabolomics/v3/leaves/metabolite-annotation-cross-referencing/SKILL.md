---
name: metabolite-annotation-cross-referencing
description: Use when an annotated feature table (e.g. FBMN + ChemWalker + SIRIUS
  output for actinomycete strains such as BRA006, BRA010, BRA177) needs KEGG compound
  mapping, pathway assignment, and cross-strain pathway comparison.
when_to_use_negative:
- The feature table has no structural/in silico annotations yet (no FBMN spectral
  pairing, ChemWalker propagation, or SIRIUS results) — cross-referencing KEGG requires
  annotations to map from
- The goal is signal detection/feature filtering of unfragmented ions (use RegFilter-based
  feature selection instead)
- Compounds of interest lack KEGG compound entries, since the skill depends entirely
  on KEGG cross-referencing
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3332
tools:
- name: KEGG
  role: Reference database for compound cross-referencing and metabolic pathway assignment
- name: ChemWalker
  role: Propagates annotations based on molecular network topology prior to KEGG cross-referencing,
    via its GNPS2 graphical interface
- name: SIRIUS
  role: Second layer of in silico annotation (v5.8.5) supplying compound annotations
    for KEGG mapping
- name: MZmine
  role: Preprocessing of LC-MS/MS data producing the feature table upstream of annotation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:4738d77c2d82c8c586cc5fed3983e2179eff942bc0860dfeb467be10c04959d4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/computational-chemical-biology/regression_filter
  tools:
  - ChemWalker
  - KEGG
  - MZmine
  - SIRIUS
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolite-annotation-cross-referencing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-references annotated molecular features from LC-MS/MS metabolomics (GNPS2 FBMN with ChemWalker and SIRIUS annotations) against the KEGG database (Kanehisa and Goto, 2000) to assign metabolic pathways, and compares pathway assignments across multiple strains to find shared pathways with different intermediates.

## When to use

Use when an annotated feature table (e.g. FBMN + ChemWalker + SIRIUS output for actinomycete strains such as BRA006, BRA010, BRA177) needs KEGG compound mapping, pathway assignment, and cross-strain pathway comparison.

## When NOT to use

- The feature table has no structural/in silico annotations yet (no FBMN spectral pairing, ChemWalker propagation, or SIRIUS results) — cross-referencing KEGG requires annotations to map from
- The goal is signal detection/feature filtering of unfragmented ions (use RegFilter-based feature selection instead)
- Compounds of interest lack KEGG compound entries, since the skill depends entirely on KEGG cross-referencing

## Inputs

- Annotated molecular feature table for strains BRA006, BRA010, and BRA177 (from GNPS2 FBMN with ChemWalker propagation and SIRIUS in silico annotations)
- KEGG database (Kanehisa and Goto, 2000)

## Outputs

- .csv table listing the 39 annotated compounds with KEGG pathway assignments and strain-level occurrence (nine for BRA006, 10 for BRA010, 20 for BRA177)
- Cross-strain pathway comparison flagging shared pathways (e.g. purine metabolism) and their differing intermediates

## How to apply

Load the annotated feature table produced by the GNPS2 FBMN workflow with ChemWalker annotation propagation (via its GNPS2 graphical interface) and SIRIUS v5.8.5 in silico annotation. Map each annotated compound to a KEGG compound entry using the KEGG database, then retrieve the KEGG pathway assignment for every mapped compound and attach it to the corresponding feature row. Compare pathway assignments across the strains to identify pathways common to all of them; in the reference case, purine metabolic pathways were shared across BRA006, BRA010, and BRA177 while the annotated intermediates differed per strain. Export a .csv table listing each annotated compound with its KEGG pathway assignment and strain-level occurrence.

## Related tools

- **KEGG** (Reference database for compound cross-referencing and metabolic pathway assignment)
- **ChemWalker** (Propagates annotations based on molecular network topology prior to KEGG cross-referencing, via its GNPS2 graphical interface)
- **SIRIUS** (Second layer of in silico annotation (v5.8.5) supplying compound annotations for KEGG mapping)
- **MZmine** (Preprocessing of LC-MS/MS data producing the feature table upstream of annotation)

## Evaluation signals

- The output table must contain exactly 39 annotated compound rows, each carrying a KEGG pathway assignment
- Per-strain counts must match the reference: nine compounds for BRA006, 10 for BRA010, and 20 for BRA177
- The cross-strain comparison must flag purine metabolism as present in all three strains
- The shared purine-metabolism pathway must show different annotated intermediates between strains
- Every feature row in the output must retain its strain-level occurrence information

## Limitations

- Only 39 of the total annotated compounds could be cross-referenced to KEGG entries — compounds absent from KEGG cannot receive pathway assignments
- Coverage depends on upstream annotation quality (GNPS2 library spectral pairing, ChemWalker propagation, SIRIUS in silico annotation)
- Pathway assignments are only as reliable as the underlying compound-level annotations propagated through the molecular network

## Evidence

- [results] From the total set, 39 annotated compounds were identified using KEGG with their respective metabolic pathway associated — nine for BRA006, 10 for BRA010, and 20 for BRA177: [evidence span withheld — anchor status: not_found]
