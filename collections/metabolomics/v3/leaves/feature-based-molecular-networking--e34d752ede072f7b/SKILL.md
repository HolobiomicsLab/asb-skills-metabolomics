---
name: feature-based-molecular-networking
description: Use when an LC–MS/MS untargeted metabolomics data set must be processed
  into a filtered spectral similarity molecular network and benchmarked against GNPS
  FBMN with paper-equivalent parameters, e.g., MSV000080502.
when_to_use_negative:
- Input is already a processed feature table or annotated network — this skill covers
  the full LC–MS/MS processing-through-networking pipeline, not re-analysis of finished
  outputs
- The analysis targets targeted metabolomics or requires absolute quantification —
  NP3 is optimized for untargeted, natural-product drug-discovery workflows with relative
  precursor-ion quantification
- Only a single tool's network is needed without a GNPS FBMN head-to-head comparison
  — the reproduction indicators (exclusive features, duplicated clusters) are defined
  relative to paired runs
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NP3 MS Workflow
  role: Executes steps 2–10 (preprocessing, clustering, quantification, cleaning,
    library identification, IVAMN [M+H]+ selection, filtered SSMN) producing the network
    under evaluation
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: GNPS (Global Natural Products Social Molecular Networking)
  role: Provides the FBMN reference workflow run on the same data set with the paper's
    parameters for comparison
- name: Tremolo
  role: Performs Library Identification (step 6), searching clean consensus spectra
    against the In Silico MS2 DataBase (ISDB)
- name: Cytoscape
  role: Graph visualization tool for opening and inspecting the resulting SSMN
- name: Mascot Generic Format (MGF)
  role: Per-sample MS2 file format produced by preprocessing and used for spectral
    clustering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:eda6c57fb121fc3d1309b6c021ac6dd9f26ccabab2dc7fb7b36e0000fca8babc
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
  - GNPS (Global Natural Products Social Molecular Networking)
  - Mascot Generic Format (MGF)
  - NP3 MS Workflow
  - Tremolo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# feature-based-molecular-networking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce and compare feature-based molecular networking outputs between NP3 MS Workflow's filtered SSMN and GNPS FBMN on an LC–MS/MS data set (e.g., Euphorbia dendroides MSV000080502), quantifying filtered node counts, single-node rates, duplicated consensus spectra, and library annotation rates.

## When to use

Use when an LC–MS/MS untargeted metabolomics data set must be processed into a filtered spectral similarity molecular network and benchmarked against GNPS FBMN with paper-equivalent parameters, e.g., MSV000080502.

## When NOT to use

- Input is already a processed feature table or annotated network — this skill covers the full LC–MS/MS processing-through-networking pipeline, not re-analysis of finished outputs
- The analysis targets targeted metabolomics or requires absolute quantification — NP3 is optimized for untargeted, natural-product drug-discovery workflows with relative precursor-ion quantification
- Only a single tool's network is needed without a GNPS FBMN head-to-head comparison — the reproduction indicators (exclusive features, duplicated clusters) are defined relative to paired runs

## Inputs

- LC–MS/MS raw data set from MassIVE (e.g., MSV000080502, Euphorbia dendroides plant extract, 16 samples)
- Metadata Table describing LC–MS/MS samples and sample groups
- GNPS FBMN job output for the same data set (paper-equivalent parameters)
- Library search references: In Silico MS2 DataBase (ISDB) and GNPS spectral library entries (e.g., CCMSLIB00000840551)

## Outputs

- Filtered SSMN from NP3 MS Workflow (viewable in Cytoscape)
- Comparison metrics CSV (e.g., case_study1_np3_vs_fbmn_comparison.csv): nodes 1413 vs 765 (1.9x), single nodes 35.32% vs 44.71%, duplicated consensus spectra 1.5% vs 0.4%, annotation rates 5.1% vs 5.4%, 51.2% NP3-exclusive features
- Per-sample MS2 files in Mascot Generic Format (MGF) and clean count tables from NP3 preprocessing

## How to apply

Download the LC–MS/MS data set (e.g., MSV000080502, 16 Euphorbia dendroides samples) from MassIVE and construct the Metadata Table describing samples and sample groups (NP3 step 1). Execute NP3 MS Workflow steps 2–10 via the `run` command — preprocessing, clustering, quantification, cleaning with the NP3-shifted cosine function, Tremolo/ISDB library identification, IVAMN [M+H]+ selection via PageRank, and filtered SSMN with K-most-similar-neighbor damped pruning — using parameters matched to the original study (Supporting Information Note S7). Run GNPS FBMN on the same data with the paper's processing parameters. Compute the paper's two comparison indicators — number of probable exclusive features (clean consensus spectra/feature counts within m/z and retention-time tolerances, excluding isotopes and blank m/z) and number of duplicated features/fragmented clusters — plus filtered SSMN node counts, single-node rate, duplicated consensus spectra rate, and library annotation rate for both tools. Assemble all metrics into a single CSV table for direct reproduction of reported values.

## Related tools

- **NP3 MS Workflow** (Executes steps 2–10 (preprocessing, clustering, quantification, cleaning, library identification, IVAMN [M+H]+ selection, filtered SSMN) producing the network under evaluation) — https://github.com/danielatrivella/NP3_MS_Workflow
- **GNPS (Global Natural Products Social Molecular Networking)** (Provides the FBMN reference workflow run on the same data set with the paper's parameters for comparison)
- **Tremolo** (Performs Library Identification (step 6), searching clean consensus spectra against the In Silico MS2 DataBase (ISDB))
- **Cytoscape** (Graph visualization tool for opening and inspecting the resulting SSMN)
- **Mascot Generic Format (MGF)** (Per-sample MS2 file format produced by preprocessing and used for spectral clustering)

## Evaluation signals

- Filtered SSMN node count ratio NP3 vs GNPS FBMN must equal ~1.9 (1413 vs 765 nodes) on MSV000080502 with paper-equivalent parameters
- Single-node rate must be 35.32% for NP3 and 44.71% for FBMN; duplicated consensus spectra rate 1.5% vs 0.4%
- Library annotation rate must be 5.1% (NP3) vs 5.4% (FBMN); fraction of features exclusive to NP3 must be 51.2%
- Every metric in the output CSV must match the article values (and Table S1) within numerical round-off
- Preprocessing must emit one MGF-format MS2 file per sample before clustering, as required by the workflow

## Limitations

- Comparability requires paper-equivalent parameters; the article used the parameters of the original GNPS phorbol-ester study (Supporting Information, Note S7), and divergent settings change node counts and rates
- Tool version changes alter behavior: v1.5.0 changed noise_cutoff to an absolute minimum base peak intensity and removed bed-sample m/z from [M+H]+ networks by default; v1.4.0 added offline GNPS2 library search requiring a re-run of `setup`
- The GNPS2 library annotations bundled with v1.4.0+ were enriched on a fixed date, so library annotation rates may differ from runs against a live GNPS library

## Evidence
