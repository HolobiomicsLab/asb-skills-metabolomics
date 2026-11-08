---
name: mz-tolerance-peak-pooling-across-varieties
description: Use when two or more DESI-MSI mass-signature peak lists (e.g., DIMPLE-processed
  replicate roots per variety) must be merged at a fixed m/z tolerance and split into
  conserved vs. variety-unique signature partitions.
when_to_use_negative:
- Input is raw DESI-MSI imaging data (.cdf/.mat) that has not yet been processed by
  DIMPLE along the developmental gradient — run the upstream DIMPLE gradient analysis
  first
- The goal is metabolite annotation or identity assignment (use METASPACE, Metlin
  Classic, or LOTUS rather than cross-variety peak pooling)
- Data are not along a linear developmental axis or do not correspond to comparable
  root sections across varieties, since pooling assumes biologically comparable signature
  lists
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: DESI-MSI
  role: Produces the spatial metabolite imaging data from which the mass-signature
    peak lists being pooled originate
- name: DIMPLE
  role: Upstream pipeline (Developmental Imaging Mass Spectrometry Pipeline for Linear
    Evaluation) that generates the per-variety mass signature lists compared here
  repo: github.com/dickinsonlab/DIMPLE-code
- name: R
  role: Peak-compilation approach used to pool signatures at 0.001 m/z tolerance and
    classify them into the three overlap partitions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:95bb6889d6159a65bd89ae6eccff91683ad980a1dba155745a4a60e832dcccb5
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mz-tolerance-peak-pooling-across-varieties

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pools m/z mass signatures from DESI-MSI/DIMPLE analyses across biological varieties (e.g., B73 vs. Oaxacan Green maize roots) by grouping peaks whose m/z values fall within a 0.001 tolerance, then partitions the pooled signatures into conserved vs. variety-unique sets.

## When to use

Use when two or more DESI-MSI mass-signature peak lists (e.g., DIMPLE-processed replicate roots per variety) must be merged at a fixed m/z tolerance and split into conserved vs. variety-unique signature partitions.

## When NOT to use

- Input is raw DESI-MSI imaging data (.cdf/.mat) that has not yet been processed by DIMPLE along the developmental gradient — run the upstream DIMPLE gradient analysis first
- The goal is metabolite annotation or identity assignment (use METASPACE, Metlin Classic, or LOTUS rather than cross-variety peak pooling)
- Data are not along a linear developmental axis or do not correspond to comparable root sections across varieties, since pooling assumes biologically comparable signature lists

## Inputs

- DIMPLE-processed mass signature lists (m/z peaks) per replicate per variety, e.g., three B73 roots and three Oaxacan Green roots
- Raw DESI-MSI data as .cdf files and Matlab workspace .mat files (e.g., B73-root11, OG1, OG5) from the DIMPLE-code repository
- m/z pooling tolerance (0.001 in the reference workflow)

## Outputs

- signature_overlap_partition.csv — a three-part partition table of pooled mass signatures: conserved, unique to B73, unique to Oaxacan Green
- Per-partition signature counts (91 conserved, 210 B73-unique, 991 Oaxacan Green-unique in the reference dataset)

## How to apply

Load the DIMPLE-processed mass signature lists for each replicate of each variety (B73 salt-susceptible and Oaxacan Green salt-resilient, three replicates each, sourced from the cdf files and Matlab workspaces in the DIMPLE-code repository). Compile the m/z peaks from both variety lists using the R peak-compilation approach, pooling signatures whose m/z values fall within a 0.001 tolerance of one another. Classify each pooled signature into exactly one of three mutually exclusive partitions: conserved (present in both varieties), unique to B73, or unique to Oaxacan Green. Count the signatures per partition and write the three-part table to signature_overlap_partition.csv, reproducing the Supplemental Table 8 partition (91 conserved, 210 B73-unique, 991 Oaxacan Green-unique). The rationale is that tolerance-based pooling accounts for small instrumental m/z drift so that the same metabolite signal is not double-counted as unique.

## Related tools

- **DESI-MSI** (Produces the spatial metabolite imaging data from which the mass-signature peak lists being pooled originate)
- **DIMPLE** (Upstream pipeline (Developmental Imaging Mass Spectrometry Pipeline for Linear Evaluation) that generates the per-variety mass signature lists compared here) — github.com/dickinsonlab/DIMPLE-code
- **R** (Peak-compilation approach used to pool signatures at 0.001 m/z tolerance and classify them into the three overlap partitions)

## Examples

```
Rscript compile_peaks.R --b73 B73_peaks.csv --og OG_peaks.csv --tol 0.001 --out signature_overlap_partition.csv
```

## Evaluation signals

- Partition row counts match Supplemental Table 8 exactly: 91 conserved, 210 unique to B73, 991 unique to Oaxacan Green
- Every pooled signature appears in exactly one of the three partitions (no signature assigned to multiple partitions; partitions are mutually exclusive and exhaustive)
- All pooled signatures within a group differ by at most the 0.001 m/z tolerance
- Sum of the three partition counts equals the total number of distinct pooled signatures compiled from both varieties
- Output file signature_overlap_partition.csv is written with the three-part table schema

## Limitations

- DESI-MSI spatial resolution is approximately 50-100 micrometers, limiting which spatial structures the pooled signatures can resolve
- The partition result is sensitive to the pooling tolerance; the 0.001 tolerance is the calibrated value used for the Supplemental Table 8 comparison and changing it will change the 91/210/991 counts
- Most mass signatures remain unannotated — the majority of metabolites in plants and animals remain unannotated, so partitioning identifies overlap structure but not metabolite identities without database searching
- Reference data are tied to specific root replicates (B73-root11, B73-root2-5, B73-root-3-3, OG1, OG5, OG2-P11-0) provided in the DIMPLE-code repository; different tissues or gradients are untested

## Evidence

- [supplementary] identified 91 mass signatures conserved between the varieties, 210 unique to B73, and 991 unique to Oaxacan Green: [evidence span withheld — anchor status: not_found]
- [supplementary] pooling signatures whose m/z values fall within a 0.001 tolerance: [evidence span withheld — anchor status: not_found]
