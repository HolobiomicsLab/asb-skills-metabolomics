---
name: kegg-pathway-mapping
description: Use when an annotated feature table from LC-MS/MS metabolomics (e.g.,
  GNPS2 FBMN with ChemWalker/SIRIUS annotations) needs KEGG compound-to-pathway assignment
  and cross-strain pathway comparison.
when_to_use_negative:
- The feature table has no compound annotations (e.g., raw MZmine CSV without GNPS2
  FBMN/ChemWalker/SIRIUS annotation layers) — KEGG cross-referencing depends on already-annotated
  molecules
- The goal is spectral library matching or molecular networking itself rather than
  pathway-level interpretation of existing annotations
- Features of interest are outside KEGG coverage (e.g., novel natural products without
  KEGG compound entries)
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MZmine
  role: Preprocessed full-scan and MS/MS data and exported the feature table (CSV,
    GNPS-FBMN export with 'Only with MS2' filter) that serves as the input feature
    table
- name: ChemWalker
  role: Propagated annotations over the molecular network topology via its GNPS2 interface,
    providing the annotated compounds that are subsequently cross-referenced to KEGG
- name: SIRIUS
  role: Second layer of in silico annotation (v5.8.5) that enriches the feature table
    with molecular identities before KEGG mapping
- name: KEGG
  role: Reference database used to assign each annotated compound to its metabolic
    pathway (Kanehisa and Goto, 2000)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:d887afb46125d0b7e9895af67dd1b22631b38111fe826ab81f4758626e6a5ff2
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
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# kegg-pathway-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-references annotated molecular features from LC-MS/MS metabolomics of actinomycete strains (BRA006, BRA010, BRA177) against the KEGG database to assign metabolic pathways and compare shared pathways across strains. In the source study it mapped 39 annotated compounds to KEGG pathways, revealing shared purine metabolism with strain-specific intermediates.

## When to use

Use when an annotated feature table from LC-MS/MS metabolomics (e.g., GNPS2 FBMN with ChemWalker/SIRIUS annotations) needs KEGG compound-to-pathway assignment and cross-strain pathway comparison.

## When NOT to use

- The feature table has no compound annotations (e.g., raw MZmine CSV without GNPS2 FBMN/ChemWalker/SIRIUS annotation layers) — KEGG cross-referencing depends on already-annotated molecules
- The goal is spectral library matching or molecular networking itself rather than pathway-level interpretation of existing annotations
- Features of interest are outside KEGG coverage (e.g., novel natural products without KEGG compound entries)

## Inputs

- Annotated feature table (.csv) for strains BRA006, BRA010, and BRA177 from GNPS2 FBMN with ChemWalker annotation propagation
- SIRIUS 5.8.5 in silico annotation results merged into the feature table
- KEGG database (compound and pathway entries, Kanehisa and Goto, 2000)

## Outputs

- .csv table listing the 39 KEGG-annotated compounds with KEGG pathway assignments and strain-level occurrence
- Cross-strain comparison flagging pathways common to all three strains with their differing intermediates

## How to apply

Load the annotated feature table produced by the GNPS2 FBMN workflow, whose annotations were propagated by ChemWalker and supplemented with a second in silico annotation layer from SIRIUS 5.8.5. Map each annotated compound to a KEGG compound entry using the KEGG database (Kanehisa and Goto, 2000), then retrieve and attach the KEGG pathway assignment to each feature row. Compare pathway assignments across the strains (BRA006, BRA010, BRA177) to identify pathways common to all of them, and inspect which annotated intermediates occupy those shared pathways. In the source study this yielded 39 KEGG-annotated compounds (9 BRA006, 10 BRA010, 20 BRA177) with purine metabolism flagged as shared but with differing intermediates per strain.

## Related tools

- **MZmine** (Preprocessed full-scan and MS/MS data and exported the feature table (CSV, GNPS-FBMN export with 'Only with MS2' filter) that serves as the input feature table)
- **ChemWalker** (Propagated annotations over the molecular network topology via its GNPS2 interface, providing the annotated compounds that are subsequently cross-referenced to KEGG)
- **SIRIUS** (Second layer of in silico annotation (v5.8.5) that enriches the feature table with molecular identities before KEGG mapping)
- **KEGG** (Reference database used to assign each annotated compound to its metabolic pathway (Kanehisa and Goto, 2000))

## Evaluation signals

- Output table contains exactly 39 annotated compound rows, each carrying a KEGG pathway assignment
- Per-strain KEGG-annotated compound counts match 9 (BRA006), 10 (BRA010), and 20 (BRA177)
- Cross-strain comparison flags purine metabolic pathway as present in all three strains
- Annotated intermediates within the shared purine pathway differ between strains (verifiable per feature row)
- Every mapped compound row resolves to a valid KEGG compound entry

## Limitations

- Only 39 of the total annotated compounds could be assigned KEGG pathways — KEGG cross-referencing covers a subset, not the full feature list
- Mapping quality is bounded by the upstream annotation layers (GNPS2 library spectral pairing, ChemWalker propagation, SIRIUS 5.8.5 in silico annotation)
- Shared pathways can have strain-specific intermediates, so pathway-level overlap alone does not imply identical metabolite complements and requires intermediate-level inspection

## Evidence
