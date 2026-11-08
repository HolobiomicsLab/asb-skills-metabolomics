---
name: graph-subsetting-with-igraph
description: Use when a curated metabolic reaction network graph must be reduced to
  metabolites supported by experimental MS1 evidence (e.g. after feature-metabolite
  matching within 15 ppm m/z and 30% relative RT tolerance), before downstream annotation
  propagation.
when_to_use_negative:
- The MRN has not yet been curated (no GNN-filtered reaction network exists to subset).
- The goal is reaction-relationship mapping or MS2 similarity filtering — those are
  subsequent pre-mapping steps, not MS1 m/z/RT-constrained subsetting.
- Data come from MS2-only spectral similarity networking (e.g. GNPS/FBMN) with no
  MS1 feature m/z and RT table.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MrnAnnoAlgo3
  role: Provides getMs1Data4Mrn3, getTableSeedLong4Mrn3, predictRtInMrn3 and getErrorAndScore
    for MS1 matching prior to subsetting
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: igraph
  role: Graph representation and induced-subgraph extraction to build the MS1-constrained
    MRN
- name: MetDNA3
  role: Downstream two-layer annotation workflow that consumes the MS1-constrained
    MRN
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: dplyr
  role: Assembling and filtering the accepted metabolite-feature cross-link table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:298fc30c200a38cd89231544f0da2809062105eefccc4b76c5e2fa29beb47664
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
  - MetDNA3
  - MrnAnnoAlgo3
  - dplyr
  - getErrorAndScore
  - igraph
  - predictRtInMrn3
  - readr
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# Graph subsetting with igraph

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Subsetting a knowledge-based metabolic reaction network (MRN) with igraph to retain only metabolites that received experimental feature cross-links, forming the MS1-constrained MRN used by MetDNA3's two-layer topology.

## When to use

Use when a curated metabolic reaction network graph must be reduced to metabolites supported by experimental MS1 evidence (e.g. after feature-metabolite matching within 15 ppm m/z and 30% relative RT tolerance), before downstream annotation propagation.

## When NOT to use

- The MRN has not yet been curated (no GNN-filtered reaction network exists to subset).
- The goal is reaction-relationship mapping or MS2 similarity filtering — those are subsequent pre-mapping steps, not MS1 m/z/RT-constrained subsetting.
- Data come from MS2-only spectral similarity networking (e.g. GNPS/FBMN) with no MS1 feature m/z and RT table.

## Inputs

- Curated MRN as an igraph graph object (from task_001, e.g. .rds)
- Experimental MS1 feature table (.csv with 'name', 'mz', 'rt' columns)
- Seed metabolite annotation table (long format) for RT model training
- MRN metadata object (md_mrn) and chromatography column type (e.g. hilic)

## Outputs

- ms1_constrained_mrn.rds — an igraph graph containing only metabolites with accepted feature cross-links
- Metabolite-feature cross-link table with match errors and scores

## How to apply

Load the curated MRN produced by GNN-based reaction curation as an igraph graph object, together with the experimental MS1 feature table and seed annotation table via getMs1Data4Mrn3 and getTableSeedLong4Mrn3 from MrnAnnoAlgo3. Fit the column-type-specific random-forest RT model with predictRtInMrn3 (parameterized by column, e.g. hilic) and predict RTs for all MRN metabolites. Match each experimental feature to candidate metabolites using MS1 m/z differences within 15 ppm and relative RT differences within 30%, computing match validity and scores with getErrorAndScore. Assemble the accepted metabolite-feature cross-links, then use igraph subsetting (induced subgraph on matched metabolite nodes) to remove redundant nodes and edges, saving the result as ms1_constrained_mrn.rds. The rationale is that constraining the knowledge network to empirically observed features reduces false-positive reaction links before the two-layer interactive annotation propagation.

## Related tools

- **MrnAnnoAlgo3** (Provides getMs1Data4Mrn3, getTableSeedLong4Mrn3, predictRtInMrn3 and getErrorAndScore for MS1 matching prior to subsetting) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **igraph** (Graph representation and induced-subgraph extraction to build the MS1-constrained MRN)
- **MetDNA3** (Downstream two-layer annotation workflow that consumes the MS1-constrained MRN) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **dplyr** (Assembling and filtering the accepted metabolite-feature cross-link table)

## Examples

```
library(igraph); ms1_mrn <- induced_subgraph(mrn_graph, V(mrn_graph)[name %in% matched_metabolites]); saveRDS(ms1_mrn, "ms1_constrained_mrn.rds")
```

## Evaluation signals

- ms1_constrained_mrn.rds exists and is readable as an igraph graph object.
- Every retained metabolite-feature link satisfies both the 15 ppm m/z tolerance and the 30% relative RT tolerance.
- Node and edge counts of the constrained graph are strictly smaller than (or equal to) the curated MRN, confirming redundancy removal.
- All retained metabolite nodes have at least one cross-network feature link; no isolated metabolite nodes remain.

## Limitations

- The RT prediction model depends on seed metabolite annotations for the specific chromatography column; poor seed coverage degrades the 30% relative-RT filter.
- MS1 m/z matching alone cannot distinguish isomeric metabolites, so the constrained network may retain isomer-ambiguous links.
- CCS validation (is_ccs_valid in getErrorAndScore) is only applicable when ion-mobility data are available.

## Evidence

- [full_text] In the pre-mapping step, experimental features are first matched to metabolites in the MRN based on MS1 m/z matching, which reduces redundant nodes and edges to form the MS1-constrained MRN: [evidence span withheld — anchor status: not_found]
