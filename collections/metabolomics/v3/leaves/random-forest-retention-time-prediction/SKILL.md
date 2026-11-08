---
name: random-forest-retention-time-prediction
description: Use when experimental MS1 features must be matched to MRN metabolites
  with a relative RT tolerance (30%) in addition to m/z, and only a subset of metabolites
  (seed annotations) have observed retention times to learn from.
when_to_use_negative:
- Seed annotations lack reliable observed retention times (e.g., no RT recalibration
  table available), leaving the random-forest model nothing to learn from
- CCS values are available and CCS-based matching (ccs_tol_rel in getErrorAndScore)
  is preferred, or m/z-only matching without RT filtering is the goal
- The task is downstream annotation propagation on the already-constrained network
  rather than building the MS1-constrained MRN
edam_operation: http://edamontology.org/operation_0286
tools:
- name: MrnAnnoAlgo3
  role: Provides predictRtInMrn3 for training the column-type-specific random-forest
    RT model and getErrorAndScore for RT/m/z match scoring during MS1 pre-mapping
  repo: github.com/ZhuMetLab/MrnAnnoAlgo3
- name: randomForest
  role: R package supplying the random-forest algorithm underlying the RT prediction
    model
- name: MetDNA3
  role: Overall workflow in which the RT predictions feed the two-layer interactive
    networking topology
  repo: http://metdna.zhulab.cn/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:9ee9ddab3f34feb8a9e216c6debb90b888912946908679ae5f77e18906b67022
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  tools:
  - MetDNA3
  - MrnAnnoAlgo3
  - dplyr
  - getErrorAndScore
  - predictRtInMrn3
  - randomForest
  - readr
  edam_operation: http://edamontology.org/operation_0286
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Random-forest retention time prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a column-type-specific random-forest model on seed metabolite annotations to predict retention times for every metabolite in the knowledge-based metabolic reaction network (MRN), enabling RT-based filtering of experimental features during MetDNA3 MS1 pre-mapping.

## When to use

Use when experimental MS1 features must be matched to MRN metabolites with a relative RT tolerance (30%) in addition to m/z, and only a subset of metabolites (seed annotations) have observed retention times to learn from.

## When NOT to use

- Seed annotations lack reliable observed retention times (e.g., no RT recalibration table available), leaving the random-forest model nothing to learn from
- CCS values are available and CCS-based matching (ccs_tol_rel in getErrorAndScore) is preferred, or m/z-only matching without RT filtering is the goal
- The task is downstream annotation propagation on the already-constrained network rather than building the MS1-constrained MRN

## Inputs

- Curated metabolic reaction network (MRN) as an igraph graph object (md_mrn)
- Experimental MS1 feature table (e.g., .csv with name, mz, rt columns)
- Seed annotation table (table_seed_long) with metabolites annotated by chemical standards and observed RTs
- Chromatography column type identifier (column, e.g., hilic) and working directory (wd)

## Outputs

- Fitted random-forest RT prediction model (column-type specific)
- Predicted RT values for all metabolites in the MRN
- Per-feature match validity flags and scores (is_rt_valid, getErrorAndScore outputs) used to assemble ms1_constrained_mrn.rds

## How to apply

Load the curated MRN (igraph object, md_mrn), the experimental MS1 feature table, and the seed annotation table via getMs1Data4Mrn3 and getTableSeedLong4Mrn3 in MrnAnnoAlgo3. Call predictRtInMrn3(wd, ms1_data, table_seed_long, md_mrn, column) to fit a random-forest model whose features are specific to the chromatography column type (e.g., HILIC), then predict RTs for all MRN metabolites with the fitted model. During feature-metabolite matching, pair the predicted RTs with MS1 m/z differences within 15 ppm, evaluating match validity and scores with getErrorAndScore(table_anno, is_rt_valid, is_ccs_valid, mz_ppm_thr, mz_tol, rt_tol_rel, ccs_tol_rel), where rt_tol_rel encodes the 30% relative RT tolerance. The rationale is that RT constraints prune redundant MRN nodes and edges, yielding the MS1-constrained network (ms1_constrained_mrn.rds) that anchors the two-layer topology.

## Related tools

- **MrnAnnoAlgo3** (Provides predictRtInMrn3 for training the column-type-specific random-forest RT model and getErrorAndScore for RT/m/z match scoring during MS1 pre-mapping) — github.com/ZhuMetLab/MrnAnnoAlgo3
- **randomForest** (R package supplying the random-forest algorithm underlying the RT prediction model)
- **MetDNA3** (Overall workflow in which the RT predictions feed the two-layer interactive networking topology) — http://metdna.zhulab.cn/

## Examples

```
rt_model <- predictRtInMrn3(wd, ms1_data, table_seed_long, md_mrn, column = "hilic")
```

## Evaluation signals

- ms1_constrained_mrn.rds exists and is a readable igraph graph object after subsetting
- Every retained metabolite-feature link satisfies both the 15 ppm m/z tolerance and the 30% relative RT tolerance
- Predicted RTs are available for all MRN metabolites, not only seed metabolites
- Seed metabolites (used for training) show small RT residuals, and is_rt_valid flags in getErrorAndScore output are consistent with the rt_tol_rel threshold

## Limitations

- RT prediction quality depends on the column type (column parameter) and on having sufficient seed metabolite annotations with measured RTs; without RT recalibration the model may mispredict
- RT alone cannot resolve isomeric metabolites, so the 30% RT constraint must be combined with m/z (and optionally CCS) constraints
- Function libraries (e.g., lib_rt) may be excluded from distributed R packages for copyright reasons, requiring users to supply their own libraries and repackage

## Evidence

- [methods] Training procedure for the RT model from seed annotations: [evidence span withheld — anchor status: not_found]
