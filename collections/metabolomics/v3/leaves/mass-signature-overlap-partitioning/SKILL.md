---
name: mass-signature-overlap-partitioning
description: Use when two or more lists of DIMPLE-processed DESI-MSI mass signatures
  (m/z peaks from replicate root sections) must be pooled across varieties and classified
  as conserved vs. variety-unique.
when_to_use_negative:
- Input is raw imaging data rather than DIMPLE-processed mass signature lists — run
  the DIMPLE gradient pipeline (linear developmental axis assumption) first
- Data come from non-root or non-linear-gradient MSI samples where the linear developmental
  axis that DIMPLE leverages does not apply
- The goal is metabolite annotation/identity assignment (e.g., METASPACE, Metlin Classic,
  LOTUS database searches) rather than between-variety overlap quantification
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DIMPLE
  role: Upstream pipeline that produced the mass signature lists along the linear
    developmental gradient which are then pooled and partitioned
  repo: github.com/dickinsonlab/DIMPLE-code
- name: R
  role: Peak-compilation approach used to pool m/z peaks within the 0.001 tolerance
    and write the partition table
- name: DESI-MSI
  role: Imaging modality that generated the mass spectrometry data underlying the
    mass signatures being partitioned
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:2a648edb2f3f50019ec6670e35444edb1520193113b020552994ce4c9e1fa539
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
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mass-signature-overlap-partitioning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pooling and partitioning DIMPLE-processed DESI-MSI mass signatures from replicate roots of two maize varieties (B73 and Oaxacan Green) into conserved, B73-unique, and Oaxacan Green-unique sets using a 0.001 m/z tolerance, yielding the Supplemental Table 8 overlap statistics.

## When to use

Use when two or more lists of DIMPLE-processed DESI-MSI mass signatures (m/z peaks from replicate root sections) must be pooled across varieties and classified as conserved vs. variety-unique.

## When NOT to use

- Input is raw imaging data rather than DIMPLE-processed mass signature lists — run the DIMPLE gradient pipeline (linear developmental axis assumption) first
- Data come from non-root or non-linear-gradient MSI samples where the linear developmental axis that DIMPLE leverages does not apply
- The goal is metabolite annotation/identity assignment (e.g., METASPACE, Metlin Classic, LOTUS database searches) rather than between-variety overlap quantification

## Inputs

- DIMPLE-processed mass signature lists (m/z peaks) for three B73 replicate roots (B73-root11, B73-root2-5, B73-root-3-3 workspaces)
- DIMPLE-processed mass signature lists (m/z peaks) for three Oaxacan Green replicate roots (OG1, OG5, OG2-P11-0 workspaces)
- m/z matching tolerance parameter (0.001)

## Outputs

- signature_overlap_partition.csv — three-part partition table with per-signature assignments (conserved, B73-unique, Oaxacan Green-unique)
- Partition counts: 91 conserved, 210 unique to B73, 991 unique to Oaxacan Green (Supplemental Table 8)

## How to apply

Load the DIMPLE-processed mass signature lists for the three B73 (salt-susceptible) replicate roots and the three Oaxacan Green (salt-resilient) replicate roots, whose raw cdf files and Matlab workspaces are provided in the DIMPLE-code repository. Compile the m/z peaks from both variety lists using the R peak-compilation approach, merging signatures whose m/z values fall within a 0.001 tolerance of one another. Classify each pooled signature into exactly one of three mutually exclusive partitions: conserved (present in both varieties), B73-unique, or Oaxacan Green-unique. Count signatures per partition and write the three-part table to signature_overlap_partition.csv. Correct application reproduces Supplemental Table 8: 91 conserved, 210 B73-unique, and 991 Oaxacan Green-unique mass signatures.

## Related tools

- **DIMPLE** (Upstream pipeline that produced the mass signature lists along the linear developmental gradient which are then pooled and partitioned) — github.com/dickinsonlab/DIMPLE-code
- **R** (Peak-compilation approach used to pool m/z peaks within the 0.001 tolerance and write the partition table)
- **DESI-MSI** (Imaging modality that generated the mass spectrometry data underlying the mass signatures being partitioned)

## Evaluation signals

- Row counts of the three partitions in signature_overlap_partition.csv equal the Supplemental Table 8 values: 91 conserved, 210 B73-unique, 991 Oaxacan Green-unique
- Sum of the three partition counts equals the total number of pooled unique mass signatures (no signatures dropped or double-counted)
- Every pooled signature appears in exactly one partition (partitions are mutually exclusive and exhaustive)
- Pooling reproduces the same merged m/z groups when the 0.001 tolerance is re-applied independently
- Signature lists trace back to the correct DIMPLE workspaces for each of the three B73 and three Oaxacan Green replicate roots

## Limitations

- Mass signatures are unannotated m/z values — the article notes the majority of metabolites remain unannotated, so partitions describe peaks, not confirmed compounds
- Partitioning depends on DIMPLE's assumption of a linear developmental axis; non-linear or non-developmental MSI datasets are out of scope
- Results are specific to the three replicate roots per variety documented in the DIMPLE-code README (B73-root11, B73-root2-5, B73-root-3-3, OG1, OG5, OG2-P11-0); additional or fewer replicates will change the counts

## Evidence
