---
name: bgc-euclidean-feature-vectorization
description: Use when BGC datasets are too large for pairwise comparison tools (e.g.,
  hundreds of thousands to millions of BGCs from antiSMASH runs) and GCF clustering
  must scale near-linearly, or when fragmented BGCs must be mapped onto prebuilt GCF
  centroid models via query mode.
when_to_use_negative:
- When the dataset is small enough for pairwise, network-based comparison (BiG-SCAPE)
  and sequence-level similarity detail per BGC pair is required
- When BGCs come from tools other than antiSMASH/MIBiG (e.g., ClusterFinder, DeepBGC,
  or manually refined borders) without first converting them via the provided generate_antismash_gbk.py
  converter script
- When contigs/fragments are the object of interest rather than complete clusters
  — the '--complete-only' model-building phase excludes fragmented BGCs (35% of the
  input in the reference run)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: BiG-SLiCE
  role: Projects BGCs into Euclidean space from pHMM/Pfam features and performs partitional,
    near-linear GCF clustering with --complete-only and --query modes
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH v5.1.1
  role: Predicts the BGCs (with --minlength 5000 contig filtering) whose GenBank files
    form the input feature space
  repo: https://antismash.secondarymetabolites.org/
- name: pyHMMER
  role: Python-native HMM scanning library used by BiG-SLiCE 2.0 to score Pfam domains,
    replacing standalone HMMER
  repo: https://github.com/althonos/pyhmmer
- name: ClusterFinder
  role: Alternative BGC predictor whose output must be converted to antiSMASH-style
    GenBank before vectorization
  repo: https://github.com/petercim/ClusterFinder
- name: DeepBGC
  role: Alternative BGC predictor whose output must be converted to antiSMASH-style
    GenBank before vectorization
  repo: https://github.com/Merck/deepbgc
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:8e4a45c8d87494260efaff6c3e5b185112a98629e1910777f2b0f650f71a6f64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - ClusterFinder
  - DeepBGC
  - antiSMASH v5.1.1
  - pyHMMER
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bgc-euclidean-feature-vectorization

## Summary

Vectorizes biosynthetic gene clusters (BGCs) into Euclidean feature space based on Pfam domain (pHMM hit) composition, enabling non-pairwise, near-linear gene cluster family (GCF) clustering instead of quadratic O(n^2) pairwise comparisons as in BiG-SCAPE.

## When to use

Use when BGC datasets are too large for pairwise comparison tools (e.g., hundreds of thousands to millions of BGCs from antiSMASH runs) and GCF clustering must scale near-linearly, or when fragmented BGCs must be mapped onto prebuilt GCF centroid models via query mode.

## When NOT to use

- When the dataset is small enough for pairwise, network-based comparison (BiG-SCAPE) and sequence-level similarity detail per BGC pair is required
- When BGCs come from tools other than antiSMASH/MIBiG (e.g., ClusterFinder, DeepBGC, or manually refined borders) without first converting them via the provided generate_antismash_gbk.py converter script
- When contigs/fragments are the object of interest rather than complete clusters — the '--complete-only' model-building phase excludes fragmented BGCs (35% of the input in the reference run)

## Inputs

- antiSMASH-predicted BGC GenBank files for bacterial, archaeal, and fungal genomes and MAGs (e.g., 209,206 genomes / 1,225,071 BGCs)
- BiG-SLiCE HMM database (bigslice-models, downloaded via download_bigslice_hmmdb)
- BiG-SLiCE input folder prepared per the project wiki

## Outputs

- Euclidean feature vectors of BGC Pfam domain compositions stored in an SQLite3 datastore
- GCF centroid models built from non-fragmented ('--complete-only') BGCs at threshold T
- GCF membership assignments (distance d) for all BGCs, including fragmented ones via --query mode

## How to apply

