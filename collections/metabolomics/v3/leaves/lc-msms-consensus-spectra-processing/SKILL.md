---
name: lc-msms-consensus-spectra-processing
description: Use when a multi-sample LC-MS/MS untargeted metabolomics data set (e.g.
  MSV000080502 Euphorbia dendroides extract) must be turned into clean consensus spectra
  with ionization variants (in-source fragments, adducts, dimers) merged into one
  representative [M+H]+ ion.
when_to_use_negative:
- The consensus spectra are already deconvoluted with a single [M+H]+ per precursor
  group — no ionization variant merging is needed
- The goal is spectral library search (Library Identification step 6 via Tremolo /
  GNPS2 Library Search) rather than grouping ionization variants
- Data are MS1-only or targeted metabolomics without MS2 spectra to cluster into consensus
  spectra
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NP3 MS Workflow
  role: 'Executes the full pipeline: clustering/cleaning consensus spectra (NP3-shifted
    cosine) and IVAMN construction with PageRank [M+H]+ selection; output written
    as GraphML'
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Cytoscape
  role: Graph visualization tool used to open and inspect the resulting GraphML molecular
    networks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:67e222afee4d1874706edcd6d623c50d4ddd64cb2bb71063a760e8d68707a09f
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# lc-msms-consensus-spectra-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

NP3 MS Workflow processing of LC-MS/MS untargeted metabolomics data: clustering noisy per-sample MS2 spectra into clean consensus spectra (Clustering/Quantification/Cleaning, steps 3-5) and deconvoluting in-source/adduct ionization variants of the same precursor into a single [M+H]+ ion entity via an ionization variant annotation molecular network (IVAMN).

## When to use

Use when a multi-sample LC-MS/MS untargeted metabolomics data set (e.g. MSV000080502 Euphorbia dendroides extract) must be turned into clean consensus spectra with ionization variants (in-source fragments, adducts, dimers) merged into one representative [M+H]+ ion.

## When NOT to use

- The consensus spectra are already deconvoluted with a single [M+H]+ per precursor group — no ionization variant merging is needed
- The goal is spectral library search (Library Identification step 6 via Tremolo / GNPS2 Library Search) rather than grouping ionization variants
- Data are MS1-only or targeted metabolomics without MS2 spectra to cluster into consensus spectra

## Inputs

- Per-sample MGF files (Mascot Generic Format) of clean consensus spectra from NP3 MS Workflow steps 3-5
- Quantification table of precursor ions per sample
- Ionization rules table encoding expected positive-mode mass shifts (in-source fragments, adducts, dimers)
- Metadata Table describing LC-MS/MS samples and sample groups (step 1)

## Outputs

- IVAMN GraphML network (e.g. ivamn_msv000080502.graphml) with [M+H]+ PageRank selections encoded as node/edge attributes
- One [M+H]+ representative ion per ionization variant group
- Updated deconvoluted quantification table keyed on representative [M+H]+ ions

## How to apply

Start from NP3 MS Workflow outputs: per-sample MGF files of clean consensus spectra (produced by Preprocessing step 2 and Clustering/Quantification/Cleaning steps 3-5, which cluster preprocessed MS2 data using the NP3-shifted cosine function, a greedy version of the modified cosine) plus the corresponding quantification table. Build an ionization rules table encoding expected positive-mode mass shifts for in-source fragments, adducts, and dimers of each consensus precursor. In the Annotation/Merging steps 7-8, perform pairwise annotation: connect consensus spectra whose m/z differences and fragmentation relationships match an ionization rules entry, building the IVAMN graph from these links. Run the PageRank algorithm on the IVAMN to select the [M+H]+ representative ion for each variant group, and write the network as GraphML with the selections encoded as node/edge attributes. Follow the repository's reproducibility convention (version 1.4.2+): write IVAMN components largest-first and sort edges by source node ID in numerical ordering.

## Related tools

- **NP3 MS Workflow** (Executes the full pipeline: clustering/cleaning consensus spectra (NP3-shifted cosine) and IVAMN construction with PageRank [M+H]+ selection; output written as GraphML) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Cytoscape** (Graph visualization tool used to open and inspect the resulting GraphML molecular networks)

## Evaluation signals

- ivamn_msv000080502.graphml exists and is a well-formed GraphML document loadable in Cytoscape
- Every variant cluster in the IVAMN contains exactly one PageRank-selected [M+H]+ representative ion
- m/z differences between linked consensus spectra match entries in the positive-mode ionization rules table (in-source fragments, adducts, dimers)
- Reproducibility check: IVAMN components written largest-first and edges sorted by source node ID in numerical ordering (NP3 v1.4.2 convention)
- Regression check: when the noise cutoff is applied during join_jobs Step 7, the filtered IVAMN resets its index to prevent matching errors (bug fixed in v1.5.1)

## Limitations

- Versions prior to 1.5.1 contain a bug in join_jobs Step 7: applying the noise cutoff to the filtered IVAMN without resetting its index causes matching errors when joining IVAMNs
- Since v1.5.0, [M+H]+ networks are created without culture media by default — m/z appearing in bed samples are removed from protonated networks, which may differ from older default behavior
- Ionization rules described here target positive mode; negative-mode processing uses separate ion-mode-specific libraries and settings (v1.6.0 split the ALL_GNPS MGF library by ion mode)

## Evidence
