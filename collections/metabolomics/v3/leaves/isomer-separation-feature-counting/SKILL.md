---
name: isomer-separation-feature-counting
description: Use when an LC–MS/MS data set contains multiple isomeric or co-eluting
  compounds at the same m/z and you need to quantify how many distinct features survive
  processing, or compare feature/node consolidation between NP3 and GNPS FBMN.
when_to_use_negative:
- The input is already a processed feature/quantification table with no MS2 spectra
  — isomer re-counting requires the raw LC–MS/MS and NP3's Clustering/Cleaning steps
- The goal is a single consolidated molecular network with minimal node count rather
  than isomer-resolved feature enumeration (GNPS FBMN's 0.4% duplication rate indicates
  stronger consolidation by design)
- Only MS1 data are available; the NP3-shifted cosine cleaning and isomer discrimination
  rely on MS2 spectra in MGF files
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: NP3 MS Workflow
  role: Executes Preprocessing through filtered SSMN (steps 2–10), including NP3-shifted-cosine
    Cleaning that keeps isomers at the same m/z as separate clean consensus spectra
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Tremolo
  role: 'Library Identification (step 6): searches clean consensus spectra against
    ISDB so isomers can be annotated'
- name: GNPS (Global Natural Products Social Molecular Networking)
  role: Baseline comparator run via FBMN on the same data set to measure consolidation
    differences (nodes, duplicated consensus spectra)
- name: Cytoscape
  role: Graph visualization of the filtered SSMN used for node inspection and counting
- name: Mascot Generic Format (MGF)
  role: MS2 file format in which per-sample spectra are exported for clustering and
    library search
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:f8d14d11bcfe29259b1295efdf93a0c20e6e2e45d2de386dcdccab15e7d3c6a9
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
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# Isomer separation feature counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Counts and validates distinct isomeric features at a shared m/z in LC–MS/MS untargeted metabolomics, exploiting NP3 MS Workflow's spectral cleaning (NP3-shifted cosine) and filtered SSMN to avoid the over-consolidation of co-eluting isomers seen in GNPS FBMN.

## When to use

Use when an LC–MS/MS data set contains multiple isomeric or co-eluting compounds at the same m/z and you need to quantify how many distinct features survive processing, or compare feature/node consolidation between NP3 and GNPS FBMN.

## When NOT to use

- The input is already a processed feature/quantification table with no MS2 spectra — isomer re-counting requires the raw LC–MS/MS and NP3's Clustering/Cleaning steps
- The goal is a single consolidated molecular network with minimal node count rather than isomer-resolved feature enumeration (GNPS FBMN's 0.4% duplication rate indicates stronger consolidation by design)
- Only MS1 data are available; the NP3-shifted cosine cleaning and isomer discrimination rely on MS2 spectra in MGF files

## Inputs

- Raw LC–MS/MS data set from untargeted metabolomics (e.g. MassIVE MSV000080502, Euphorbia dendroides plant extract, 16 samples)
- Metadata Table describing LC–MS/MS samples and sample groups (NP3 step 1 input)
- MGF-format MS2 spectra files produced by NP3 Preprocessing
- Reference spectral library (GNPS/ISDB, e.g. ALL_GNPS_NO_PROPOGATED.mgf split by ion mode)
- Blank and/or culture-media samples for exclusion filtering

## Outputs

- Clean consensus spectra table with per-feature m/z, retention time, and quantification across samples
- Filtered SSMN (K-most-similar-neighbor, damped-pruned) viewable in Cytoscape
- Per-m/z isomer counts (e.g. 11 distinct isomers of m/z 589.31) with library annotations
- Comparison metrics table (CSV): filtered node counts, single-node rate, duplicated consensus spectra rate, library annotation rate, exclusive-feature fraction vs GNPS FBMN

## How to apply

Run NP3 MS Workflow steps 2–10 on the raw LC–MS/MS data (e.g. MSV000080502 from Euphorbia dendroides): Preprocessing detects LC–MS features (MS1 peaks) and extracts MS2 spectra to Mascot Generic Format (MGF); Clustering/Quantification/Cleaning (steps 3–5) group MS2 by spectral similarity using the NP3-shifted cosine function, a greedy version of the modified cosine, so co-eluting isomers at one m/z remain as separate clean consensus spectra rather than being merged. Library Identification (step 6) searches clean consensus spectra against the In Silico MS2 DataBase (ISDB) via Tremolo, and IVAMN with PageRank selects [M + H]+ representative ions before the filtered SSMN applies link removal, K-most-similar-neighbor, and damped pruning. Count distinct isomers per m/z (the paper demonstrated 11 distinct isomers of m/z 589.31, all annotated with the GNPS spectral database), the filtered node count, and the duplicated consensus spectra rate, then benchmark against GNPS FBMN run on the same data with the same parameters — the paper's benchmarks were 1413 vs 765 nodes (1.9x), duplicated consensus spectra 1.5% vs 0.4%, and 51.2% NP3-exclusive features.

## Related tools

- **NP3 MS Workflow** (Executes Preprocessing through filtered SSMN (steps 2–10), including NP3-shifted-cosine Cleaning that keeps isomers at the same m/z as separate clean consensus spectra) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Tremolo** (Library Identification (step 6): searches clean consensus spectra against ISDB so isomers can be annotated)
- **GNPS (Global Natural Products Social Molecular Networking)** (Baseline comparator run via FBMN on the same data set to measure consolidation differences (nodes, duplicated consensus spectra))
- **Cytoscape** (Graph visualization of the filtered SSMN used for node inspection and counting)
- **Mascot Generic Format (MGF)** (MS2 file format in which per-sample spectra are exported for clustering and library search)

## Evaluation signals

- Number of distinct isomers at a known m/z matches expectation (the paper recovered all 11 distinct isomers of m/z 589.31 with GNPS spectral annotations)
- Duplicated consensus spectra rate falls near the paper's NP3 benchmark (~1.5%) rather than FBMN's ~0.4%, confirming isomers were not over-consolidated
- Filtered SSMN node count for the Euphorbia dendroides data set is ~1413 vs 765 for FBMN (1.9x), and ~51.2% of features are NP3-exclusive within m/z and retention-time tolerances
- Each computed metric in the comparison CSV matches the article values (Table S1) within numerical round-off
- Cleaning verifiably used the NP3-shifted cosine (greedy modified cosine) rather than plain cosine, and the filtered SSMN respected minimum-matched-peaks link removal with damped K-most-similar-neighbor pruning

## Limitations

- Duplicated consensus spectra are expected in NP3 by design (sensitivity to minority compounds); the trade-off is more single nodes (35.32% vs FBMN 44.71%) and slightly lower library annotation rate (5.1% vs 5.4%)
- Version-dependent behavior: in v1.5.1 a bug in join_jobs Step 7 caused filtered IVAMN index mismatches when noise cutoff was applied, and the noise_cutoff parameter changed to an absolute minimum base peak intensity in v1.5.0 — older runs may not reproduce isomer counts
- setup must be re-executed after library or ion-mode changes (e.g. v1.4.0/v1.6.0 splitting ALL_GNPS MGF by ion mode), and GNPS2 library annotations in the repo were enriched at a fixed date, so newly added library spectra are not visible offline
- If maximum clustering rounds are reached in clean step 5, some spectra removal is skipped (fixed in v1.5.0), which can inflate feature counts

## Evidence
