---
name: network-based-annotation-propagation
description: Use when you have MS1/MS2 untargeted metabolomics feature tables plus
  a set of known/seed metabolite annotations, and need to extend annotations to unknown
  features via network propagation rather than spectral-library matching alone.
when_to_use_negative:
- When you have no seed/known metabolite annotations at all — propagation starts from
  seeds with chemical standards, so an empty seed set yields no propagation
- When the analysis goal is spectral-library search or FBMN/IIMN molecular networking
  within GNPS rather than extending annotations beyond library matches via network
  topology
- When strict FDR control on the reaction network itself is required — the paper explicitly
  refrains from reporting an FDR for the curated MRN due to lack of clearly defined
  non-RPs
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetDNA3
  role: 'Core implementation: two-layer interactive networking topology and recursive
    annotation propagation (via the MrnAnnoAlgo3 R module)'
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: MetDNA2
  role: Single-layer knowledge-guided baseline comparator for measuring coverage/correct-rate
    improvement of the two-layer propagation
  repo: https://github.com/ZhuMetLab/MetDNA2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:2949508daf6fd20b3ca496fcc0468eb28837621e6d86e60b497eccc4ce530ba1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  repo_url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  tools:
  - MetDNA2
  - MetDNA3
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# network-based-annotation-propagation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recursively propagates metabolite annotations across a two-layer knowledge/data network (knowledge-driven metabolic reaction network + data-driven MS2 similarity network) using MetDNA3's MrnAnnoAlgo3, to annotate unknown features from seed metabolites with chemical standards and benchmark coverage/correct-rate against MetDNA2.

## When to use

Use when you have MS1/MS2 untargeted metabolomics feature tables plus a set of known/seed metabolite annotations, and need to extend annotations to unknown features via network propagation rather than spectral-library matching alone.

## When NOT to use

- When you have no seed/known metabolite annotations at all — propagation starts from seeds with chemical standards, so an empty seed set yields no propagation
- When the analysis goal is spectral-library search or FBMN/IIMN molecular networking within GNPS rather than extending annotations beyond library matches via network topology
- When strict FDR control on the reaction network itself is required — the paper explicitly refrains from reporting an FDR for the curated MRN due to lack of clearly defined non-RPs

## Inputs

- MS1 peak table (.csv with 'name', 'mz', 'rt' columns)
- MS2 data files (.mgf or .msp format)
- Sample information table (.csv with 'sample.name' and 'group' columns)
- Seed metabolite annotations (e.g., Supplementary Data 2 seed/validation metabolite annotations; >1600 seed metabolites with chemical standards)
- Curated metabolic reaction network (MRN) object with GNN-predicted reaction relationships

## Outputs

- Annotated feature-metabolite result table with confidence levels (assignConfidenceInMrn3 output) and redundancy removed
- Per-fold and aggregated cross-validation metrics (coverage, Top 1/3/10 correct rates, Top 3 FDR) written to cv_annotation_benchmark.csv

## How to apply

First pre-map experimental features onto the knowledge-based metabolic reaction network via sequential MS1 m/z matching, reaction relationship mapping, and MS2 similarity constraints to build the two-layer topology. Then run recursive annotation propagation with MrnAnnoAlgo3's getMetAnno(table_anno, idx_seeds, round, obj_mrn, table_ms2_network_filter, step_mrn, topN_neighbor, lib_type_mrn, thread), propagating annotations from seed metabolites (e.g., >1600 standards) across cross-network interactions. Post-process each result table with assignConfidenceInMrn3(result_df) to assign confidence levels and rmRedunInMrn3(result_df) to remove redundant annotations. For benchmarking, split known annotations into 10 cross-validation folds (30% seeds / 70% validation targets), compute per-fold annotation coverage on validation metabolites, and aggregate Top 1/3/10 correct rates and Top 3 FDR. Compare against the MetDNA2 single-layer baseline to quantify the coverage/accuracy gain (39.9%→68.1% coverage; 65.6%→84.4% Top 3 correct rate).

## Related tools

- **MetDNA3** (Core implementation: two-layer interactive networking topology and recursive annotation propagation (via the MrnAnnoAlgo3 R module)) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **MetDNA2** (Single-layer knowledge-guided baseline comparator for measuring coverage/correct-rate improvement of the two-layer propagation) — https://github.com/ZhuMetLab/MetDNA2

## Examples

```
devtools::install_github("ZhuMetLab/MrnAnnoAlgo3"); anno <- getMetAnno(table_anno, idx_seeds, round, obj_mrn, table_ms2_network_filter, step_mrn, topN_neighbor, lib_type_mrn, thread)
```

## Evaluation signals

- Aggregated 10-fold cross-validation annotation coverage reaches ~68.1% for MetDNA3 vs ~39.9% MetDNA2 baseline on the 20 Orbitrap datasets
- Aggregated Top 1/3/10 correct rates land at ~68.0%/84.4%/91.0% and Top 3 FDR at ~15.6%, matching Source Data values for Fig. 3e–g within tolerance
- cv_annotation_benchmark.csv contains one row per fold plus aggregate rows, with fold splits at 30% seeds / 70% validation targets
- Result table passes assignConfidenceInMrn3 and rmRedunInMrn3 without redundant (multiple candidates per feature) annotations remaining
- Output annotation table preserves feature-metabolite pairing schema comparable to MetDNA2's table1_identification.csv / table3_identification_pair.csv

## Limitations

- No FDR can be reported for the curated MRN because clearly defined non-RPs (negative examples) are lacking under the screening conditions
- GNN-based direct prediction of reaction pairs may include a considerable number of false positives, mitigated only by a two-step pre-screening strategy
- MS2 spectra often cannot distinguish isomeric metabolites, so both network propagation and MS2-matching-based validation may yield optimistic FDR estimates; cross-tool concordance may also overestimate performance without chemical standards

## Evidence
