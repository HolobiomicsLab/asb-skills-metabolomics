---
name: network-topology-validation
description: Use when building or filtering MetDNA3 feature/metadata reaction networks
  and you must verify intermediate graph sizes (node/edge counts) against published
  reference values before recursive annotation propagation.
when_to_use_negative:
- The analysis is not following MetDNA3's two-layer pre-mapping workflow — published
  reference counts (e.g. 2,993/55,674 for NIST urine HILIC-MS(+)) are dataset- and
  method-specific and do not validate other pipelines.
- You are only running the final annotation output (table1_identification.csv) and
  do not need to audit intermediate network topology — use downstream annotation-quality
  validation instead.
- Input is a single-network (knowledge-only) result from a tool like NetID or FBMN
  where no data-constrained MRN intermediate exists to compare against.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: MetDNA3
  role: Provides the two-layer networking pipeline (data and knowledge pre-mapping,
    recursive annotation propagation) whose intermediate network topologies are being
    validated.
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: igraph
  role: R graph library used to load the filtered edge table and compute node/edge
    counts of the data-constrained MRN.
- name: SpectraTools
  role: Used upstream to compute modified dot-product MS2 spectral similarity, whose
    ≥ 0.5 cutoff produces the edge table whose topology is validated here.
  repo: https://github.com/ZhuMetLab/SpectraTools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:d8a074aafdb3d5875a3386b01bebd016668bacef16958c2c0886f572dd09d206
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
  - SpectraTools
  - igraph
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# network-topology-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validate network topology counts at each stage of MetDNA3's two-layer annotation pipeline, e.g. checking that the data-constrained MRN for NIST urine HILIC-MS(+) contains exactly 2,993 metabolites and 55,674 reaction pairs before annotation propagation proceeds.

## When to use

Use when building or filtering MetDNA3 feature/metadata reaction networks and you must verify intermediate graph sizes (node/edge counts) against published reference values before recursive annotation propagation.

## When NOT to use

- The analysis is not following MetDNA3's two-layer pre-mapping workflow — published reference counts (e.g. 2,993/55,674 for NIST urine HILIC-MS(+)) are dataset- and method-specific and do not validate other pipelines.
- You are only running the final annotation output (table1_identification.csv) and do not need to audit intermediate network topology — use downstream annotation-quality validation instead.
- Input is a single-network (knowledge-only) result from a tool like NetID or FBMN where no data-constrained MRN intermediate exists to compare against.

## Inputs

- MS1 peak/feature table (.csv) from data preprocessing (e.g. NIST urine HILIC-MS(+) demo data)
- MS2 spectra files (.mgf or .msp)
- Reaction-linked feature pair table from the GNN-curated knowledge-layer MRN
- Filtered edge table at intermediate pipeline stage (e.g. after MS2 similarity ≥ 0.5 filtering)
- Seed annotation table (seed metabolites with chemical standards)

## Outputs

- Node and edge counts for the current network layer (e.g. metabolite count and reaction-pair count)
- Comparison against expected reference counts from the article's Source Data (e.g. 2,993 metabolites / 55,674 pairs for NIST urine HILIC-MS(+) data-constrained MRN)
- Pass/fail signal indicating whether the network topology matches the published topology before annotation propagation

## How to apply

After each network-construction or filtering step (MS1 matching, reaction relationship mapping, MS2 similarity filtering), load the resulting edge table (e.g. data_constrained_mrn_edges.csv) and compute node and reaction-pair counts, e.g. via igraph in R. Compare against the article's reference numbers for the same dataset and layer: for NIST urine HILIC-MS(+) the data-constrained MRN must contain exactly 2,993 metabolites and 55,674 reaction pairs, matching the Source Data files (e.g. Fig. 2c–d). The rationale is that the two-layer pre-mapping workflow deliberately refines metabolite–metabolite, feature–feature, and metabolite–feature links; a count mismatch indicates an upstream pipeline step (MS1 matching threshold, MS2 similarity cutoff of 0.5, or reaction-pair retrieval from the GNN-curated knowledge layer) was not applied correctly and would propagate errors into recursive annotation. Run counts at every intermediate stage — knowledge-driven MRN, data-layer feature network, and data-constrained MRN — so any drift is localized to the step that introduced it.

## Related tools

- **MetDNA3** (Provides the two-layer networking pipeline (data and knowledge pre-mapping, recursive annotation propagation) whose intermediate network topologies are being validated.) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **igraph** (R graph library used to load the filtered edge table and compute node/edge counts of the data-constrained MRN.)
- **SpectraTools** (Used upstream to compute modified dot-product MS2 spectral similarity, whose ≥ 0.5 cutoff produces the edge table whose topology is validated here.) — https://github.com/ZhuMetLab/SpectraTools

## Examples

```
library(igraph); g <- graph_from_data_frame(read.csv('data_constrained_mrn_edges.csv'), directed=FALSE); c(vcount(g), ecount(g))  # expect 2993 nodes, 55674 edges for NIST urine HILIC-MS(+)
```

## Evaluation signals

- For NIST urine HILIC-MS(+), the data-constrained MRN edge table must contain exactly 2,993 metabolites and 55,674 reaction pairs, matching the article's Source Data for Fig. 2c–d.
- Counts must be computed identically at each intermediate stage (knowledge-driven MRN, data-layer feature network, data-constrained MRN) so drift is localized to the offending step.
- The edge table should only contain reaction-linked feature pairs that survived MS2 similarity ≥ 0.5 filtering (getMatchParam output feeding getMetAnno as table_ms2_network_filter).
- Node count should equal the number of unique metabolites appearing in the edge table (no orphan nodes if built from the filtered pair list).
- Recompute counts from the persisted edge CSV (data_constrained_mrn_edges.csv) rather than trusting in-memory pipeline state, to catch silent upstream reprocessing.

## Limitations

- The MRN cannot be assigned a false discovery rate because non-RPs (negative examples) are not clearly defined under the screening conditions, and GNN performance metrics are inapplicable to the filtered candidate set.
- MS2 spectra often lack sufficient fragment information and cannot distinguish isomeric metabolites, so even a topologically correct network may propagate annotation errors.
- Published counts are tied to the specific NIST urine HILIC-MS(+) demo dataset; they serve as regression anchors for that configuration, not universal validation targets.
- GNN-based direct prediction of reaction pairs may include a considerable number of false positives in the knowledge layer before data-driven filtering.

## Evidence

- [other] Validation: the exported data-constrained MRN must contain exactly 2,993 metabolites and 55,674 reaction pairs for NIST urine HILIC-MS(+), matching the Source Data file for Fig. 2c–d.: [evidence span withheld — anchor status: not_found]
