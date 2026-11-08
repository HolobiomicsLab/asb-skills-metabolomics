---
name: ms1-feature-metabolite-mass-matching
description: 'Use when starting MetDNA3/MrnAnnoAlgo3 annotation: you have a curated
  MRN (igraph), an MS1 feature table, and seed metabolite annotations, and must constrain
  the network before reaction mapping and MS2-similarity propagation.'
when_to_use_negative:
- The MRN has not yet been curated by the GNN-based reaction-prediction step (task_001)
  — there is no knowledge network to match features onto.
- You are running MetDNA2's legacy pipeline, whose RT libraries (lib_rt, lib_ccs,
  zhuMetLib) were removed from the open-source package and would require repackaging
  your own libraries.
- Data are MS2-only or features lack precursor m/z and RT information (e.g. already-collapsed
  peak-group tables) — MS1 matching is the wrong entry point.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MrnAnnoAlgo3 (MetDNA3 core)
  role: Provides getMs1Data4Mrn3, getTableSeedLong4Mrn3, predictRtInMrn3 and getErrorAndScore
    used for MS1 m/z + RT matching of features to MRN metabolites
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: igraph
  role: Graph object representation and subsetting of the MRN to form the MS1-constrained
    network
- name: MetDNA2
  role: Predecessor package; documents the MS1 peak-table input format ('name', 'mz',
    'rt') and randomForest dependency reused in this matching step
  repo: https://github.com/ZhuMetLab/MetDNA2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:14ba944ce02b916404a76075250bc15cde7cce21010a0389b909ac9e0234242e
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
  - MrnAnnoAlgo3
  - MrnAnnoAlgo3 (MetDNA3 core)
  - dplyr
  - getErrorAndScore
  - igraph
  - predictRtInMrn3
  - readr
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# MS1 feature-metabolite mass matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-maps experimental MS1 features onto a knowledge-based metabolic reaction network (MRN) by matching precursor m/z within 15 ppm and relative retention-time differences within 30%, producing an MS1-constrained MRN with cross-layer metabolite–feature links as the first step of MetDNA3's two-layer topology.

## When to use

Use when starting MetDNA3/MrnAnnoAlgo3 annotation: you have a curated MRN (igraph), an MS1 feature table, and seed metabolite annotations, and must constrain the network before reaction mapping and MS2-similarity propagation.

## When NOT to use

- The MRN has not yet been curated by the GNN-based reaction-prediction step (task_001) — there is no knowledge network to match features onto.
- You are running MetDNA2's legacy pipeline, whose RT libraries (lib_rt, lib_ccs, zhuMetLib) were removed from the open-source package and would require repackaging your own libraries.
- Data are MS2-only or features lack precursor m/z and RT information (e.g. already-collapsed peak-group tables) — MS1 matching is the wrong entry point.

## Inputs

- Curated metabolic reaction network (MRN) as an igraph graph object (.rds) from the prior network-construction task
- Experimental MS1 feature/peak table (.csv, e.g. XCMS/MS-DIAL/MZmine output with 'name', 'mz', 'rt' columns)
- Seed metabolite annotation table (long-format, from chemical-standard-confirmed seeds)
- MRN metadata object (md_mrn) and column specification (e.g. 'hilic')

## Outputs

- MS1-constrained MRN saved as ms1_constrained_mrn.rds (igrapgh-subset network with only matched metabolites/edges retained)
- Metabolite–feature cross-link table satisfying both 15 ppm m/z and 30% relative RT tolerances
- Predicted retention times for all MRN metabolites from the fitted random-forest RT model

## How to apply

Load the curated MRN as an igraph graph object together with the experimental MS1 feature table and seed annotation table via getMs1Data4Mrn3 and getTableSeedLong4Mrn3 from MrnAnnoAlgo3. Train the column-type-specific retention-time model (random-forest, column = e.g. hilic) from the seed metabolite annotations with predictRtInMrn3, then predict RTs for all MRN metabolites. Match each experimental feature to candidate metabolites using MS1 m/z differences within 15 ppm plus relative RT differences within 30%, computing match validity and scores with getErrorAndScore (which also accepts CCS tolerance via ccs_tol_rel). Assemble the accepted metabolite–feature cross-links and subset the MRN with igraph to form the MS1-constrained network, saving it as ms1_constrained_mrn.rds. The rationale is that MS1-only pre-mapping reduces redundant nodes and edges and establishes the cross-network links later exploited by the two-layer, recursively propagated annotation.

## Related tools

- **MrnAnnoAlgo3 (MetDNA3 core)** (Provides getMs1Data4Mrn3, getTableSeedLong4Mrn3, predictRtInMrn3 and getErrorAndScore used for MS1 m/z + RT matching of features to MRN metabolites) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **igraph** (Graph object representation and subsetting of the MRN to form the MS1-constrained network)
- **MetDNA2** (Predecessor package; documents the MS1 peak-table input format ('name', 'mz', 'rt') and randomForest dependency reused in this matching step) — https://github.com/ZhuMetLab/MetDNA2

## Examples

```
rt_model <- predictRtInMrn3(wd = wd, ms1_data = getMs1Data4Mrn3(wd), table_seed_long = getTableSeedLong4Mrn3(wd), md_mrn = md_mrn, column = "hilic")
```

## Evaluation signals

- ms1_constrained_mrn.rds exists, is readable, and parses as an igraph graph object.
- Every retained metabolite–feature link satisfies both the 15 ppm m/z tolerance and the 30% relative RT tolerance.
- No retained node/edge pair falls outside the 15 ppm + 30% RT acceptance window returned by getErrorAndScore.
- RT predictions for MRN metabolites come from a random-forest model trained only on the seed metabolite annotations for the specified LC column type.
- The constrained network has strictly fewer redundant nodes and edges than the input MRN (redundancy reduction is the stated purpose of pre-mapping).

## Limitations

- Retention-time prediction depends on seed annotations for the specific column and LC method; MetDNA2 notes RT library objects (lib_rt, lib_ccs) are removed for copyright reasons, so users must supply their own libraries when adapting the pipeline.
- MS1 m/z matching alone cannot distinguish isomeric metabolites; isomer resolution is deferred to downstream MS2-similarity constraints, so candidate lists at this stage may be redundant.
- The upstream MRN itself may contain false positives from GNN-based reaction prediction, and no FDR is reported for the curated MRN due to lack of clearly defined non-RPs.

## Evidence
