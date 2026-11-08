---
name: untargeted-metabolomics-feature-table-processing
description: Use when an untargeted metabolomics MS1 feature table (.csv with name/mz/rt
  columns) must be linked onto a curated metabolic reaction network before annotation
  propagation.
when_to_use_negative:
- You only have raw LC-MS/MS files and no feature table yet — peak picking with XCMS,
  MS-DIAL, or MZmine2 must first produce the name/mz/rt MS1 peak table.
- No curated MRN exists yet — this step consumes the igraph MRN produced by the prior
  GNN-based reaction-network construction task.
- The goal is MS2 spectral-similarity networking (e.g., GNPS FBMN/IIMN) rather than
  MS1-constrained feature-to-metabolite mapping onto a reaction network.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3822
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3272
tools:
- name: MrnAnnoAlgo3
  role: R package supplying getMs1Data4Mrn3, getTableSeedLong4Mrn3, predictRtInMrn3,
    and getErrorAndScore for loading data, training the RT model, and scoring MS1
    feature-to-metabolite matches
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: MetDNA3
  role: Two-layer annotation pipeline that consumes the MS1-constrained MRN; completed
    functions are served via the MetDNA3 webserver
  repo: http://metdna.zhulab.cn/
- name: igraph
  role: Loads the curated MRN and subsets its nodes/edges to build and save the MS1-constrained
    network as .rds
- name: randomForest
  role: Fits the column-type-specific retention-time prediction model over seed metabolite
    annotations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:8ca4449f7f2a658643666adc489f0efdcd0c4d27579c3ea61320ca3ddcf88bde
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
  - randomForest
  - readr
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3822
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# untargeted-metabolomics-feature-table-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-maps experimental MS1 feature tables from untargeted metabolomics onto the knowledge-based metabolic reaction network (MRN) using 15 ppm m/z matching plus 30% relative-RT matching, pruning redundant nodes/edges to produce the MS1-constrained MRN that anchors MetDNA3's two-layer annotation topology.

## When to use

Use when an untargeted metabolomics MS1 feature table (.csv with name/mz/rt columns) must be linked onto a curated metabolic reaction network before annotation propagation.

## When NOT to use

- You only have raw LC-MS/MS files and no feature table yet — peak picking with XCMS, MS-DIAL, or MZmine2 must first produce the name/mz/rt MS1 peak table.
- No curated MRN exists yet — this step consumes the igraph MRN produced by the prior GNN-based reaction-network construction task.
- The goal is MS2 spectral-similarity networking (e.g., GNPS FBMN/IIMN) rather than MS1-constrained feature-to-metabolite mapping onto a reaction network.

## Inputs

- Curated metabolic reaction network (MRN) from the GNN-based curation step, as an igraph graph object
- Experimental MS1 feature/peak table (.csv; first three columns 'name', 'mz', 'rt', loaded via getMs1Data4Mrn3)
- Seed annotation table of seed metabolites with chemical standards, long format (loaded via getTableSeedLong4Mrn3)
- LC column type (e.g., hilic) selecting the column-specific RT prediction model (argument 'column')

## Outputs

- ms1_constrained_mrn.rds — igraph graph object of the MS1-constrained MRN with reduced nodes/edges
- Metabolite-feature cross-links with m/z (ppm) and relative-RT error validity flags and scores from getErrorAndScore
- Predicted retention times for all MRN metabolites from the fitted random-forest RT model

## How to apply

Load the curated MRN as an igraph graph object and import the experimental MS1 feature table and seed annotation table via getMs1Data4Mrn3 and getTableSeedLong4Mrn3 from MrnAnnoAlgo3 (installed with devtools::install_github('ZhuMetLab/MrnAnnoAlgo3')). Train the column-type-specific random-forest retention-time model on the seed metabolite annotations with predictRtInMrn3(wd, ms1_data, table_seed_long, md_mrn, column), then predict RTs for every metabolite in the MRN. Match each experimental feature to candidate metabolites accepting MS1 m/z differences within 15 ppm and relative RT differences within 30%, computing validity flags and scores with getErrorAndScore(table_anno, is_rt_valid, is_ccs_valid, mz_ppm_thr, mz_tol, rt_tol_rel, ccs_tol_rel). Assemble the accepted metabolite-feature cross-links, subset the MRN with igraph to drop unmatched (redundant) nodes and edges, and save the result as ms1_constrained_mrn.rds. The rationale is that this MS1-constrained knowledge layer, joined to the data-driven MS2 similarity layer, enables recursive annotation propagation with >10-fold computational efficiency; matches at this stage are candidates that later reaction-relationship mapping and MS2 similarity constraints refine.

## Related tools

- **MrnAnnoAlgo3** (R package supplying getMs1Data4Mrn3, getTableSeedLong4Mrn3, predictRtInMrn3, and getErrorAndScore for loading data, training the RT model, and scoring MS1 feature-to-metabolite matches) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **MetDNA3** (Two-layer annotation pipeline that consumes the MS1-constrained MRN; completed functions are served via the MetDNA3 webserver) — http://metdna.zhulab.cn/
- **igraph** (Loads the curated MRN and subsets its nodes/edges to build and save the MS1-constrained network as .rds)
- **randomForest** (Fits the column-type-specific retention-time prediction model over seed metabolite annotations)

## Examples

```
library(MrnAnnoAlgo3); predictRtInMrn3(wd, ms1_data, table_seed_long, md_mrn, column)
```

## Evaluation signals

- ms1_constrained_mrn.rds exists in the working directory and readRDS() returns a valid igraph graph object.
- Every retained metabolite-feature cross-link satisfies both tolerances: m/z error ≤ 15 ppm AND relative RT error ≤ 30%.
- Node and edge counts of the MS1-constrained MRN are smaller than the input curated MRN (redundant nodes/edges removed by the constraint).
- The random-forest RT model was trained per LC column type on seed annotations and produced RT predictions for all MRN metabolites.
- Match records carry validity flags and scores computed by getErrorAndScore (mz_ppm_thr, rt_tol_rel evaluated per link).

## Limitations

- RT prediction is column-type-specific and relies on seed metabolites annotated with chemical standards; the 30% relative-RT gate is only as good as that model.
- MS1 m/z + RT pre-mapping cannot resolve isomeric metabolites — the paper notes isomeric metabolites may be indistinguishable even by MS2, so links here are candidates, not final annotations.
- The underlying MRN has no reported FDR and GNN-predicted reaction pairs may include a considerable number of false positives; MS1 constraining prunes but does not eliminate false edges.
- MrnAnnoAlgo3 provides only the core algorithm module; full MetDNA3 functionality requires additional modules available via the webserver.

## Evidence

- [readme] A MS1 peak table (.csv format, **required**). The first three columns must be "name" , "mz" , and "rt".: [evidence span withheld — anchor status: external_doc]
