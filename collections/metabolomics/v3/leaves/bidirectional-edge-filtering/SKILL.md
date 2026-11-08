---
name: bidirectional-edge-filtering
description: Use when combining pairwise subalignment match tables from two or more
  independently processed nontargeted LC-MS datasets into a feature graph, and only
  mutually supported (bidirectional) matches should become edges before clustering
  into feature groups.
when_to_use_negative:
- The input is a single dataset rather than two or more independently processed datasets
  that must be aligned (Eclipse's scope is aligning multiple same-method datasets)
- Only one-directional pairwise match evidence exists — these edges will be discarded,
  so the skill would eliminate rather than reconcile the matches
- The task is redundancy clustering of features within a single dataset by RT and
  correlation — that is Gravity's role, not Eclipse graph alignment
edam_operation: http://edamontology.org/operation_3432
tools:
- name: Eclipse
  role: Provides the MSAligner that loads subalignment match tables into the directed
    graph, applies bidirectional edge retention with summed penalties, and runs the
    group-ranking clustering to form feature groups
  repo: https://github.com/broadinstitute/bmxp
- name: Python
  role: Runtime language in which Eclipse (and the bmxp package providing MSAligner)
    is implemented and invoked
  repo: https://github.com/broadinstitute/bmxp
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:8bc25af14d8335dc5b2d3bdec8356978bb6761ae6f4180f359a115ffc60a3141
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/broadinstitute/bmxp
  tools:
  - Eclipse
  - Python
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bidirectional-edge-filtering

## Summary

A graph-construction step in the Eclipse LC-MS alignment workflow where all pairwise subalignment match tables are loaded into a directed graph (features as nodes, matches as edges, match penalties as edge weights), and only edges supported by matches in both directions between datasets are retained, with penalties from both directions summed. This handles complex matching scenarios arising when n > 2 datasets are aligned.

## When to use

Use when combining pairwise subalignment match tables from two or more independently processed nontargeted LC-MS datasets into a feature graph, and only mutually supported (bidirectional) matches should become edges before clustering into feature groups.

## When NOT to use

- The input is a single dataset rather than two or more independently processed datasets that must be aligned (Eclipse's scope is aligning multiple same-method datasets)
- Only one-directional pairwise match evidence exists — these edges will be discarded, so the skill would eliminate rather than reconcile the matches
- The task is redundancy clustering of features within a single dataset by RT and correlation — that is Gravity's role, not Eclipse graph alignment

## Inputs

- Pairwise dataset match tables from completed subalignments (feature match pairs with match penalties)
- Merged Feature Metadata + Feature Abundances table in the BMXP shared schema (Compound_ID feature index, injection_id injection index)
- Clustering configuration parameters: minimum group size, minimum clique size, diameter setting (1, 2, or 3)

## Outputs

- Directed feature-match graph restricted to bidirectional edges with summed match-penalty weights
- Clustered feature groups written to eclipse_feature_groups.csv, keyed by feature ID per the shared schema

## How to apply

Load the merged Feature Metadata + Feature Abundances table (BMXP shared schema: Compound_ID as feature index, injection_id as injection index) required by Eclipse. Load the pairwise dataset match tables into a directed graph in the Eclipse MSAligner, where features are nodes, matches are edges, and match penalties are edge weights. Retain only edges that are bidirectional between datasets, summing the match penalties from both directions for each retained edge — this drops one-directional matches that lack reciprocal support. Then run the iterative group-ranking clustering algorithm with the configured minimum group size, minimum clique size, and diameter setting (1, 2, or 3) to form feature groups across datasets. The rationale is that requiring reciprocal matches before weighting makes the resulting feature grouping robust for more than two datasets, where pairwise alignment alone cannot resolve complex matching scenarios.

## Related tools

- **Eclipse** (Provides the MSAligner that loads subalignment match tables into the directed graph, applies bidirectional edge retention with summed penalties, and runs the group-ranking clustering to form feature groups) — https://github.com/broadinstitute/bmxp
- **Python** (Runtime language in which Eclipse (and the bmxp package providing MSAligner) is implemented and invoked) — https://github.com/broadinstitute/bmxp

## Examples

```
from bxmp.eclipse import MSAligner
```

## Evaluation signals

- Every retained edge is supported by matches in both directions between the two datasets, and its weight equals the sum of the two directional match penalties
- Order-independence invariant: rerunning the alignment with a different dataset insertion order yields identical feature group assignments
- Output eclipse_feature_groups.csv is keyed by Compound_ID per the BMXP shared schema headers (injection_id for injections)
- Resulting feature groups respect the configured minimum group size, minimum clique size, and diameter setting (1, 2, or 3)
- All features present in the merged Feature Metadata + Feature Abundances input appear as graph nodes or are accounted for as unmatched

## Limitations

- Designed for aligning two or more same-method nontargeted LC-MS datasets; it presumes features were extracted consistently across datasets
- Eclipse and Blueshift require merging Feature Metadata + Feature Abundances in the BMXP shared schema, so mismatched column headers must be remapped via the bmxp.FMDATA / bmxp.IMDATA dictionaries before loading
- No version numbers for bmxp or its modules are documented in the article's methods
- XIC shape-based feature clustering is not yet implemented in the companion Gravity tool, limiting downstream cluster refinement

## Evidence

- [abstract] Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets.: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [readme] Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances.: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] All BMXP modules use a shared schema and file formats with our prefered columns headers.: "All BMXP modules use a shared schema and file formats with our prefered columns headers."
