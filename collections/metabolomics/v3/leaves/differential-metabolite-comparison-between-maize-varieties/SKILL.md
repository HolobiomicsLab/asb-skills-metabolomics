---
name: differential-metabolite-comparison-between-maize-varieties
description: Use when comparing two maize root DESI-MSI datasets whose mass signatures
  were already extracted along a linear developmental gradient by DIMPLE and you need
  a conserved/unique partition of pooled m/z peaks per variety.
when_to_use_negative:
- Input is raw MSI spectra (cdf files) not yet processed by DIMPLE along the linear
  developmental axis — use the DIMPLE gradient pipeline first to generate per-root
  mass signature lists.
- The comparison involves more than two varieties or non-maize tissue where the linear
  root developmental axis does not apply — the skill's partition thresholds and validation
  counts are specific to the B73 vs. Oaxacan Green maize root setup.
- The goal is annotating unknown mass signatures rather than partitioning conserved
  vs. unique ones — use METASPACE, Metlin Classic, or LOTUS instead.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DESI-MSI
  role: Source instrument producing mass spectrometry imaging data of developing maize
    roots along a longitudinal developmental gradient, the basis of the mass signature
    lists being compared
- name: DIMPLE
  role: Upstream pipeline that processes mass signatures along linear gradients to
    generate per-root mass signature lists; those lists are the direct inputs pooled
    and partitioned here
  repo: github.com/dickinsonlab/DIMPLE-code
- name: R
  role: Language used for the peak-compilation workflow that pools m/z signatures
    across replicates within a 0.001 tolerance and classifies each pooled signature
    as conserved or variety-unique
- name: Matlab workspace
  role: Data format (e.g. B73-root11-workspace.mat, OG1-workspace-2.mat) holding the
    per-root data referenced for each replicate root in the comparison
  repo: github.com/dickinsonlab/DIMPLE-code
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:e03514cb64ca1ea8c4820cc8070eed6d2c452bcddc882d3f01d63739da73c30e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/dickinsonlab/DIMPLE-code
  tools:
  - DESI-MSI
  - DIMPLE
  - Matlab workspace
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# differential-metabolite-comparison-between-maize-varieties

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pools DESI-MSI mass signatures from two maize varieties (within a 0.001 m/z tolerance, via DIMPLE peak-compilation in R) and partitions them into conserved vs. variety-unique sets, counting each partition — as done for salt-resilient Oaxacan Green vs. salt-susceptible B73 (91 conserved, 210 B73-unique, 991 OG-unique, Supplemental Table 8).

## When to use

Use when comparing two maize root DESI-MSI datasets whose mass signatures were already extracted along a linear developmental gradient by DIMPLE and you need a conserved/unique partition of pooled m/z peaks per variety.

## When NOT to use

- Input is raw MSI spectra (cdf files) not yet processed by DIMPLE along the linear developmental axis — use the DIMPLE gradient pipeline first to generate per-root mass signature lists.
- The comparison involves more than two varieties or non-maize tissue where the linear root developmental axis does not apply — the skill's partition thresholds and validation counts are specific to the B73 vs. Oaxacan Green maize root setup.
- The goal is annotating unknown mass signatures rather than partitioning conserved vs. unique ones — use METASPACE, Metlin Classic, or LOTUS instead.

## Inputs

- DIMPLE-processed mass signature lists for each replicate root per variety (e.g. B73-root11, B73-root2-5, B73-root-3-3; OG1, OG5, OG2-P11-0), from cdf files and Matlab workspaces in the DIMPLE-code repo
- R peak-compilation workflow for pooling m/z peaks across replicates within a 0.001 tolerance

## Outputs

- Three-part partition table (conserved / unique-to-B73 / unique-to-Oaxacan-Green) written to signature_overlap_partition.csv
- Counts of signatures in each partition (91 conserved, 210 B73-unique, 991 OG-unique), validated against Supplemental Table 8

## How to apply

Load the DIMPLE-processed mass signature lists for each replicate root (e.g. B73 roots 1–3 from B73-root11/B73-root2-5/B73-root-3-3 workspaces and Oaxacan Green OG1/OG5/OG2-P11-0), which are provided as cdf files and Matlab workspaces in the DIMPLE-code repo. Using the R peak-compilation approach, pool signatures across replicates whose m/z values fall within a 0.001 tolerance of one another, yielding a pooled signature list per variety. Classify each pooled signature into exactly one of three partitions: conserved (present in both varieties), unique to the first variety (B73), or unique to the second (Oaxacan Green). Count signatures per partition and write the three-part partition table (91 conserved, 210 B73-unique, 991 OG-unique) to signature_overlap_partition.csv, validating counts against Supplemental Table 8. This comparison underpins DIMPLE's identification of specific differences in mass signatures and overall enrichment patterns between the two varieties.

## Related tools

- **DESI-MSI** (Source instrument producing mass spectrometry imaging data of developing maize roots along a longitudinal developmental gradient, the basis of the mass signature lists being compared)
- **DIMPLE** (Upstream pipeline that processes mass signatures along linear gradients to generate per-root mass signature lists; those lists are the direct inputs pooled and partitioned here) — github.com/dickinsonlab/DIMPLE-code
- **R** (Language used for the peak-compilation workflow that pools m/z signatures across replicates within a 0.001 tolerance and classifies each pooled signature as conserved or variety-unique)
- **Matlab workspace** (Data format (e.g. B73-root11-workspace.mat, OG1-workspace-2.mat) holding the per-root data referenced for each replicate root in the comparison) — github.com/dickinsonlab/DIMPLE-code

## Evaluation signals

- Row counts of the three partitions must exactly match the values reported in Supplemental Table 8: 91 conserved, 210 unique to B73, 991 unique to Oaxacan Green.
- The three partitions must be mutually exclusive and collectively exhaustive — every pooled signature is assigned to exactly one partition.
- Pooling across replicates must collapse signatures whose m/z values differ by ≤ 0.001; verify that replicate-level signature counts exceed the pooled per-variety counts (i.e. tolerance-based merging occurred).
- The output CSV signature_overlap_partition.csv must contain exactly three partitions (conserved, B73-unique, Oaxacan Green-unique) with no unassigned rows.

## Limitations

- The 0.001 tolerance and resulting partition counts are specific to this B73 vs. Oaxacan Green DESI-MSI dataset; other datasets may require re-tuning tolerance.
- Partitioning relies on DIMPLE's per-root mass signature lists; errors or variability in upstream gradient processing propagate into the conserved/unique counts.
- Only two varieties were compared in the source study; extending to additional varieties or tissues is not validated here.

## Evidence

- [results] Comparison of the pooled unique DESI-MSI mass signatures for B73 and Oaxacan Green (0.001 tolerance) identified 91 mass signatures conserved between the varieties, 210 unique to B73, and 991 unique to Oaxacan Green.: [evidence span withheld — anchor status: not_found]
