---
name: in-source-fragment-rule-matching
description: Use when per-sample MGF clean consensus spectra from LC–MS/MS contain
  multiple ionization variants of one precursor (in-source fragments, adducts, dimers)
  that must be collapsed to a single [M + H]+ ion entity before molecular networking.
when_to_use_negative:
- When the goal is a spectral-similarity molecular network (SSMN) rather than merging
  ionization variants of the same precursor
- When input is already a deconvoluted feature table containing only single [M + H]+
  ions per precursor, with no adduct/fragment/dimer multiplicity
- When the rules table does not cover the observed ion mode or chemistry (the article
  encodes positive-mode shifts; negative-mode handling requires an appropriate rules
  table)
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: NP3 MS Workflow
  role: 'Implements the Annotation/Merging steps 7–8: pairwise ionization-variant
    linking via the ionization rules table, IVAMN construction, PageRank [M+H]+ selection,
    and GraphML output'
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Cytoscape
  role: Used to open and visually validate the resulting IVAMN GraphML network
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:6163b0b312cba00cc1697383ab2f12f0968ae022e455e49da523fc5586d19fba
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# in-source-fragment-rule-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Groups ionization variants (in-source fragments, adducts, dimers) of the same precursor by matching m/z differences and fragmentation relationships against an ionization rules table, then uses an ionization variant annotation molecular network (IVAMN) with PageRank to select a single [M + H]+ representative ion per variant group.

## When to use

Use when per-sample MGF clean consensus spectra from LC–MS/MS contain multiple ionization variants of one precursor (in-source fragments, adducts, dimers) that must be collapsed to a single [M + H]+ ion entity before molecular networking.

## When NOT to use

- When the goal is a spectral-similarity molecular network (SSMN) rather than merging ionization variants of the same precursor
- When input is already a deconvoluted feature table containing only single [M + H]+ ions per precursor, with no adduct/fragment/dimer multiplicity
- When the rules table does not cover the observed ion mode or chemistry (the article encodes positive-mode shifts; negative-mode handling requires an appropriate rules table)

## Inputs

- Per-sample MGF (Mascot Generic Format) files of clean consensus spectra clustered with the NP3-shifted cosine function (e.g., from MSV000080502 Euphorbia dendroides data)
- Quantification table of precursor ions per sample
- Ionization rules table encoding positive-mode mass shifts for in-source fragments, adducts, and dimers

## Outputs

- IVAMN GraphML network (e.g., ivamn_msv000080502.graphml) with [M+H]+ PageRank selections encoded as node/edge attributes
- Ionization variant groups, each represented by exactly one [M + H]+ representative ion
- Merged/deconvoluted consensus spectra feedable into downstream SSMN and library identification

## How to apply

Load the NP3 MS Workflow per-sample MGF files of clean consensus spectra (produced by Clustering, Quantification, and Cleaning steps 3–5 using the NP3-shifted cosine function) plus the corresponding quantification table. Construct an ionization rules table encoding expected positive-mode mass shifts for in-source fragments, adducts, and dimers of each consensus precursor. During the Annotation/Merging steps 7–8, connect consensus spectra pairwise whose m/z differences and fragmentation relationships match an entry in the rules table, and build the IVAMN graph from these links. Run the PageRank algorithm on the IVAMN to select the [M + H]+ representative ion for each variant group. Write the IVAMN as GraphML with components ordered largest-first and edges sorted by source node ID for reproducibility; by default (v1.5.0+) the [M+H]+ networks exclude m/z ions present in culture media/bed samples.

## Related tools

- **NP3 MS Workflow** (Implements the Annotation/Merging steps 7–8: pairwise ionization-variant linking via the ionization rules table, IVAMN construction, PageRank [M+H]+ selection, and GraphML output) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Cytoscape** (Used to open and visually validate the resulting IVAMN GraphML network)

## Evaluation signals

- Every ionization variant cluster in the IVAMN contains exactly one PageRank-selected [M + H]+ representative ion
- ivamn_msv000080502.graphml exists, is a well-formed GraphML document, and loads in Cytoscape
- Reproducibility check: IVAMN components are written largest-first and edges are sorted by source node ID (fixed in v1.4.2)
- Each edge in the IVAMN corresponds to an m/z difference and fragmentation relationship matching an entry in the ionization rules table
- Protonated [M+H]+ networks contain no m/z ions detected in culture media/bed samples (default behavior since v1.5.0)

## Limitations

- The ionization rules table encodes expected positive-mode mass shifts; adapting to negative mode requires a new rules table
- Prior to v1.5.1, applying the noise cutoff during join_jobs could leave the filtered IVAMN with a stale index, causing matching errors in subsequent filters
- With aggressive noise cutoffs, low-abundance variant spectra may be removed before merging, splitting variant groups

## Evidence
