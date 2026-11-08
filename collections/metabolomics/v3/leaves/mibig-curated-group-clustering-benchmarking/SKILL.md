---
name: mibig-curated-group-clustering-benchmarking
description: Use when a new or existing BGC clustering method must be validated against
  the manually curated MIBiG reference groups, or when reporting a clustering-quality
  baseline (V-score, NMI, ARI) for a BGC-GCF grouping algorithm.
when_to_use_negative:
- The input is not an antiSMASH/BiG-SLiCE-prepared BGC dataset but an arbitrary feature
  table or an already-clustered result file — the skill expects raw BGC GenBank input
  and a clustering run.
- The goal is to cluster a large exploratory dataset (e.g. 1.2M BGCs) rather than
  to measure clustering quality against curated labels — use BiG-SLiCE's full clustering
  workflow instead.
- The MIBiG entries have fragmented or manually refined cluster borders that require
  the --complete-only filter; running on unfiltered fragmented BGCs will confound
  the benchmark.
edam_operation: http://edamontology.org/operation_3461
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: BiG-SCAPE
  role: Pairwise-network clustering baseline; builds a weighted similarity network
    from BGC domain architectures and clusters it with Affinity Propagation to produce
    GCF assignments scored against MIBiG curated groups.
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: BiG-SLiCE
  role: Partitional (non-pairwise) clustering of the same MIBiG reference set at threshold
    T=1,100 to produce the comparison GCF assignments (V-score 0.81).
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH
  role: 'Preprocessing step: predicts and annotates the BGCs in the MIBiG reference
    genomes before clustering.'
  repo: https://antismash.secondarymetabolites.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:b39f2b31489d1646aa7b092474a4237a5907435dcc4adb86e192e2ecc27506e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - BiG-SLiCE
  - antiSMASH
  edam_operation: http://edamontology.org/operation_3461
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# mibig-curated-group-clustering-benchmarking

## Summary

Benchmarks a BGC clustering tool by clustering the manually curated MIBiG v1.3 (or v2.0) reference set into groups and scoring assignments against the curated labels with NMI, ARI, and their harmonic-mean V-score. It provides the ground-truth baseline that BiG-SLiCE (V-score 0.81 at T=1,100) is compared against BiG-SCAPE's Affinity Propagation baseline (V-score 0.91).

## When to use

Use when a new or existing BGC clustering method must be validated against the manually curated MIBiG reference groups, or when reporting a clustering-quality baseline (V-score, NMI, ARI) for a BGC-GCF grouping algorithm.

## When NOT to use

- The input is not an antiSMASH/BiG-SLiCE-prepared BGC dataset but an arbitrary feature table or an already-clustered result file — the skill expects raw BGC GenBank input and a clustering run.
- The goal is to cluster a large exploratory dataset (e.g. 1.2M BGCs) rather than to measure clustering quality against curated labels — use BiG-SLiCE's full clustering workflow instead.
- The MIBiG entries have fragmented or manually refined cluster borders that require the --complete-only filter; running on unfiltered fragmented BGCs will confound the benchmark.

## Inputs

- antiSMASH-processed GenBank files for MIBiG v1.3 (92 curated groups) or MIBiG v2.0 (1,910 entries) reference BGCs
- BiG-SLiCE input folder prepared per wiki instructions (or BiG-SCAPE input folder with BGC domain annotations)

## Outputs

- Per-BGC GCF (cluster family) assignments for the MIBiG reference set
- V-score, NMI, and ARI metrics against the curated group labels (e.g. v_score_bigscape.csv)

## How to apply

