---
name: algorithm-order-invariance-testing
description: Use when aligning two or more independently processed nontargeted LC-MS
  datasets with Eclipse's MSAligner and you need to verify that the graph-based feature
  grouping result does not depend on dataset insertion order.
when_to_use_negative:
- The datasets have already been merged into a single complete dataset by an upstream
  tool (e.g., Gravity clustering or Formation formatting), so no pairwise re-alignment
  is being tested
- You are evaluating a non-Eclipse aligner whose algorithm makes no order-independence
  claim — the invariant comparison is only meaningful if the tool states it is order-independent
edam_topics:
- http://edamontology.org/topic_3383
tools:
- name: Eclipse
  role: The MSAligner module whose graph-based alignment and clustering output is
    re-run under different dataset insertion orders to test the order-independence
    invariant
  repo: https://github.com/broadinstitute/bmxp/blob/main/bmxp/eclipse/readme.md
- name: bmxp
  role: Provides the shared schema (Compound_ID, injection_id) and the Python package
    via which MSAligner is invoked
  repo: https://github.com/broadinstitute/bmxp
- name: Python
  role: Scripting language used to orchestrate the two alignment runs and diff the
    resulting eclipse_feature_groups.csv outputs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:ec62e0d5e7b58aef6e886dd13a1c82fa5395f8f242ed2a22bbc3baadbd95b886
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# algorithm-order-invariance-testing

## Summary

A validation technique for Eclipse's graph-based LC-MS dataset alignment in which the alignment is re-run with datasets inserted in a different order and the resulting feature group assignments are compared, verifying the tool's stated order-independence invariant for n > 2 dataset alignment.

## When to use

Use when aligning two or more independently processed nontargeted LC-MS datasets with Eclipse's MSAligner and you need to verify that the graph-based feature grouping result does not depend on dataset insertion order.

## When NOT to use

- The datasets have already been merged into a single complete dataset by an upstream tool (e.g., Gravity clustering or Formation formatting), so no pairwise re-alignment is being tested
- You are evaluating a non-Eclipse aligner whose algorithm makes no order-independence claim — the invariant comparison is only meaningful if the tool states it is order-independent

## Inputs

- Merged Feature Metadata + Feature Abundances table (BMXP shared schema: Compound_ID x injection_id)
- Pairwise subalignment match tables loaded into the Eclipse MSAligner directed graph
- Eclipse configuration parameters: minimum group size, minimum clique size, and diameter setting (1, 2, or 3)

## Outputs

- Two eclipse_feature_groups.csv outputs, one per dataset insertion order, keyed by feature ID per the shared schema
- A pass/fail comparison confirming identical feature group assignments across insertion orders

## How to apply

After running the Eclipse MSAligner workflow — loading the merged Feature Metadata + Feature Abundances table under the shared BMXP schema (Compound_ID as feature index, injection_id as injection index), building the directed graph where features are nodes, matches are edges and match penalties are edge weights, retaining only bidirectional edges with summed penalties, and running the iterative group-ranking clustering with the configured minimum group size, minimum clique size, and diameter setting (1, 2, or 3) — rerun the entire alignment with a different dataset insertion order. Then compare the two outputs keyed by feature ID and confirm the clustered feature group assignments are identical. Because the invariant is only meaningful once the full clustering pipeline has executed, this test doubles as an end-to-end check of the graph construction and group-ranking stages.

## Related tools

- **Eclipse** (The MSAligner module whose graph-based alignment and clustering output is re-run under different dataset insertion orders to test the order-independence invariant) — https://github.com/broadinstitute/bmxp/blob/main/bmxp/eclipse/readme.md
- **bmxp** (Provides the shared schema (Compound_ID, injection_id) and the Python package via which MSAligner is invoked) — https://github.com/broadinstitute/bmxp
- **Python** (Scripting language used to orchestrate the two alignment runs and diff the resulting eclipse_feature_groups.csv outputs)

## Examples

```
from bmxp.eclipse import MSAligner; aligner = MSAligner(features, matches); aligner.align()
```

## Evaluation signals

- Feature group assignments in eclipse_feature_groups.csv are byte-identical across runs with different dataset insertion orders
- Every feature ID (Compound_ID) in the output maps to the same group in both runs
- Only bidirectional edges (matches confirmed in both directions between datasets) appear in the graph, with penalties summed from both directions
- The configured clustering parameters (minimum group size, minimum clique size, diameter 1/2/3) are identical between the two runs so the comparison isolates insertion order
- Output column headers match the BMXP shared schema (Compound_ID, injection_id) or the globally overridden schema names (e.g., Feature_ID, Filename)

## Limitations

- The invariant only holds if the full graph construction (bidirectional edge retention with summed penalties) and iterative group-ranking clustering run identically in both passes; any configuration drift between runs invalidates the comparison
- Eclipse is designed for same-method nontargeted LC-MS datasets; order-invariance testing on mixed methods or mixed targeted/nontargeted data is outside the tool's stated scope
- XIC shape-based feature clustering is not yet implemented in Gravity, so the clustered groups being validated rely on RT and correlation only

## Evidence

- [abstract] Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets.: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [methods] from bxmp.eclipse import MSAligner: [evidence span withheld — anchor status: not_found]
- [readme] Eclipse - Align two or more same-method nontargeted LCMS datasets: "Eclipse - Align two or more same-method nontargeted LCMS datasets"
- [readme] Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances.: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
