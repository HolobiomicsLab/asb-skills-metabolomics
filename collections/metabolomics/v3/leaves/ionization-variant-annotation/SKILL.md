---
name: ionization-variant-annotation
description: Use when processing LC–MS/MS untargeted metabolomics data where the same
  precursor appears as multiple ionization variants (in-source fragments, adducts,
  dimers) and a single deconvoluted [M + H]+ ion entity is needed per compound before
  molecular networking or quantification.
when_to_use_negative:
- When the input is raw vendor LC–MS/MS data that has not yet been preprocessed, clustered,
  quantified, and cleaned by NP3 MS Workflow steps 2–5
- When analyzing data where in-source fragments and adducts have already been deconvoluted
  by another method, making variant merging redundant
- For negative-ion-mode data without first adapting the ionization rules table, since
  the described rules encode expected positive-mode mass shifts
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NP3 MS Workflow
  role: Produces the clean consensus spectra (MGF), quantification table, and performs
    Annotation/Merging steps 7–8 that build the IVAMN and run PageRank for [M + H]+
    selection
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Mascot Generic Format (MGF)
  role: File format of the per-sample clean consensus spectra that serve as input
    to the IVAMN construction
- name: Cytoscape
  role: Graph visualization tool used to open and inspect the resulting GraphML IVAMN
    network
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:a9cd1f28ad6ffe91688b39776c7fc486eb42cca8dd3abce468ad6b65792c312b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/danielatrivella/NP3_MS_Workflow
  repo_url: https://github.com/danielatrivella/NP3_MS_Workflow
  tools:
  - Cytoscape
  - Mascot Generic Format (MGF)
  - NP3 MS Workflow
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# ionization-variant-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds an ionization variant annotation molecular network (IVAMN) that groups in-source fragments, adducts, and dimers of the same precursor from NP3 MS Workflow clean consensus spectra, then applies PageRank to select the [M + H]+ representative ion per variant group.

## When to use

Use when processing LC–MS/MS untargeted metabolomics data where the same precursor appears as multiple ionization variants (in-source fragments, adducts, dimers) and a single deconvoluted [M + H]+ ion entity is needed per compound before molecular networking or quantification.

## When NOT to use

- When the input is raw vendor LC–MS/MS data that has not yet been preprocessed, clustered, quantified, and cleaned by NP3 MS Workflow steps 2–5
- When analyzing data where in-source fragments and adducts have already been deconvoluted by another method, making variant merging redundant
- For negative-ion-mode data without first adapting the ionization rules table, since the described rules encode expected positive-mode mass shifts

## Inputs

- Per-sample MGF files of clean consensus spectra from NP3 MS Workflow steps 3–5 (e.g., MSV000080502 Euphorbia dendroides data set)
- NP3 MS Workflow quantification table of precursor ions
- Ionization rules table encoding expected positive-mode mass shifts for in-source fragments, adducts, and dimers

## Outputs

- IVAMN GraphML network file (e.g., ivamn_msv000080502.graphml) with [M+H]+ representative ion selections encoded as node/edge attributes
- Deconvoluted set of consensus spectra where each ionization variant group maps to exactly one [M + H]+ representative ion

## How to apply

Load the per-sample MGF files of clean consensus spectra produced by NP3 MS Workflow steps 3–5 (clustered with the NP3-shifted cosine function) plus the quantification table. Build an ionization rules table encoding expected positive-mode mass shifts for in-source fragments, adducts, and dimers of each consensus precursor. In Annotation/Merging steps 7–8, pairwise-connect consensus spectra whose m/z differences and fragmentation relationships match a rules-table entry, then construct the IVAMN graph from these pairwise links. Run the PageRank algorithm on the IVAMN to select the [M + H]+ representative ion for each ionization variant group. Write the IVAMN as a GraphML network with [M+H]+ selections encoded as node/edge attributes, ordering components largest-first and sorting edges by source node ID for reproducibility.

## Related tools

- **NP3 MS Workflow** (Produces the clean consensus spectra (MGF), quantification table, and performs Annotation/Merging steps 7–8 that build the IVAMN and run PageRank for [M + H]+ selection) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Mascot Generic Format (MGF)** (File format of the per-sample clean consensus spectra that serve as input to the IVAMN construction)
- **Cytoscape** (Graph visualization tool used to open and inspect the resulting GraphML IVAMN network)

## Examples

```
nextflow run danielatrivella/NP3_MS_Workflow --cmd run --metadata metadata.tsv --ion_mode positive --project msv000080502
```

## Evaluation signals

- The IVAMN output (e.g., ivamn_msv000080502.graphml) exists and is a well-formed GraphML document loadable in Cytoscape
- Every ionization variant cluster in the IVAMN contains exactly one PageRank-selected [M + H]+ representative ion
- GraphML components are written largest-first and edges are sorted by source node ID in numerical ordering, matching the reproducibility convention
- Each edge in the IVAMN corresponds to a pairwise variant link whose m/z difference and fragmentation relationship match an entry in the ionization rules table
- After the noise cutoff in join_jobs step 7, the filtered IVAMN resets its index to prevent matching errors in subsequent filters

## Limitations

- When joining IVAMNs (join_jobs step 7), applying the noise cutoff requires resetting the filtered IVAMN's index, otherwise matching errors occur in the next filters
- The ionization rules table encodes expected positive-mode mass shifts; negative-mode analyses require adapted rules
- PageRank selects a single [M + H]+ representative per variant group, so misassignment propagates if the ionization rules table is incomplete or the fragmentation relationships are ambiguous

## Evidence
