---
name: purine-metabolism-pathway-interpretation
description: Use when annotated LC-MS/MS feature tables (GNPS2 FBMN with ChemWalker
  propagation and SIRIUS annotations) need KEGG pathway assignment and cross-strain
  pathway comparison, e.g. testing whether purine metabolism is shared but with strain-specific
  intermediates.
when_to_use_negative:
- Input is raw LC-MS/MS data (.mzXML) without prior MZmine preprocessing, FBMN molecular
  networking, and compound-level annotation — pathway interpretation requires already-annotated
  features
- The goal is selecting which precursor ions to fragment (DDA gap-filling) — that
  is the RegFilter feature-selection skill, not pathway interpretation
- Compounds have no KEGG cross-references (e.g., purely novel or library-only spectral
  matches) — the KEGG mapping step cannot assign pathways to them
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: KEGG
  role: Reference database used to cross-reference each annotated molecule to a compound
    entry and retrieve its metabolic pathway assignment
- name: ChemWalker
  role: Propagates annotations based on molecular network topology via its GNPS2 graphical
    interface, producing the annotated compounds that are subsequently mapped to KEGG
    pathways
- name: MZmine
  role: Preprocessed the full scan and MS/MS data (mass detection, ADAP, chromatogram
    deconvolution, join aligner) that underlie the exported feature table submitted
    to GNPS-FBMN
- name: SIRIUS
  role: Second-layer in silico annotation (version 5.8.5) that enriches the compound
    identities subsequently cross-referenced against KEGG
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:c4d7a58a03a4897c293bbc10193e3645f824fb2225dd70a1c6cbef5b64701ea4
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# purine-metabolism-pathway-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-references annotated molecular features from the three actinomycete strains (BRA006, BRA010, BRA177) against the KEGG database to assign metabolic pathways, then compares pathway occurrences across strains to find shared pathways with differing intermediates (e.g., purine metabolism).

## When to use

Use when annotated LC-MS/MS feature tables (GNPS2 FBMN with ChemWalker propagation and SIRIUS annotations) need KEGG pathway assignment and cross-strain pathway comparison, e.g. testing whether purine metabolism is shared but with strain-specific intermediates.

## When NOT to use

- Input is raw LC-MS/MS data (.mzXML) without prior MZmine preprocessing, FBMN molecular networking, and compound-level annotation — pathway interpretation requires already-annotated features
- The goal is selecting which precursor ions to fragment (DDA gap-filling) — that is the RegFilter feature-selection skill, not pathway interpretation
- Compounds have no KEGG cross-references (e.g., purely novel or library-only spectral matches) — the KEGG mapping step cannot assign pathways to them

## Inputs

- Annotated feature table (.csv) for strains BRA006, BRA010, and BRA177 from GNPS2 FBMN with ChemWalker propagation and SIRIUS annotations
- KEGG database (Kanehisa and Goto, 2000) compound and pathway entries

## Outputs

- .csv table of 39 annotated compounds with KEGG pathway assignments and strain-level occurrence
- Cross-strain pathway comparison flagging purine metabolism as present in all three strains with differing intermediates

## How to apply

Load the annotated feature table for strains BRA006, BRA010, and BRA177 produced by the GNPS2 FBMN workflow with ChemWalker annotation propagation and SIRIUS in silico annotation (SIRIUS version 5.8.5). Map each annotated compound to a KEGG compound entry and retrieve its KEGG pathway assignment, attaching the pathway to the corresponding feature row. Compare pathway assignments across the three strains to identify pathways common to all of them, verifying that purine metabolic pathways appear in all three while the annotated intermediates differ. Export a .csv table listing the 39 annotated compounds (nine for BRA006, 10 for BRA010, and 20 for BRA177) with their KEGG pathway assignments and strain-level occurrence.

## Related tools

- **KEGG** (Reference database used to cross-reference each annotated molecule to a compound entry and retrieve its metabolic pathway assignment)
- **ChemWalker** (Propagates annotations based on molecular network topology via its GNPS2 graphical interface, producing the annotated compounds that are subsequently mapped to KEGG pathways)
- **MZmine** (Preprocessed the full scan and MS/MS data (mass detection, ADAP, chromatogram deconvolution, join aligner) that underlie the exported feature table submitted to GNPS-FBMN)
- **SIRIUS** (Second-layer in silico annotation (version 5.8.5) that enriches the compound identities subsequently cross-referenced against KEGG)

## Evaluation signals

- Output .csv must contain exactly 39 annotated compound rows, each carrying a KEGG pathway assignment
- Per-strain annotated-compound counts must match 9 (BRA006), 10 (BRA010), and 20 (BRA177)
- Cross-strain comparison must flag purine metabolism as present in all three strains (BRA006, BRA010, BRA177) with differing annotated intermediates
- Every output row includes strain-level occurrence information traceable to the source annotated feature table

## Limitations

- Only compounds that map to KEGG entries receive pathway assignments — the majority of annotated features may remain unassigned (39 out of the broader annotated set)
- Annotation quality depends on the stochastic nature of DDA fragmentation and library/in silico annotation confidence, which propagates into the KEGG interpretation
- Pathway comparison is limited to the three strains studied (BRA006, BRA010, BRA177) and their specific intermediates

## Evidence
