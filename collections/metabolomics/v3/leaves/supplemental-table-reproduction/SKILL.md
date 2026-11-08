---
name: supplemental-table-reproduction
description: Use when a paper publishes shared pipeline outputs (e.g., DIMPLE mass-signature
  lists) plus a numeric supplemental table, and the goal is to independently re-derive
  and validate those exact counts.
when_to_use_negative:
- Input is raw DESI-MSI imaging data (.cdf) that has not yet been processed by DIMPLE
  — run the upstream DIMPLE gradient analysis first
- The goal is metabolite annotation (e.g., METASPACE, Metlin Classic, LOTUS searches)
  rather than overlap counting of unannotated mass signatures
- The desired comparison uses a different m/z pooling tolerance than the published
  0.001, since counts will not match Supplemental Table 8
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DIMPLE
  role: Upstream pipeline that produced the mass-signature lists being partitioned;
    Developmental Imaging Mass Spectrometry Pipeline for Linear Evaluation (Sama et
    al. 2025 version)
  repo: https://github.com/dickinsonlab/DIMPLE-code
- name: R
  role: Peak-compilation approach used to pool m/z signatures within 0.001 tolerance
    and classify them into the three partitions
- name: DESI-MSI
  role: Imaging modality that generated the maize-root metabolite data underlying
    the mass signatures being compared
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:ed9435ae3cb88882232fab988ea22fe9a857131c1767841eb9a1266b53267dbe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/dickinsonlab/DIMPLE-code
  repo_url: https://github.com/dickinsonlab/DIMPLE-code
  tools:
  - DESI-MSI
  - DIMPLE
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# supplemental-table-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces a published supplemental-table result exactly, by re-running the paper's shared pipeline outputs — here, partitioning DIMPLE-processed DESI-MSI mass signatures from three B73 and three Oaxacan Green maize root replicates into conserved vs. variety-unique sets and validating counts against Supplemental Table 8.

## When to use

Use when a paper publishes shared pipeline outputs (e.g., DIMPLE mass-signature lists) plus a numeric supplemental table, and the goal is to independently re-derive and validate those exact counts.

## When NOT to use

- Input is raw DESI-MSI imaging data (.cdf) that has not yet been processed by DIMPLE — run the upstream DIMPLE gradient analysis first
- The goal is metabolite annotation (e.g., METASPACE, Metlin Classic, LOTUS searches) rather than overlap counting of unannotated mass signatures
- The desired comparison uses a different m/z pooling tolerance than the published 0.001, since counts will not match Supplemental Table 8

## Inputs

- DIMPLE-processed mass signature lists for three B73 root replicates (B73-root11, B73-root2-5, B73-root-3-3 workspaces)
- DIMPLE-processed mass signature lists for three Oaxacan Green root replicates (OG1, OG5, OG2-P11-0)
- m/z peak lists with tolerance parameter (0.001) for pooling

## Outputs

- signature_overlap_partition.csv containing the three-part partition: 91 conserved, 210 unique to B73, 991 unique to Oaxacan Green

## How to apply

Load the DIMPLE-processed mass-signature lists for the three B73 (salt-susceptible) and three Oaxacan Green (salt-resilient) root replicates distributed as .cdf files and Matlab workspaces in the DIMPLE-code repository (B73-root11, B73-root2-5, B73-root-3-3; OG1, OG5, OG2-P11-0). Compile the m/z peaks from both varieties using the R peak-compilation approach, pooling signatures whose m/z values fall within a 0.001 tolerance of one another. Assign each pooled signature to exactly one of three partitions: conserved (present in both), B73-unique, or Oaxacan Green-unique, ensuring no signature is double-counted. Write the three-part table to signature_overlap_partition.csv and check that the partition row counts equal the published values (91 conserved, 210 B73-unique, 991 Oaxacan Green-unique). The rationale is that supplemental tables are the paper's own ground truth: any discrepancy indicates an upstream processing or pooling-tolerance mismatch rather than a new biological finding.

## Related tools

- **DIMPLE** (Upstream pipeline that produced the mass-signature lists being partitioned; Developmental Imaging Mass Spectrometry Pipeline for Linear Evaluation (Sama et al. 2025 version)) — https://github.com/dickinsonlab/DIMPLE-code
- **R** (Peak-compilation approach used to pool m/z signatures within 0.001 tolerance and classify them into the three partitions)
- **DESI-MSI** (Imaging modality that generated the maize-root metabolite data underlying the mass signatures being compared)

## Evaluation signals

- Partition counts match Supplemental Table 8 exactly: 91 conserved, 210 B73-unique, 991 Oaxacan Green-unique
- Each pooled mass signature appears in exactly one partition (no double assignment, no signatures dropped)
- Sum of the three partitions equals the total number of pooled signatures (91 + 210 + 991 = 1292)
- Pooling was performed with the published 0.001 m/z tolerance, not a different threshold
- All six replicates (3 B73, 3 Oaxacan Green) from the repository workspaces were included in the pooled comparison

## Limitations

- Reproduction requires the exact DIMPLE-processed outputs; the repository distributes .cdf files and Matlab workspaces for the analyzed roots (e.g., OG2-P11-0 lacking a paired workspace file in the README listing), so missing or renamed files block the workflow
- Partition counts are sensitive to the pooling tolerance; a tolerance other than 0.001 will change how many signatures are conserved vs. unique
- Mass signatures are unannotated m/z peaks, so the partition reflects chemical-feature overlap, not confirmed metabolite identity

## Evidence
