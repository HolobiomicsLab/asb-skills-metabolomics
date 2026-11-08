---
name: vscore-clustering-quality-evaluation
description: Use when benchmarking BiG-SLiCE GCF clustering against curated MIBiG
  compound-group reference groupings, e.g., to pick a clustering threshold T by maximizing
  the V-score across a sweep.
when_to_use_negative:
- Your dataset has no trusted reference labels (e.g., 1.2M predicted BGCs from NCBI
  RefSeq genomes) — there is no ground truth to compute a V-score against; use the
  clustering skill itself instead
- You are using BiG-SLiCE v2.0, which switched to cosine-like (l2-normalized) distances
  and PFAM 35.0 — published V-scores from the v1 Euclidean-space paper will not directly
  transfer
- Your input is fragmented BGCs only, or you need pairwise BGC network analysis (BiG-SCAPE
  territory) rather than Euclidean-space partitional clustering evaluation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SLiCE
  role: Performs the Euclidean-space partitional clustering of BGCs into GCFs at each
    threshold T, whose assignments are evaluated with the V-score
  repo: https://github.com/medema-group/bigslice
- name: SQLite3
  role: Querying the BiG-SLiCE output database to extract GCF counts and per-BGC cluster
    assignments for V-score computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:5b1cb2c08f5c868d8c3021fc973d73de73d3fbe05b3c073784e300ab70228c7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - SQLite3
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# vscore-clustering-quality-evaluation

## Summary

Evaluates BiG-SLiCE GCF clustering quality against a curated reference set by sweeping distance thresholds T = 300–1,500, querying the output SQLite3 database, and computing the V-score (harmonic mean of homogeneity and completeness) against MIBiG compound-group labels to select an optimal threshold.

## When to use

Use when benchmarking BiG-SLiCE GCF clustering against curated MIBiG compound-group reference groupings, e.g., to pick a clustering threshold T by maximizing the V-score across a sweep.

## When NOT to use

- Your dataset has no trusted reference labels (e.g., 1.2M predicted BGCs from NCBI RefSeq genomes) — there is no ground truth to compute a V-score against; use the clustering skill itself instead
- You are using BiG-SLiCE v2.0, which switched to cosine-like (l2-normalized) distances and PFAM 35.0 — published V-scores from the v1 Euclidean-space paper will not directly transfer
- Your input is fragmented BGCs only, or you need pairwise BGC network analysis (BiG-SCAPE territory) rather than Euclidean-space partitional clustering evaluation

## Inputs

- 92 MIBiG v1.3 reference BGCs with compound-group annotations (ground-truth labels)
- BiG-SLiCE input folder (antiSMASH GenBank BGC files prepared per the bigslice wiki Input-folder guide)
- BiG-SLiCE output folder containing the SQLite3 database of processed input data

## Outputs

- GCF models and cluster assignment of each reference BGC per threshold T
- delta-GCF table (change in GCF count between consecutive thresholds T = 300, 600, 900, 1,200, 1,500)
- V-score (harmonic mean of homogeneity and completeness) per threshold, used to select the optimal T

## How to apply

Assemble a BiG-SLiCE input folder containing the 92 MIBiG v1.3 reference BGCs (compound-group annotations serve as ground-truth labels) following the medema-group/bigslice wiki 'Input folder' guide. Run the clustering analysis at the tested thresholds T = {300, 600, 900, 1,200, 1,500} plus the finer T = 1,100 with `bigslice -i <input_folder> <output_folder>`. For each run, query the SQLite3 database in the output folder to extract the number of GCF models and the GCF assignment of each reference BGC. Compute delta-GCF (change in GCF count between consecutive thresholds) and the V-score — the harmonic mean of homogeneity and completeness of the clustering versus the MIBiG compound groups — for every threshold, then select the threshold maximizing the V-score. Note that BiG-SLiCE builds GCF models in Euclidean space via non-pairwise partitional clustering, so the V-score reflects how faithfully this heuristic reproduces the manual reference groupings rather than 'perfect' clustering.

## Related tools

- **BiG-SLiCE** (Performs the Euclidean-space partitional clustering of BGCs into GCFs at each threshold T, whose assignments are evaluated with the V-score) — https://github.com/medema-group/bigslice
- **SQLite3** (Querying the BiG-SLiCE output database to extract GCF counts and per-BGC cluster assignments for V-score computation)

## Examples

```
bigslice -i mibig_v1.3_input_folder output_T1100 && sqlite3 output_T1100/data.db "SELECT bgc_id, gcf_id FROM assignments;"
```

## Evaluation signals

- V-score at the selected threshold T = 1,100 reproduces the reported value of 0.81 within ±0.01
- Number of GCF models decreases monotonically as T increases from 300 to 1,500
- Every one of the 92 MIBiG v1.3 reference BGCs has exactly one GCF assignment in each threshold run
- delta-GCF is non-negative and finite for every consecutive threshold pair in the sweep
- V-score decomposes into homogeneity and completeness components whose harmonic mean equals the reported value

## Limitations

- BiG-SLiCE's heuristic Euclidean-space clustering cannot capture the 'perfect' clustering of the manually curated MIBiG reference groups
- Fragmented BGCs are excluded from model building with --complete-only (802,287 of ~1.2M, 65%, non-fragmented BGCs were used to build GCF models), so clustered input is a subset
- BiG-SLiCE 2.0 changed the distance metric to cosine-like (via l2-normalization) and updated pHMM databases to PFAM 35.0, so threshold-specific results differ from the v1-era Euclidean-distance study

## Evidence

- [results] Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models: "Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models"
- [results] complete-only filtering for GCF model building: "the “–complete-only” parameter was used for the clustering phase, using 802,287 (65%) non-fragmented BGCs from the input data to build the GCF models"
- [readme] BiG-SLiCE clustering run command: "user@local:~$ bigslice -i <input_folder> <output_folder>"
- [readme] SQLite access to BiG-SLiCE output data: "To access **BiG-SLiCE**'s preprocessed data, (advanced) users need to be able to [run SQL(ite) queries]"
