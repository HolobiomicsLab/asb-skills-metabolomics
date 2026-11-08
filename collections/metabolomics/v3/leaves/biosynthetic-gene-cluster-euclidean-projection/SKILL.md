---
name: biosynthetic-gene-cluster-euclidean-projection
description: Use when you need to cluster large numbers of antiSMASH-predicted BGCs
  into GCFs (e.g., >100,000 BGCs) and pairwise network tools like BiG-SCAPE are computationally
  infeasible due to O(n²) runtime.
when_to_use_negative:
- The dataset is small enough for pairwise similarity-network tools (e.g., BiG-SCAPE)
  and you need pairwise distance matrices rather than Euclidean-space partitional
  clusters
- BGCs come from non-antiSMASH tools (ClusterFinder, DeepBGC) or have manually refined
  borders without first converting them with the provided generate_antismash_gbk.py
  script
- You need 'perfect' clustering identical to manually curated MIBiG compound groups
  — the paper shows V-score plateaus around 0.81 and BiG-SLiCE cannot capture the
  reference clustering exactly
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3174
tools:
- name: BiG-SLiCE
  role: Projects BGCs into Euclidean space and runs the partitional GCF clustering;
    outputs an SQLite3 database and query mode
  repo: github.com/medema-group/bigslice
- name: antiSMASH v5.1.1
  role: Upstream BGC prediction supplying the GenBank input files for clustering
- name: SQLite3
  role: Used to query the output database for GCF counts and per-BGC cluster assignments
    during postprocessing
- name: pyHMMER
  role: HMM scanning library used by BiG-SLiCE 2.0 to detect Pfam domains during feature
    extraction
  repo: althonos/pyhmmer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:e7ad8133fa71a8d0893eb6d692ac2440a7d8c70dc24c2bb97cbb80f39a4be7b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SLiCE
  - SQLite3
  - antiSMASH v5.1.1
  - pyHMMER
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# biosynthetic-gene-cluster-euclidean-projection

## Summary

BiG-SLiCE projects biosynthetic gene clusters (BGCs) into Euclidean space so they can be grouped into gene cluster families (GCFs) with a partitional, non-pairwise clustering algorithm of near-linear complexity, enabling GCF analysis at the scale of >1.2 million BGCs.

## When to use

Use when you need to cluster large numbers of antiSMASH-predicted BGCs into GCFs (e.g., >100,000 BGCs) and pairwise network tools like BiG-SCAPE are computationally infeasible due to O(n²) runtime.

## When NOT to use

- The dataset is small enough for pairwise similarity-network tools (e.g., BiG-SCAPE) and you need pairwise distance matrices rather than Euclidean-space partitional clusters
- BGCs come from non-antiSMASH tools (ClusterFinder, DeepBGC) or have manually refined borders without first converting them with the provided generate_antismash_gbk.py script
- You need 'perfect' clustering identical to manually curated MIBiG compound groups — the paper shows V-score plateaus around 0.81 and BiG-SLiCE cannot capture the reference clustering exactly

## Inputs

- BiG-SLiCE input folder of antiSMASH (e.g., v5.1.1) GenBank output files for BGCs
- MIBiG reference BGCs (e.g., v1.3 with compound-group labels, or v2.0 with 1,910 entries) for threshold calibration
- BiG-SLiCE pHMM database (download_bigslice_hmmdb; bigslice-models-2022-11-30)

## Outputs

- SQLite3 database in the output folder containing GCF models and per-BGC cluster assignments
- V-score and delta-GCF table per threshold for threshold selection
- Flask-based interactive visualization app served from the output folder
- Ranked GCF/BGC query results (--query mode) for placing new BGCs into existing GCFs

## How to apply

