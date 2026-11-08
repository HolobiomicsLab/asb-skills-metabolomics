---
name: cross-strain-metabolome-comparison
description: Use when an annotated LC-MS/MS feature table (e.g. from GNPS2 FBMN with
  ChemWalker and SIRIUS annotations) must be assigned metabolic pathways via KEGG
  cross-referencing and compared across strains to find common pathways.
when_to_use_negative:
- The input is a raw MS file or an unannotated MZmine feature list — pathway cross-referencing
  requires compounds already annotated through GNPS2 FBMN, ChemWalker, and SIRIUS
- Only one strain is available — the skill's core output is a cross-strain comparison
  of shared pathways, which cannot be computed from a single feature table
- The goal is spectral library matching or annotation propagation itself (use GNPS2
  FBMN/ChemWalker workflows directly) rather than assigning KEGG pathways to already-annotated
  compounds
edam_operation: http://edamontology.org/operation_0226
tools:
- name: MZmine
  role: Preprocesses the LC-MS/MS full-scan and MS/MS data (mass detection, ADAP chromatogram
    building, deconvolution, Join aligner) and exports the feature table that carries
    the annotated compounds to be KEGG-mapped
- name: ChemWalker
  role: Propagates annotations across the FBMN molecular network topology via its
    GNPS2 graphical interface, providing the annotated compounds that the KEGG cross-referencing
    step consumes
- name: SIRIUS
  role: Provides the second layer of in silico annotation (version 5.8.5) whose compound
    identities are the candidates mapped to KEGG entries
- name: KEGG
  role: Reference database used to cross-reference each annotated molecule to a compound
    entry and retrieve its metabolic pathway assignment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:9f7d42f38ce77623acb5bbd98bcd80e2f04a88e97578c9b914c63a137e2e3ef6
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-strain-metabolome-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps annotated molecular features from multiple bacterial strains (here actinomycetes BRA006, BRA010, BRA177) to KEGG compound entries, attaches their KEGG pathway assignments, and compares pathways across strains to find shared pathways with differing intermediates, such as purine metabolism.

## When to use

Use when an annotated LC-MS/MS feature table (e.g. from GNPS2 FBMN with ChemWalker and SIRIUS annotations) must be assigned metabolic pathways via KEGG cross-referencing and compared across strains to find common pathways.

## When NOT to use

- The input is a raw MS file or an unannotated MZmine feature list — pathway cross-referencing requires compounds already annotated through GNPS2 FBMN, ChemWalker, and SIRIUS
- Only one strain is available — the skill's core output is a cross-strain comparison of shared pathways, which cannot be computed from a single feature table
- The goal is spectral library matching or annotation propagation itself (use GNPS2 FBMN/ChemWalker workflows directly) rather than assigning KEGG pathways to already-annotated compounds

## Inputs

- Annotated feature table (.csv exported from MZmine/GNPS2 FBMN) for each strain (BRA006, BRA010, BRA177) with ChemWalker-propagated and SIRIUS v5.8.5 annotations
- KEGG compound database (Kanehisa and Goto, 2000) for cross-referencing compound entries and pathway assignments

## Outputs

- .csv table listing the 39 KEGG-mapped annotated compounds (9 BRA006, 10 BRA010, 20 BRA177) with their KEGG pathway assignments
- Cross-strain pathway comparison flagging pathways common to all three strains and their differing annotated intermediates (e.g. purine metabolism)

## How to apply

Load the annotated feature tables exported as .csv from MZmine preprocessing and the GNPS2 FBMN workflow, where annotations were propagated by ChemWalker through its GNPS2 interface and layered with SIRIUS version 5.8.5 in silico annotation. For each strain (BRA006, BRA010, BRA177), map every annotated compound to a KEGG compound entry using the KEGG database (Kanehisa and Goto, 2000), then retrieve the KEGG pathway assignment for every mapped compound and attach it to its feature row. Compare pathway assignments across the three strains to identify pathways common to all of them, and inspect whether the annotated intermediates of a shared pathway (e.g. purine metabolism) differ between strains. Export a .csv table listing the KEGG-mapped annotated compounds with their pathway assignments and strain-level occurrence; the reference analysis resolved 39 compounds (9 BRA006, 10 BRA010, 20 BRA177) and flagged purine metabolism as common across strains with different intermediates.

## Related tools

- **MZmine** (Preprocesses the LC-MS/MS full-scan and MS/MS data (mass detection, ADAP chromatogram building, deconvolution, Join aligner) and exports the feature table that carries the annotated compounds to be KEGG-mapped)
- **ChemWalker** (Propagates annotations across the FBMN molecular network topology via its GNPS2 graphical interface, providing the annotated compounds that the KEGG cross-referencing step consumes)
- **SIRIUS** (Provides the second layer of in silico annotation (version 5.8.5) whose compound identities are the candidates mapped to KEGG entries)
- **KEGG** (Reference database used to cross-reference each annotated molecule to a compound entry and retrieve its metabolic pathway assignment)

## Evaluation signals

- Output .csv must contain exactly 39 annotated compound rows, each carrying a KEGG pathway assignment
- Strain-level occurrence counts must match 9 compounds for BRA006, 10 for BRA010, and 20 for BRA177
- The cross-strain comparison must flag purine metabolism as present in all three strains with differing annotated intermediates
- Every compound row must trace back to an annotation level present in the FBMN/ChemWalker/SIRIUS feature table (no unmapped features carried into the KEGG table)

## Limitations

- Only compounds whose annotations can be matched to KEGG compound entries receive pathway assignments — the remainder of the annotated feature set stays unmapped (39 of the total annotated set)
- Pathway assignments are inherited from upstream annotation quality (GNPS2 FBMN spectral pairing, ChemWalker propagation, SIRIUS in silico annotation), so false annotations propagate into the pathway table
- Detecting 'different intermediates' within a shared pathway depends on which intermediates were annotated per strain, not on exhaustive pathway coverage

## Evidence

- [results] KEGG cross-referencing of annotated molecules: "we cross-referenced the annotation data of each of the molecules contained in the three bacteria with the KEGG database (Kanehisa and Goto, 2000)"
- [methods] ChemWalker annotation propagation input: "propagated annotations based on the topology of the molecular network using the ChemWalker tool (Borelli et al., 2023) through its new graphical interface available on GNPS2"
- [methods] SIRIUS annotation layer: "A second layer of in silico annotation was then performed using the SIRIUS tool (Dührkop et al., 2019) version 5.8.5"
- [results] 39 compounds with pathway counts: [evidence span withheld — anchor status: not_found]