Assemble the antiSMASH-processed GenBank files for the MIBiG reference entries and run the clustering tool under test — e.g. BiG-SCAPE, which builds a weighted pairwise similarity network from domain architectures and clusters it with Affinity Propagation to assign gene cluster families (GCFs). Alternatively run BiG-SLiCE at a fixed threshold (e.g. T=1,100, using --complete-only to exclude fragmented BGCs) and record the resulting GCF assignments. Map predicted GCF labels onto the curated MIBiG group labels, then compute normalized mutual information (NMI) and adjusted Rand index (ARI) with scikit-learn and their harmonic mean (V-score), writing metrics to v_score_bigscape.csv. The reproduced BiG-SCAPE baseline must match the reported V-score of 0.91; BiG-SLiCE at T=1,100 yields 0.81. Rationale: MIBiG's 92 manually curated groups provide independent, human-verified ground truth, so clustering quality on this set is a defensible proxy for real-world GCF recovery.

## Related tools

- **BiG-SCAPE** (Pairwise-network clustering baseline; builds a weighted similarity network from BGC domain architectures and clusters it with Affinity Propagation to produce GCF assignments scored against MIBiG curated groups.) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **BiG-SLiCE** (Partitional (non-pairwise) clustering of the same MIBiG reference set at threshold T=1,100 to produce the comparison GCF assignments (V-score 0.81).) — https://github.com/medema-group/bigslice
- **antiSMASH** (Preprocessing step: predicts and annotates the BGCs in the MIBiG reference genomes before clustering.) — https://antismash.secondarymetabolites.org/

## Examples

```
bigslice -i mibig_input_folder mibig_output --complete-only && python score_clusters.py --pred mibig_output/gcf_assignments.tsv --labels mibig_curated_groups.tsv --out v_score_bigscape.csv
```

## Evaluation signals

- The reproduced BiG-SCAPE V-score must match the reported value of 0.91 on the 92 MIBiG v1.3 curated groups.
- BiG-SLiCE at T=1,100 must reproduce V-score = 0.81 on the same curated groups.
- NMI and ARI must be computed with scikit-learn and their harmonic mean (V-score) must be reported; the three values must be consistent (V-score = harmonic mean of NMI and ARI).
- All 92 curated MIBiG groups must be represented in the input; any missing or fragmented BGC (filtered by --complete-only) must be excluded from the scoring.
- The output file v_score_bigscape.csv must exist and contain the three metrics.

## Limitations

- V-score on MIBiG curated groups is a proxy for real-world GCF recovery; it cannot capture the novelty of BGCs outside the reference set.
- The benchmark is sensitive to the clustering threshold parameter (BiG-SLiCE T=1,100 was chosen for its reported V-score), so comparing tools requires fixing the threshold a priori.
- Fragmented BGCs must be excluded (--complete-only) because they lack complete domain architectures needed for a fair domain-based comparison.
- BiG-SCAPE's pairwise O(n^2) complexity makes it infeasible for the 1.2M BGC dataset (estimated 37,000 hours on a 36-core CPU), so it is only used as a baseline on the curated subset.

## Evidence

- [results] BiG-SCAPE achieves a V-score of 0.91 on the 92 manually curated MIBiG v1.3 groups, outperforming BiG-SLiCE (V-score = 0.81 at T = 1,100) because its Affinity Propagation-based algorithm can find non-convex clusters.: [evidence span withheld — anchor status: not_found]
- [results] we incorporated all 1,910 entries from MIBiG v2.0 [78] as a reference set of known and experimentally verified BGCs: "we incorporated all 1,910 entries from MIBiG v2.0 [78] as a reference set of known and experimentally verified BGCs"
- [results] the “–complete-only” parameter was used for the clustering phase, using 802,287 (65%) non-fragmented BGCs from the input data to build the GCF models: "the “–complete-only” parameter was used for the clustering phase, using 802,287 (65%) non-fragmented BGCs from the input data to build the GCF models"
- [methods] our previous tool, [BiG-SCAPE](https://git.wageningenur.nl/medema-group/BiG-SCAPE): "our previous tool, [BiG-SCAPE](https://git.wageningenur.nl/medema-group/BiG-SCAPE)"