Prepare a BiG-SLiCE input folder of antiSMASH-derived GenBank files (per the wiki Input-folder spec), optionally excluding fragmented BGCs with --complete-only so only complete clusters (802,287 of 1,225,071 BGCs in the paper) build the models. Run 'bigslice -i <input_folder> <output_folder>'; BiG-SLiCE detects Pfam domains via its pHMM database (HMMER in v1.x, pyHMMER in v2.0, PFAM 35.0), represents each BGC as a feature vector in Euclidean space, and applies partitional clustering at a chosen threshold T — the paper tested T = {300, 600, 900, 1200, 1500}. For threshold selection, compute delta-GCF (change in GCF count between consecutive thresholds) and a V-score (harmonic mean of homogeneity and completeness) against curated MIBiG compound groups, then pick the threshold maximizing the V-score (T = 1100 gave V = 0.81 against 92 curated MIBiG v1.3 groups). Results are written to an SQLite3 database in the output folder; fragmented BGCs can be mapped back onto GCF models via the --query mode using membership values d with n_ranks=1.

## Related tools

- **BiG-SLiCE** (Projects BGCs into Euclidean space and runs the partitional GCF clustering; outputs an SQLite3 database and query mode) — github.com/medema-group/bigslice
- **antiSMASH v5.1.1** (Upstream BGC prediction supplying the GenBank input files for clustering)
- **SQLite3** (Used to query the output database for GCF counts and per-BGC cluster assignments during postprocessing)
- **pyHMMER** (HMM scanning library used by BiG-SLiCE 2.0 to detect Pfam domains during feature extraction) — althonos/pyhmmer

## Examples

```
bigslice -i <input_folder> <output_folder>  # then: bigslice --query <antismash_output_folder> --n_ranks 1 <output_folder>
```

## Evaluation signals

- V-score (harmonic mean of homogeneity and completeness) against MIBiG compound groups reproduces ~0.81 at T = 1100 within ±0.01
- GCF model count decreases monotonically as T increases from 300 to 1500; delta-GCF between consecutive thresholds is computable and traceable
- With --complete-only, the number of BGCs used to build GCF models equals the count of non-fragmented input BGCs (65% = 802,287 of 1,225,071 in the paper)
- Output folder contains a valid SQLite3 database; SQL queries return GCF assignments for every input BGC
- bigslice --version reports a consistent HMM database version (e.g., bigslice-models-2022-11-30) with matching Biosynthetic-pfam md5 37495cac452bf1dd8aff2c4ad92065fe

## Limitations

- Heuristic Euclidean-space clustering cannot reproduce the 'perfect' clustering of manually curated MIBiG compound groups (V-score ~0.81 at best tested threshold)
- Fragmented BGCs are excluded from model building and must be mapped onto GCFs post hoc via membership values d (n_ranks=1)
- Results are threshold-dependent: GCF count varies substantially across T = 300–1500, so threshold choice must be justified via V-score/delta-GCF calibration
- The pre-processed 1.2M-BGC query result distributed with the paper was built with BiG-SLiCE v1; no pre-processed result for v2 is yet available

## Evidence

- [abstract] Near-linear scaling rationale: "By representing them in Euclidean space, BiG-SLiCE can group BGCs into GCFs in a non-pairwise, near-linear fashion."
- [abstract] Scale demonstrated: "We used BiG-SLiCE to analyze 1,225,071 BGCs collected from 209,206 publicly available microbial genomes and metagenome-assembled genomes within 10 days on a typical 36-core CPU server."
- [results] Threshold sweep: "Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models"
- [results] Fragmented BGC exclusion: "the “–complete-only” parameter was used for the clustering phase, using 802,287 (65%) non-fragmented BGCs from the input data to build the GCF models"
- [results] Membership mapping of fragmented BGCs: "the full input datasets were queried back against the GCF models to map the fragmented BGCs onto their corresponding GCFs based on the calculated membership values d"
- [readme] CLI invocation: "user@local:~$ bigslice -i <input_folder> <output_folder>"
- [readme] Query mode: "user@local:~$ bigslice --query <antismash_output_folder> --n_ranks <int> <output_folder>"
- [readme] SQLite output access: [evidence span withheld — anchor status: not_found]
- [intro] Pairwise bottleneck motivation: [evidence span withheld — anchor status: not_found]
