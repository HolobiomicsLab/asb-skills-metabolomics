---
name: bgc-domain-architecture-similarity-comparison
description: Use when input is antiSMASH-processed GenBank files of BGCs and the goal
  is pairwise domain-architecture distances, a similarity network, or benchmarking
  clustering quality (NMI/ARI/V-score) against MIBiG curated groups.
when_to_use_negative:
- 'Datasets at the scale of hundreds of thousands to >1 million BGCs: pairwise comparison
  has quadratic O(n^2) complexity (BiG-SCAPE estimated at 37,000 hours on a 36-core
  CPU for the full dataset), so use BiG-SLiCE''s Euclidean-space clustering instead'
- Raw genome assemblies without prior BGC prediction and Pfam domain annotation —
  antiSMASH processing must happen first
- Clustering inputs are already distance matrices of another feature type (e.g. sequence-only
  or spectral similarity), not Pfam domain architectures
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SCAPE
  role: Computes pairwise BGC distances from domain architecture and sequence similarity,
    builds the weighted similarity network, and clusters it with Affinity Propagation;
    serves as the pairwise baseline being benchmarked
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: BiG-SLiCE
  role: Scalable non-pairwise alternative that projects BGCs into Euclidean space
    to avoid quadratic pairwise comparison when datasets grow too large
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH
  role: Upstream predictor that identifies BGCs in genomes and produces the domain-annotated
    GenBank outputs consumed by this comparison
  repo: https://antismash.secondarymetabolites.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:f8963086d5e69bec2864b2d502ca7f6747f22c3065966d4b8a7f02b2a6ffee43
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - BiG-SLiCE
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bgc-domain-architecture-similarity-comparison

## Summary

Compares biosynthetic gene clusters by their Pfam domain architecture to compute pairwise distances and/or build weighted similarity networks for gene cluster family (GCF) assignment, benchmarked against manually curated MIBiG group labels.

## When to use

Use when input is antiSMASH-processed GenBank files of BGCs and the goal is pairwise domain-architecture distances, a similarity network, or benchmarking clustering quality (NMI/ARI/V-score) against MIBiG curated groups.

## When NOT to use

- Datasets at the scale of hundreds of thousands to >1 million BGCs: pairwise comparison has quadratic O(n^2) complexity (BiG-SCAPE estimated at 37,000 hours on a 36-core CPU for the full dataset), so use BiG-SLiCE's Euclidean-space clustering instead
- Raw genome assemblies without prior BGC prediction and Pfam domain annotation — antiSMASH processing must happen first
- Clustering inputs are already distance matrices of another feature type (e.g. sequence-only or spectral similarity), not Pfam domain architectures

## Inputs

- antiSMASH-processed GenBank files with BGC Pfam domain annotations
- BiG-SCAPE/BiG-SLiCE input folder of BGC records
- Curated BGC group labels (e.g. the 92 manually curated MIBiG v1.3 groups) for benchmarking

## Outputs

- All-pairs BGC distance matrix / weighted BGC similarity network
- Per-BGC gene cluster family (GCF) assignments
- Benchmark metric table (NMI, ARI, V-score), e.g. v_score_bigslice.csv

## How to apply

Prepare an input folder of antiSMASH-annotated GenBank files containing Pfam domain annotations for each BGC (per the BiG-SLiCE wiki Input folder instructions). Compute all pairwise BGC distances based on domain architecture and sequence similarity — e.g. with BiG-SCAPE — and build the weighted similarity network. Cluster the network (BiG-SCAPE uses Affinity Propagation, which can find non-convex clusters) to assign each BGC to a GCF. Evaluate assignments against curated labels (e.g. the 92 manually curated MIBiG v1.3 groups) using normalized mutual information (NMI) and adjusted Rand index (ARI) from scikit-learn, taking the harmonic mean as the V-score. Be aware this pairwise approach is quadratic O(n^2); for datasets beyond tens of thousands of BGCs, use BiG-SLiCE's Euclidean-space projection with thresholds such as T = {300, 600, 900, 1,200, 1,500} instead.

## Related tools

- **BiG-SCAPE** (Computes pairwise BGC distances from domain architecture and sequence similarity, builds the weighted similarity network, and clusters it with Affinity Propagation; serves as the pairwise baseline being benchmarked) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **BiG-SLiCE** (Scalable non-pairwise alternative that projects BGCs into Euclidean space to avoid quadratic pairwise comparison when datasets grow too large) — https://github.com/medema-group/bigslice
- **antiSMASH** (Upstream predictor that identifies BGCs in genomes and produces the domain-annotated GenBank outputs consumed by this comparison) — https://antismash.secondarymetabolites.org/

## Examples

```
from sklearn.metrics import normalized_mutual_info_score, adjusted_rand_score; nmi = normalized_mutual_info_score(labels_true, labels_pred); ari = adjusted_rand_score(labels_true, labels_pred); v = 2*nmi*ari/(nmi+ari)
```

## Evaluation signals

- Reproduced pairwise baseline (BiG-SCAPE + Affinity Propagation) V-score matches the reported 0.91 on the 92 MIBiG v1.3 curated groups
- NMI and ARI are each computed with scikit-learn and the V-score equals their harmonic mean; all three values are written to v_score_bigslice.csv
- The similarity network is complete over the input BGCs (every pair compared), matching the all-pairs definition of the baseline
- Every input BGC receives exactly one GCF assignment, and cluster count is sensible relative to the number of curated reference groups (92)
- If switching to BiG-SLiCE, verify the run completes near-linearly and test multiple thresholds (T = 300/600/900/1200/1500), with GCF count decreasing as T increases

## Limitations

- Quadratic O(n^2) pairwise comparison does not scale: BiG-SCAPE was estimated to need ~37,000 hours on a 36-core CPU for ~1.2M BGCs
- Affinity Propagation finds non-convex clusters (helping V-score 0.91 vs BiG-SLiCE's 0.81 at T = 1,100) but is the slow component; results depend on network thresholds
- Tool version changes alter distances and clustering: BiG-SLiCE 2.0 switched to cosine-like (l2-normalized) distances and PFAM 35.0 pHMMs, and no pre-processed v2 reference result exists yet

## Evidence

- [intro] A major bottleneck is the expensive pairwise BGC comparison used to construct similarity networks and perform clustering analysis, leading to quadratic time complexity [O(n2)]: [evidence span withheld — anchor status: not_found]
- [intro] BiG-SCAPE runtime estimate: "even the currently fastest tool (a tool that we previously developed, BiG-SCAPE [22]) will require an estimated 37,000 hours of runtime on a 36-core CPU"
- [intro] BiG-SLiCE Euclidean alternative: "we introduce BiG-SLiCE (Biosynthetic Genes Super-Linear Clustering Engine), which projects BGCs into Euclidean space"
- [results] Threshold sweep for scalable variant: "Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models"
- [readme] Domain architecture as basis of GCFs: "by doing a large scale comparative analysis of homologous BGCs sharing similar domain architectures (we call them **"Gene Cluster Families (GCFs)"**)"
