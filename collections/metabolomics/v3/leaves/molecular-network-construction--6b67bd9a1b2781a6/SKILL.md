---
name: molecular-network-construction
description: Use when building molecular networks from clustered LC–MS/MS consensus
  spectra (per-sample MGF files) to represent ionization variants of the same precursor
  as one [M+H]+ ion entity for dereplication and bioactivity ranking.
when_to_use_negative:
- Input is already a quantification/feature table without MS2 consensus spectra —
  network construction requires per-sample MGF clean consensus spectra
- The goal is spectral library identification (step 6, Tremolo search against GNPS/ISDB
  libraries) rather than network-based grouping of ionization variants
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NP3 MS Workflow
  role: Performs clustering (NP3-shifted cosine), SSMN construction/filtering, IVAMN
    generation via ionization rules, PageRank [M+H]+ selection, and GraphML export
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Cytoscape
  role: Graph visualization tool used to open and inspect the resulting SSMN/IVAMN
    GraphML networks
- name: Mascot Generic Format (MGF)
  role: File format of the per-sample clean consensus MS2 spectra that serve as network
    node inputs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:213f04c3ca14239dfe8af567ce7e1e2f6146e9c9b77937d2991ec04dd5949398
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# molecular-network-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construction of spectral similarity molecular networks (SSMN) and ionization variant annotation molecular networks (IVAMN) from clean consensus MS2 spectra in NP3 MS Workflow, grouping in-source/adduct ionization variants of the same precursor under a single PageRank-selected [M + H]+ representative ion.

## When to use

Use when building molecular networks from clustered LC–MS/MS consensus spectra (per-sample MGF files) to represent ionization variants of the same precursor as one [M+H]+ ion entity for dereplication and bioactivity ranking.

## When NOT to use

- Input is already a quantification/feature table without MS2 consensus spectra — network construction requires per-sample MGF clean consensus spectra
- The goal is spectral library identification (step 6, Tremolo search against GNPS/ISDB libraries) rather than network-based grouping of ionization variants

## Inputs

- Per-sample MGF (Mascot Generic Format) files of clean consensus spectra from NP3 MS Workflow steps 3–5
- Corresponding precursor ion quantification table
- Metadata Table describing LC–MS/MS samples and sample groups
- Ionization rules table encoding expected positive-mode mass shifts (in-source fragments, adducts, dimers)

## Outputs

- Spectral similarity molecular network (SSMN) after link filtering (minimum matched peaks, K-most similar neighbors, damped pruning)
- IVAMN written as a GraphML network with [M+H]+ PageRank selections encoded as node/edge attributes

## How to apply

Start from the NP3 MS Workflow outputs for clustered/cleaned MS2 data (steps 3–5), where clustering relies on the NP3-shifted cosine function (a greedy version of the modified cosine) applied to per-sample MGF files of clean consensus spectra. Build the spectral similarity molecular network (SSMN) from these consensus spectra, then filter it by removing links below the minimum number of matched peaks, keeping only the K-most similar neighbors of each node, and applying damped pruning to limit component size. To group ionization variants, construct an ionization rules table encoding expected positive-mode mass shifts for in-source fragments, adducts, and dimers, and connect consensus spectra whose m/z differences and fragmentation relationships match a rule — this yields the IVAMN. Run the PageRank algorithm on the IVAMN to select the [M + H]+ representative ion for each variant group, then write the network as GraphML with components written largest-first and edges sorted by source node ID for reproducibility. Validate by loading the GraphML in Cytoscape and checking each variant cluster contains exactly one PageRank-selected [M + H]+ ion.

## Related tools

- **NP3 MS Workflow** (Performs clustering (NP3-shifted cosine), SSMN construction/filtering, IVAMN generation via ionization rules, PageRank [M+H]+ selection, and GraphML export) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Cytoscape** (Graph visualization tool used to open and inspect the resulting SSMN/IVAMN GraphML networks)
- **Mascot Generic Format (MGF)** (File format of the per-sample clean consensus MS2 spectra that serve as network node inputs)

## Evaluation signals

- The ivamn GraphML file is well-formed and loadable in Cytoscape
- Every ionization variant cluster contains exactly one PageRank-selected [M + H]+ representative ion
- GraphML components are written largest-first and edges are sorted by source node ID (numerical ordering), matching the reproducibility convention
- Network edges correspond to m/z differences and fragmentation relationships that match an entry in the ionization rules table
- SSMN links below the minimum number of matched peaks have been removed, and each node retains at most its K-most similar neighbors

## Limitations

- Reproducibility of the IVAMN GraphML depends on the fixed component/edge ordering convention introduced in v1.4.2; older outputs may not be reproducible
- In join_jobs (v1.5.1), applying the noise cutoff to filtered IVAMNs previously caused index-matching errors unless the filtered IVAMN reset its index
- By default, [M+H]+ networks exclude m/z ions that appear in bed/culture media samples, which alters which variants are represented
- Variant grouping is restricted to the encoded ionization rules table (positive-mode in-source fragments, adducts, dimers); unmodeled ionization variants will not be merged

## Evidence
