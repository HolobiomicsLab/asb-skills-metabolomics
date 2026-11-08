---
name: multi-tool-annotation-comparison
description: Use when two or more annotation tools (e.g. mWISE and xMSannotator) must
  be compared on identical LC-MS peak lists with known compound ground truth, to rank
  tools by sensitivity, F1, and structural closeness of proposals.
when_to_use_negative:
- If the datasets have no known compound annotations to serve as ground truth — sensitivity
  and Tanimoto-vs-original metrics cannot be computed.
- If you only need to annotate peaks once (no second tool to compare against); use
  the annotation skill directly rather than a comparison pipeline.
- If mWISE's KEGG-only database coverage is insufficient for the compounds of interest
  — the article notes this is a limitation relative to xMSannotator's KEGG, HMDB,
  Lipid Maps, and T3DB support.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mWISE
  role: Annotation tool under evaluation; provides four configurations (FELLA/RClass
    networks × probability-z / binary-raw score settings) yielding top-three KEGG
    compound candidates per peak via diffusion prioritization.
  repo: https://github.com/b2slab/mWISE
- name: xMSannotator
  role: State-of-the-art baseline annotation tool compared against mWISE, run on the
    same filtered peak lists in both ionization modes.
- name: FELLA
  role: R package supplying the KEGG biological network used in one of the four mWISE
    diffusion configurations benchmarked.
- name: DiffuStats
  role: R package performing the label-vector diffusion that produces mWISE's prioritized
    candidate scores in the benchmarked configurations.
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:291071484afe3043652868f79cae8dd26d56f7bb74b4894d962281baa6850aee
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  repo_url: https://github.com/b2slab/mWISE
  tools:
  - Diffu Stats
  - DiffuStats
  - FELLA
  - Genomic Ranges
  - R
  - mWISE
  - xMSannotator
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# multi-tool-annotation-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark LC-MS feature annotation algorithms (mWISE vs xMSannotator, and optionally mummichog) side by side on the same filtered peak lists, computing sensitivity and specificity of top candidate lists plus Tanimoto structural similarity of proposed compounds against known references.

## When to use

Use when two or more annotation tools (e.g. mWISE and xMSannotator) must be compared on identical LC-MS peak lists with known compound ground truth, to rank tools by sensitivity, F1, and structural closeness of proposals.

## When NOT to use

- If the datasets have no known compound annotations to serve as ground truth — sensitivity and Tanimoto-vs-original metrics cannot be computed.
- If you only need to annotate peaks once (no second tool to compare against); use the annotation skill directly rather than a comparison pipeline.
- If mWISE's KEGG-only database coverage is insufficient for the compounds of interest — the article notes this is a limitation relative to xMSannotator's KEGG, HMDB, Lipid Maps, and T3DB support.

## Inputs

- LC-MS peak/feature lists (positive and negative acquisition mode) from public benchmark studies
- Known reference compound identifiers (e.g. KEGG compound IDs) per peak as ground truth
- InChIKey structure representations for original and proposed compounds
- Annotation tool outputs: mWISE top-three candidate lists (four diffusion configurations) and xMSannotator proposals

## Outputs

- Sensitivity/specificity/F1 table per dataset and annotation tool configuration, with mean (standard deviation) aggregated across datasets
- Tanimoto similarity comparison table (per-peak values and aggregated statistics) e.g. tanimoto_similarity_comparison.csv
- Computation-time comparison between annotation tools

## How to apply

First apply the identical input peak-list filtering to every tool: remove LC-MS features without signal and apply the 80% rule (drop peaks with intensity 0 in more than 20% of samples) so the comparison is not confounded by preprocessing differences. Run each annotation tool on the same filtered peak lists across all studies and ionization modes — for mWISE select the top three candidates per peak as the prioritized proposal (test all four configurations: FELLA and RClass networks, each with probability input/z-normalized scores and binary input/raw scores with the unique-annotation option), and run xMSannotator in parallel with its KEGG/HMDB/LipidMaps/T3DB databases. Score each tool's top candidates against the known reference compounds to compute per-dataset sensitivity, specificity, and F1, then aggregate mean and standard deviation across datasets. Additionally compute Tanimoto chemical-structure similarity between original and proposed compounds using their InChIKey representations, and record computation time, since the article found all four mWISE configurations outperform xMSannotator in both performance and computation time (mWISE sensitivity mean (SD) of 0.63 (0.07) vs xMSannotator's 0.55 (0.19)).

## Related tools

- **mWISE** (Annotation tool under evaluation; provides four configurations (FELLA/RClass networks × probability-z / binary-raw score settings) yielding top-three KEGG compound candidates per peak via diffusion prioritization.) — https://github.com/b2slab/mWISE
- **xMSannotator** (State-of-the-art baseline annotation tool compared against mWISE, run on the same filtered peak lists in both ionization modes.)
- **FELLA** (R package supplying the KEGG biological network used in one of the four mWISE diffusion configurations benchmarked.)
- **DiffuStats** (R package performing the label-vector diffusion that produces mWISE's prioritized candidate scores in the benchmarked configurations.)

## Evaluation signals

- mWISE's FELLA/raw-scores configuration should reproduce a sensitivity mean (standard deviation) of 0.63 (0.07) across the six public datasets, while xMSannotator should give 0.55 (0.19).
- Cluster-based filtering should substantially reduce false positives before diffusion (e.g. Assay 2 FP dropped from 17598 to 1671 with specificity 0.91).
- Tanimoto hit ratios of mWISE proposals should lie around 0.59–0.76 across assays (e.g. 0.69 for Assay 1 with FELLA raw score), exceeding xMSannotator's similarity to original compounds.
- All four mWISE configurations should show better F1 and lower computation time than xMSannotator; best mWISE F1 (0.49, Assay 6) and xMSannotator F1 (0.22, Assay 6) bound the expected range.
- Per-dataset metrics must be computed identically for both tools on the same filtered peak lists (80% rule applied) to make the comparison fair.

## Limitations

- mWISE only supports annotation against the KEGG database, an important limitation relative to xMSannotator which also supports HMDB, Lipid Maps, and T3DB.
- Results depend on the availability of known compound references for scoring; performance numbers are only meaningful on datasets with ground-truth annotations.
- The benchmark evaluates the top three candidates per peak, not full annotation lists, so conclusions may not generalize to other candidate cutoffs.

## Evidence
