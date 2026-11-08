---
name: mibig-reference-dataset-preparation
description: Use when you need a gold-standard benchmark of manually curated BGC groupings
  (MIBiG compound groups) to quantify how faithfully a clustering tool like BiG-SLiCE
  reproduces known compound families.
when_to_use_negative:
- Your BGCs come from tools not covered by antiSMASH/MIBiG (e.g., ClusterFinder or
  DeepBGC outputs or manually refined borders) — BiG-SLiCE provides a dedicated GenBank
  converter script for that case instead
- Your goal is large-scale diversity cataloging (e.g., 1.2M BGCs) rather than benchmarking
  clustering fidelity against curated reference groups
- Your input is already clustered or labeled and you only need label comparison metrics
  (homogeneity/completeness) — skip the dataset assembly and compute V-scores directly
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: BiG-SLiCE
  role: Consumes the prepared input folder of MIBiG reference BGCs and produces GCF
    cluster assignments at each threshold T, stored in an SQLite3 output database
  repo: https://github.com/medema-group/bigslice
- name: SQLite3
  role: Queries the BiG-SLiCE output database to extract GCF counts and per-BGC cluster
    assignments for V-score computation
- name: antiSMASH 5.0
  role: Source pipeline for BGC predictions in the reference dataset; MIBiG entries
    are antiSMASH-format BGCs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:f3c96f268f4db51bba9a48ce780ca2bb2ffc657b10d02fd615e54d6264b8b08c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - SQLite3
  - antiSMASH 5.0
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# mibig-reference-dataset-preparation

## Summary

Assembles MIBiG reference BGCs (e.g., the 92 curated MIBiG v1.3 compound groups) into a BiG-SLiCE input folder to serve as ground-truth labels for validating heuristic GCF clustering against a threshold sweep.

## When to use

Use when you need a gold-standard benchmark of manually curated BGC groupings (MIBiG compound groups) to quantify how faithfully a clustering tool like BiG-SLiCE reproduces known compound families.

## When NOT to use

- Your BGCs come from tools not covered by antiSMASH/MIBiG (e.g., ClusterFinder or DeepBGC outputs or manually refined borders) — BiG-SLiCE provides a dedicated GenBank converter script for that case instead
- Your goal is large-scale diversity cataloging (e.g., 1.2M BGCs) rather than benchmarking clustering fidelity against curated reference groups
- Your input is already clustered or labeled and you only need label comparison metrics (homogeneity/completeness) — skip the dataset assembly and compute V-scores directly

## Inputs

- MIBiG v1.3 reference BGCs in antiSMASH GenBank format, organized per the BiG-SLiCE input-folder spec
- MIBiG compound-group annotations for the 92 curated groups (ground-truth labels)

## Outputs

- BiG-SLiCE input folder of reference BGCs with ground-truth labels
- Per-threshold SQLite3 result databases with GCF models and per-BGC cluster assignments
- Table of V-score and delta-GCF per threshold (T=300–1500, T=1100)
- Selected threshold maximizing the V-score (reported: T=1100, V=0.81)

## How to apply

Assemble a BiG-SLiCE input folder containing the MIBiG v1.3 reference BGCs, following the wiki Input-folder specification, and attach each BGC's compound-group annotation as the ground-truth label. Run BiG-SLiCE clustering at the threshold sweep T = {300, 600, 900, 1200, 1500} plus the finer selected threshold T = 1100. For each run, query the SQLite3 database in the output folder to extract the number of GCF models and the GCF assignment of each reference BGC. Compute delta-GCF (change in GCF count between consecutive thresholds) and the V-score — the harmonic mean of homogeneity and completeness against the MIBiG compound groups — per threshold. Select the threshold maximizing the V-score; the T=1100 run should yield V ≈ 0.81 (±0.01).

## Related tools

- **BiG-SLiCE** (Consumes the prepared input folder of MIBiG reference BGCs and produces GCF cluster assignments at each threshold T, stored in an SQLite3 output database) — https://github.com/medema-group/bigslice
- **SQLite3** (Queries the BiG-SLiCE output database to extract GCF counts and per-BGC cluster assignments for V-score computation)
- **antiSMASH 5.0** (Source pipeline for BGC predictions in the reference dataset; MIBiG entries are antiSMASH-format BGCs)

## Examples

```
bigslice -i <input_folder> <output_folder>
```

## Evaluation signals

- V-score at T=1100 reproduces the reported 0.81 within ±0.01
- Number of GCF models decreases monotonically as T increases from 300 to 1500
- Every reference BGC has exactly one GCF assignment retrievable from the SQLite3 output database at each threshold
- delta-GCF values are computed for all consecutive threshold pairs in the sweep
- Selected threshold equals the argmax of the V-score column in the tabulated sweep

## Limitations

- BiG-SLiCE cannot capture the 'perfect' clustering of the curated MIBiG reference groups — V-score peaks around 0.81 rather than 1.0
- Clustering fidelity is threshold-dependent: GCF count falls as T rises, so a single-threshold result can misrepresent family granularity
- BiG-SLiCE projects BGCs into Euclidean space (heuristic), which trades exact pairwise fidelity for near-linear scalability

## Evidence

- [results] MIBiG reference set: "we incorporated all 1,910 entries from MIBiG v2.0 [78] as a reference set of known and experimentally verified BGCs"
- [results] threshold sweep: "Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models"
- [methods] input folder preparation: "(see [wiki:Input folder](https://github.com/medema-group/bigslice/wiki/Input-folder) on how to prepare the input folder)"
- [readme] output database: [evidence span withheld — anchor status: not_found]
