---
name: directed-graph-clustering
description: Use when two or more independently processed LC-MS datasets must be aligned
  into a Combined Dataset, and pairwise subalignment match tables are available to
  load into a graph-based clustering step.
when_to_use_negative:
- The input is already a single aligned/complete dataset with no pairwise match tables
  to reconcile
- The task is clustering redundant features within one dataset based on RT and correlation
  — that is Gravity's cluster step, not Eclipse's cross-dataset graph alignment
- The task is drift correction via pooled technical replicates and internal standards
  — that is Blueshift's DriftCorrection, not graph-based alignment
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Eclipse
  role: 'Core tool: its MSAligner loads subalignment match tables into a directed
    graph and runs the iterative group-ranking clustering to produce feature groups'
  repo: https://github.com/broadinstitute/bmxp
- name: bmxp
  role: Package providing the shared schema (bmxp.FMDATA, bmxp.IMDATA) and the Eclipse
    MSAligner module used for graph construction and clustering
  repo: https://github.com/broadinstitute/bmxp
- name: Python
  role: Implementation language in which Eclipse and the MSAligner are invoked
  repo: https://github.com/broadinstitute/bmxp
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:3bc0bf213d8faf8380d497d714160eb37d531e8e2aea7c497710b13eec07ced7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/broadinstitute/bmxp
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# directed-graph-clustering

## Summary

Eclipse aligns two or more independently processed nontargeted LC-MS metabolomics datasets by loading pairwise subalignment match tables into a directed graph (features = nodes, matches = edges, match penalties = edge weights), retaining bidirectional edges, and running an iterative group-ranking clustering to form cross-dataset feature groups.

## When to use

Use when two or more independently processed LC-MS datasets must be aligned into a Combined Dataset, and pairwise subalignment match tables are available to load into a graph-based clustering step.

## When NOT to use

- The input is already a single aligned/complete dataset with no pairwise match tables to reconcile
- The task is clustering redundant features within one dataset based on RT and correlation — that is Gravity's cluster step, not Eclipse's cross-dataset graph alignment
- The task is drift correction via pooled technical replicates and internal standards — that is Blueshift's DriftCorrection, not graph-based alignment

## Inputs

- Merged Feature Metadata + Feature Abundances table (required by Eclipse), with Compound_ID as feature index and injection_id as injection index per the BMXP shared schema
- Pairwise subalignment match tables between datasets, containing matched feature pairs and match penalties

## Outputs

- eclipse_feature_groups.csv — clustered feature groups keyed by feature ID per the shared BMXP schema

## How to apply

Load the merged Feature Metadata + Feature Abundances table required by Eclipse using the shared BMXP schema headers (Compound_ID as the feature index, injection_id as the injection index). Import the MSAligner from bmxp.eclipse and load the pairwise dataset match tables into a directed graph in which features are nodes, matches are edges, and match penalties are edge weights. Retain only edges that are bidirectional between datasets, summing the match penalties from both directions for each retained edge. Run the iterative group-ranking clustering algorithm configured with the minimum group size, minimum clique size, and diameter setting (1, 2, or 3) to form feature groups across datasets. Write the clustered feature groups to eclipse_feature_groups.csv keyed by feature ID per the shared schema, then verify order-independence by rerunning with a different dataset insertion order and confirming identical group assignments.

## Related tools

- **Eclipse** (Core tool: its MSAligner loads subalignment match tables into a directed graph and runs the iterative group-ranking clustering to produce feature groups) — https://github.com/broadinstitute/bmxp
- **bmxp** (Package providing the shared schema (bmxp.FMDATA, bmxp.IMDATA) and the Eclipse MSAligner module used for graph construction and clustering) — https://github.com/broadinstitute/bmxp
- **Python** (Implementation language in which Eclipse and the MSAligner are invoked) — https://github.com/broadinstitute/bmxp

## Examples

```
import bmxp; from bxmp.eclipse import MSAligner
```

## Evaluation signals

- Order-independence invariant: rerun the alignment with a different dataset insertion order and confirm the resulting feature group assignments are identical
- Output eclipse_feature_groups.csv is keyed by feature ID using the shared BMXP schema headers (Compound_ID, injection_id)
- Every retained edge in the graph is bidirectional between datasets, with match penalties summed from both directions
- Resulting feature groups satisfy the configured minimum group size, minimum clique size, and diameter setting (1, 2, or 3)

## Limitations

- Eclipse is designed for aligning same-method nontargeted LCMS datasets; datasets processed under different methods fall outside its stated scope
- Schema headers are global: if column names differ, bmxp.FMDATA['Compound_ID'] and bmxp.IMDATA['injection_id'] must be modified in the module directly prior to running code
- BMXP modules are standalone; the pipeline ability to run all steps is not yet available — the workflow must be orchestrated manually

## Evidence

- [abstract] graph-based approach for n>2 datasets: [evidence span withheld — anchor status: not_found]
- [methods] MSAligner import for MS alignment: "from bxmp.eclipse import MSAligner"
- [methods] Blueshift and Eclipse require merged metadata: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] Eclipse purpose statement: "Align two or more same-method nontargeted LCMS datasets"
- [readme] shared schema headers for features and injections: "Feature Abundances - Pivot table of Feature x Injection (`Compound_ID` x `injection_id`) containing the abundances."
