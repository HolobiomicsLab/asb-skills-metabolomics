---
name: cluster-overlap-statistics-reporting
description: Use when two or more independently processed same-method nontargeted
  LC-MS datasets have been aligned with Eclipse (clique-based, diameter 1) and the
  analysis goal is quantifying feature overlap across datasets, e.g. common-to-all
  counts and fragmented cluster counts.
when_to_use_negative:
- Only a single dataset is being processed — redundant-feature clustering within one
  dataset is Gravity's role, not Eclipse alignment overlap statistics
- Datasets were not processed with the same LC-MS method — Eclipse is scoped to 'same-method
  nontargeted LCMS datasets'
- The task is drift correction or formatting/QC rather than cross-dataset alignment
  — Blueshift or Formation should be used instead
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Eclipse
  role: MSAligner performs the clique-based all-by-all alignment whose cluster assignments
    the overlap statistics are computed from
  repo: github.com/broadinstitute/bmxp
- name: bmxp
  role: Provides the shared schema (FMDATA/Feature Abundances, Compound_ID x injection_id)
    and merged-table format Eclipse requires
  repo: github.com/broadinstitute/bmxp
- name: Python
  role: Language used to import MSAligner, run the alignment, and compute/report the
    cluster overlap counts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:f4918e6620a45ec6c80d357258de1fabfc6c75092ed4388eb33ec7fcbef49fde
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cluster-overlap-statistics-reporting

## Summary

Computes and reports n-way feature overlap statistics (common to all datasets, present in ≥n datasets, and fragmented small clusters) from an Eclipse MSAligner clique-based alignment of same-method nontargeted LC-MS metabolomics datasets.

## When to use

Use when two or more independently processed same-method nontargeted LC-MS datasets have been aligned with Eclipse (clique-based, diameter 1) and the analysis goal is quantifying feature overlap across datasets, e.g. common-to-all counts and fragmented cluster counts.

## When NOT to use

- Only a single dataset is being processed — redundant-feature clustering within one dataset is Gravity's role, not Eclipse alignment overlap statistics
- Datasets were not processed with the same LC-MS method — Eclipse is scoped to 'same-method nontargeted LCMS datasets'
- The task is drift correction or formatting/QC rather than cross-dataset alignment — Blueshift or Formation should be used instead

## Inputs

- Feature Metadata table (bmxp.FMDATA, indexed by Compound_ID) for each dataset to align
- Feature Abundances pivot table (Feature x Injection, Compound_ID x injection_id) for each dataset
- Aligned feature table with Eclipse cluster assignments (e.g. from MSAligner run at minimum group size 1, minimum clique size 1, diameter 1)

## Outputs

- Cluster overlap counts: features common to all datasets (1435 for the six-plasma case), features in n≥4 datasets (6376), fragmented clusters of size n<4 (47522)
- Aligned feature table with cluster assignments exported as eclipse_alignment_results.csv

## How to apply

Load each dataset's Feature Metadata and Feature Abundances tables using the shared bmxp schema (Compound_ID and injection_id as index headers), then merge Feature Metadata + Feature Abundances per dataset, since Eclipse requires the merged representation. Import MSAligner from bmxp.eclipse and align the datasets with minimum group size = 1, minimum clique size = 1, and diameter = 1 (strict clique clustering), as specified in the Supplementary Code S4 workflow for the six-plasma demonstration (DS1-4, DS10, DS11). From the resulting alignment, bin clusters by the number of datasets in which each feature appears: features common to all six datasets (reported 1435), features present in ≥4 datasets (reported 6376), and fragmented clusters of size <4 (reported 47 522, a large number that results from allowing groups of size one). Write the aligned feature table with cluster assignments to eclipse_alignment_results.csv. Eclipse handles the n>2 matching problem with a graph-based approach, so overlap counts derive directly from the clique cluster assignments rather than pairwise merges.

## Related tools

- **Eclipse** (MSAligner performs the clique-based all-by-all alignment whose cluster assignments the overlap statistics are computed from) — github.com/broadinstitute/bmxp
- **bmxp** (Provides the shared schema (FMDATA/Feature Abundances, Compound_ID x injection_id) and merged-table format Eclipse requires) — github.com/broadinstitute/bmxp
- **Python** (Language used to import MSAligner, run the alignment, and compute/report the cluster overlap counts)

## Examples

```
from bmxp.eclipse import MSAligner; aligner = MSAligner(min_group_size=1, min_clique_size=1, diameter=1); aligned = aligner.align(datasets)
```

## Evaluation signals

- Feature count common to all six plasma datasets equals 1435
- Feature count in n≥4 datasets equals 6376
- Count of fragmented clusters of size n<4 equals 47522
- Output CSV eclipse_alignment_results.csv contains the aligned feature table with per-feature cluster assignments consistent with the computed counts
- Strict clique settings verified: minimum group size = 1, minimum clique size = 1, diameter = 1 (runtime reported ~52 s is hardware-dependent and not a pass/fail signal)

## Limitations

- Allowing groups of size one and minimum clique size 1 produces a very large number of fragmented clusters (47 522 with n<4); changing these parameters alters the reported counts
- Reported runtime (~52 s) is hardware-dependent and cannot serve as a strict validation criterion
- No version numbers for bmxp or its modules are documented, complicating reproducibility pinning
- Overlap statistics depend on upstream per-dataset feature clustering (Gravity) and drift correction (Blueshift) having been performed consistently

## Evidence

- [abstract] Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets.: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [readme] Align two or more same-method nontargeted LCMS datasets: "Align two or more same-method nontargeted LCMS datasets"
- [readme] Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances.: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] Feature Abundances - Pivot table of Feature x Injection (`Compound_ID` x `injection_id`) containing the abundances.: "Feature Abundances - Pivot table of Feature x Injection (`Compound_ID` x `injection_id`) containing the abundances."
- [methods] from bxmp.eclipse import MSAligner: "from bxmp.eclipse import MSAligner"
