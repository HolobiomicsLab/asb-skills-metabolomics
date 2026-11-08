---
name: alignment-order-dependence-assessment
description: Use when combining three or more independently processed LC-MS feature
  tables with a sequential alignment tool and you must verify that matched annotations
  are order-invariant and that spurious annotation mismatches are quantified against
  a reference alignment.
when_to_use_negative:
- You are aligning only two datasets, where sequential-order effects of the kind seen
  for n > 2 dataset combinations do not arise
- Your features lack a stable cross-dataset identifier such as Annotation_ID, making
  cross-referencing against a reference alignment impossible
- The aligner under test uses a graph-based formulation that is inherently order-invariant
  (e.g., Eclipse's graph-based approach for n > 2 datasets), so an order-permutation
  assessment adds no information
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: metabCombiner
  role: Sequential multi-dataset aligner under test, run in intersection mode through
    the DS1–DS4 combination sequence and re-run under permuted dataset orders to expose
    order dependence
- name: Eclipse
  role: Reference order-invariant alignment via Eclipse MSAligner clique mode; its
    graph-based approach handles complex matching scenarios for n > 2 datasets and
    supplies the 381-annotation reference set
  repo: https://github.com/broadinstitute/bmxp
- name: bmxp
  role: Provides the shared schema (bmxp.FMDATA, bmxp.IMDATA) and merged Feature Metadata
    + Feature Abundances format required by the alignment modules
  repo: https://github.com/broadinstitute/bmxp
- name: Python
  role: Implementation language for the comparison workflow and CSV output generation
  repo: https://github.com/broadinstitute/bmxp
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:2966b5fc769633c9a9f6cfb9307df369ba1765a0f82200bd7b9474835bdc449b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/broadinstitute/bmxp
  tools:
  - Eclipse
  - Python
  - bmxp
  - metabCombiner
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# alignment-order-dependence-assessment

## Summary

Assesses whether multi-dataset LC-MS alignment results change when the order in which datasets are combined is permuted, and benchmarks the alignment against a reference tool by counting overlapping annotations matched and spurious annotation-mismatch hits (e.g., metabCombiner 26 vs Eclipse 3 on DS1–4).

## When to use

Use when combining three or more independently processed LC-MS feature tables with a sequential alignment tool and you must verify that matched annotations are order-invariant and that spurious annotation mismatches are quantified against a reference alignment.

## When NOT to use

- You are aligning only two datasets, where sequential-order effects of the kind seen for n > 2 dataset combinations do not arise
- Your features lack a stable cross-dataset identifier such as Annotation_ID, making cross-referencing against a reference alignment impossible
- The aligner under test uses a graph-based formulation that is inherently order-invariant (e.g., Eclipse's graph-based approach for n > 2 datasets), so an order-permutation assessment adds no information

## Inputs

- Feature tables for the datasets to align (DS1–DS4 human plasma), as bmxp shared-schema files (Feature Metadata indexed on Compound_ID plus Feature Abundances pivot of Compound_ID x injection_id)
- Reference alignment results from an order-invariant aligner (Eclipse MSAligner clique mode), keyed by Annotation_ID
- Aligner configuration scripts (Supplementary Code S5/S6 for metabCombiner, intersection mode)

## Outputs

- metabcombiner_vs_eclipse_comparison.csv with per-tool match counts, spurious annotation-mismatch hit counts, and order-dependence outcomes
- Tally of overlapping annotations recovered out of the reference total (e.g., 362 of 381) and count of spurious annotation-mismatch hits per tool
- Permutation results showing whether alignment output varies with dataset combination order

## How to apply

Conform all feature tables to the bmxp shared schema (Feature Metadata indexed on Compound_ID, Injection Metadata on injection_id), merging Feature Metadata with Feature Abundances since Eclipse and Blueshift require the merged form. Run the sequential aligner (e.g., metabCombiner per Supplementary Code S5/S6) through the full dataset sequence — here DS1–DS4 in intersection mode — then cross-reference its aligned features against a reference alignment (Eclipse MSAligner clique mode) via Annotation_ID, tallying overlapping annotations recovered (362 of 381) and spurious annotation-mismatch hits (26 vs 3 for Eclipse). Finally re-run the combination with permuted dataset orders and record whether match counts and pairings shift; order dependence is demonstrated when permuted runs yield different results. Write per-tool match counts, spurious-hit counts, and order-dependence outcomes to a CSV such as metabcombiner_vs_eclipse_comparison.csv.

## Related tools

- **metabCombiner** (Sequential multi-dataset aligner under test, run in intersection mode through the DS1–DS4 combination sequence and re-run under permuted dataset orders to expose order dependence)
- **Eclipse** (Reference order-invariant alignment via Eclipse MSAligner clique mode; its graph-based approach handles complex matching scenarios for n > 2 datasets and supplies the 381-annotation reference set) — https://github.com/broadinstitute/bmxp
- **bmxp** (Provides the shared schema (bmxp.FMDATA, bmxp.IMDATA) and merged Feature Metadata + Feature Abundances format required by the alignment modules) — https://github.com/broadinstitute/bmxp
- **Python** (Implementation language for the comparison workflow and CSV output generation) — https://github.com/broadinstitute/bmxp

## Examples

```
import bmxp; from bmxp.eclipse import MSAligner; aligner = MSAligner(); results = aligner.align([ds1, ds2, ds3, ds4], mode='clique')
```

## Evaluation signals

- metabCombiner recovers 362 of 381 reference annotations on DS1–DS4, matching Eclipse clique mode's overlap count (parity on matched annotations)
- Spurious annotation-mismatch hits for metabCombiner total 26 versus Eclipse's 3 on the same datasets
- Re-running the metabCombiner combination with permuted dataset orders yields different results, demonstrating order dependence (whereas Eclipse's results should be order-invariant)
- All aligned features cross-reference successfully against the Eclipse reference via Annotation_ID, confirming schema conformance of Compound_ID/injection_id headers
- Output CSV metabcombiner_vs_eclipse_comparison.csv contains per-tool match counts, spurious-hit counts, and order-dependence outcomes for every dataset permutation tested

## Limitations

- metabCombiner results depend on the order in which datasets are combined, so a single sequential run is not reproducible without fixing the dataset order
- metabCombiner produces many more spurious annotation-mismatch hits than Eclipse (26 vs 3) at equal overlap, inflating false matches
- bmxp modules have no published version numbers, complicating exact reproduction of the reference alignment

## Evidence

- [results] metabCombiner identified the same number of overlapping annotations as Eclipse's clique mode (362 of 381) but produced many more spurious hits (26 vs 3), and its results depended on dataset order.: [evidence span withheld — anchor status: not_found]
- [abstract] Eclipse uses a novel graph-based approach to handle complex matching scenarios arising from n > 2 datasets: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [readme] Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances.: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [intro] Eclipse: align two or more same-method nontargeted LCMS datasets: "Align two or more same-method nontargeted LCMS datasets"
- [supplementary] Cross-reference metabCombiner's aligned features against the Eclipse reference via Annotation_ID: [evidence span withheld — anchor status: not_found]
