---
name: bgc-gcf-clustering-threshold-optimization
description: Use when a BiG-SLiCE run's threshold parameter T must be selected or
  validated, e.g. when balancing the number of GCF models against fidelity to known
  MIBiG compound groups.
when_to_use_negative:
- Datasets are small enough for pairwise comparison tools like BiG-SCAPE, where network-based
  clustering is preferable and the Euclidean-space heuristic is unnecessary.
- BGCs are already assigned to curated families and no threshold tuning is needed.
- Fragmented BGCs must be clustered directly rather than mapped onto pre-built GCF
  models (use the query/membership-assignment mode with --complete-only model building
  instead).
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3174
tools:
- name: BiG-SLiCE
  role: Performs the threshold-parameterized Euclidean-space clustering of BGCs into
    GCFs and writes an SQLite3 output database
  repo: https://github.com/medema-group/bigslice
- name: SQLite3
  role: Query interface used to extract GCF counts and per-BGC cluster assignments
    from each BiG-SLiCE run's output database
- name: antiSMASH v5.1.1
  role: Predicts the BGCs that form the clustering input (with contigs <5,000 bp filtered
    via --minlength 5000)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:cfa5475b4274eabb4dd51cc4cc44913adad12e8d7a9c06319137d7d2694198a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - SQLite3
  - antiSMASH v5.1.1
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bgc-gcf-clustering-threshold-optimization

## Summary

Optimize the biosynthetic gene cluster (BGC) clustering threshold for BiG-SLiCE by sweeping T = 300–1,500 and scoring each run's gene cluster family (GCF) assignments against curated MIBiG reference groupings with the V-score (harmonic mean of homogeneity and completeness).

## When to use

Use when a BiG-SLiCE run's threshold parameter T must be selected or validated, e.g. when balancing the number of GCF models against fidelity to known MIBiG compound groups.

## When NOT to use

- Datasets are small enough for pairwise comparison tools like BiG-SCAPE, where network-based clustering is preferable and the Euclidean-space heuristic is unnecessary.
- BGCs are already assigned to curated families and no threshold tuning is needed.
- Fragmented BGCs must be clustered directly rather than mapped onto pre-built GCF models (use the query/membership-assignment mode with --complete-only model building instead).

## Inputs

- BiG-SLiCE input folder of antiSMASH-predicted reference BGCs (MIBiG v1.3 entries with compound-group annotations as ground-truth labels)
- Candidate threshold values T (e.g. {300, 600, 900, 1200, 1500} plus a finer value such as 1100)

## Outputs

- Per-threshold SQLite3 output database with GCF models and per-BGC cluster assignments
- Table of V-score and delta-GCF per threshold value
- Selected optimal threshold (reported optimum: T = 1100, V-score = 0.81)

## How to apply

Assemble a BiG-SLiCE input folder containing the reference BGCs (here, 92 MIBiG v1.3 compound groups whose annotations serve as ground-truth labels), then run clustering at T = {300, 600, 900, 1200, 1500} plus a finer candidate such as T = 1100 using `bigslice -i <input_folder> <output_folder>`. For each run, query the SQLite3 database in the output folder to extract the number of GCF models and each reference BGC's cluster assignment; GCF count decreases as T increases. Compute delta-GCF (change in GCF count between consecutive thresholds) and the V-score of the clustering against the MIBiG compound-group labels, then select the threshold maximizing the V-score. The rationale is that GCF models should recapitulate curated compound families without over-fragmenting; note BiG-SLiCE projects BGCs into Euclidean space for non-pairwise, near-linear partitional clustering, so threshold sweeps are computationally cheap compared to pairwise tools.

## Related tools

- **BiG-SLiCE** (Performs the threshold-parameterized Euclidean-space clustering of BGCs into GCFs and writes an SQLite3 output database) — https://github.com/medema-group/bigslice
- **SQLite3** (Query interface used to extract GCF counts and per-BGC cluster assignments from each BiG-SLiCE run's output database)
- **antiSMASH v5.1.1** (Predicts the BGCs that form the clustering input (with contigs <5,000 bp filtered via --minlength 5000))

## Examples

```
bigslice -i mibig_v13_input/ out_T1100/ && sqlite3 out_T1100/data.db "SELECT bgc_id, gcf_id FROM bgcs;"
```

## Evaluation signals

- V-score at the selected threshold T = 1100 reproduces the reported value of 0.81 within ±0.01.
- Number of GCF models decreases monotonically as T increases from 300 to 1,500.
- delta-GCF (change in GCF count between consecutive thresholds) is computed for every consecutive threshold pair.
- Cluster assignments for all 92 MIBiG v1.3 reference BGCs are extractable from the output SQLite3 database for every run.
- The chosen threshold maximizes the V-score across the tested sweep.

## Limitations

- BiG-SLiCE cannot capture the 'perfect' clustering of the 92 curated MIBiG v1.3 reference groups; V-score plateaus below 1.0 even at the optimum threshold.
- Results are version-dependent: BiG-SLiCE 2.0 switched to cosine-like (l2-normalized) distances and PFAM 35.0 pHMM databases, so V-scores calibrated on v1.x may not transfer.
- Fragmented BGCs were excluded from model building (--complete-only) in the published workflow and must instead be mapped onto GCF models via membership values d, which can bias assignments.

## Evidence

- [results] BiG-SLiCE produces a generally agreeable clustering result against the 92 curated MIBiG v1.3 reference groups at the selected example threshold T = 1,100, achieving a V-score of 0.81: [evidence span withheld — anchor status: not_found]
- [results] Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models: "Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models"
- [methods] bigslice input folder wiki quote from methods: "(see [wiki:Input folder](https://github.com/medema-group/bigslice/wiki/Input-folder) on how to prepare the input folder)"
- [readme] README invocation line: "user@local:~$ bigslice -i <input_folder> <output_folder>"
- [readme] README on SQLite output access: "output folder contains both the processed input data (in the form of an [SQLite3](https://www.sqlite.org/index.html) database file)"