Run antiSMASH (v5.1.1 with '–minlength 5000' to filter contigs <5,000 bp) to detect BGCs, then scan cluster proteins against the BiG-SLiCE pHMM database (downloaded via 'download_bigslice_hmmdb') to obtain Pfam domain feature vectors in Euclidean space. Build GCF centroid models only from non-fragmented BGCs using 'bigslice -i <input_folder> <output_folder>' with '--complete-only' at chosen threshold T (e.g., T = 300 in the full-scale run; T = {300, 600, 900, 1200, 1500} were tested), which uses 802,287 of 1,225,071 BGCs (65%) in the reference study. Map fragmented BGCs by querying the full dataset back against the GCF models in '--query' mode with '--n_ranks 1', assigning membership by distance d. Store all processed input data, GCF models, and cluster assignments in the SQLite3 database in the output folder; the workflow completed 1,225,071 BGCs from 209,206 genomes in ~240 hours on a 36-core CPU with 128 GB RAM.

## Related tools

- **BiG-SLiCE** (Projects BGCs into Euclidean space from pHMM/Pfam features and performs partitional, near-linear GCF clustering with --complete-only and --query modes) — https://github.com/medema-group/bigslice
- **antiSMASH v5.1.1** (Predicts the BGCs (with --minlength 5000 contig filtering) whose GenBank files form the input feature space) — https://antismash.secondarymetabolites.org/
- **pyHMMER** (Python-native HMM scanning library used by BiG-SLiCE 2.0 to score Pfam domains, replacing standalone HMMER) — https://github.com/althonos/pyhmmer
- **ClusterFinder** (Alternative BGC predictor whose output must be converted to antiSMASH-style GenBank before vectorization) — https://github.com/petercim/ClusterFinder
- **DeepBGC** (Alternative BGC predictor whose output must be converted to antiSMASH-style GenBank before vectorization) — https://github.com/Merck/deepbgc

## Examples

```
bigslice -i <input_folder> <output_folder> --complete-only && bigslice --query <antismash_output_folder> --n_ranks 1 <output_folder>
```

## Evaluation signals

- The populated SQLite3 datastore contains exactly 1,225,071 BGCs from 209,206 genomes and MAGs (Table 1 counts)
- Model-building phase used only the 802,287 (65%) non-fragmented BGCs when --complete-only was applied
- Feature extraction through membership assignment completed within 10 days (~240 hours) on a 36-core CPU server with 128 GB RAM
- 'bigslice --version' reports the expected version banner and HMM database identifiers (e.g., Biosynthetic-pfam and Sub-pfam md5 checksums)
- Every fragmented BGC received a GCF assignment based on membership value d from the --query run with --n_ranks 1

## Limitations

- Pairwise approaches such as BiG-SCAPE would need an estimated 37,000 hours on a 36-core CPU for the same dataset — Euclidean projection trades per-pair sequence resolution for scalability
- Fragmented BGCs (~35% of the 1.2M input) are not used to build GCF centroids; they are only assigned by query-back membership
- Pre-processed result files for BiG-SLiCE v2 (cosine-like/l2-normalized distances, PFAM 35.0) are not yet available; the released 17GB full-run result corresponds to v1

## Evidence

- [abstract] Euclidean projection enables near-linear clustering: "By representing them in Euclidean space, BiG-SLiCE can group BGCs into GCFs in a non-pairwise, near-linear fashion."
- [results] complete-only filter for model building: "the “–complete-only” parameter was used for the clustering phase, using 802,287 (65%) non-fragmented BGCs from the input data to build the GCF models"
- [results] query-back of fragmented BGCs: "the full input datasets were queried back against the GCF models to map the fragmented BGCs onto their corresponding GCFs based on the calculated membership values d"
- [results] antiSMASH contig length filter: "We then predicted their BGCs using antiSMASH v5.1.1, filtering out contigs <5,000 bp (“–minlength 5000”)"
- [readme] README: query mode and full-run scale: "Using the `--query` mode, you can perform a blazing-fast query of a putative BGC against the pre-processed set of Gene Cluster Family (GCF) models"
