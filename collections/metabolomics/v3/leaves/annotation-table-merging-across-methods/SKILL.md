---
name: annotation-table-merging-across-methods
description: Use when two annotation methods (e.g. SMART 2.0 and atomic sort) have
  been run on the same compounds and you need a side-by-side concordance table with
  explicit difference notes per mismatched call.
when_to_use_negative:
- Only one annotation method was applied and there is no second method's calls to
  reconcile against
- The compounds lack a shared identifier key, so SMART 2.0 and atomic sort rows cannot
  be matched compound-by-compound
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: SMART 2.0
  role: NMR-based machine learning annotation tool whose calls form one side of the
    comparison table
- name: atomic sort
  role: The recently appearing comparator method whose assignments form the other
    side of the comparison table
- name: NP Classifier
  role: Programmatic chemical-class classifier (deployed via docker/docker-compose,
    /classify?smiles=<>) that can supply class assignments consistent with the annotation
    fields being compared
  repo: https://github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:21d2fb8139d0cb6ca4e2a406e881d1c48058ae278990f0d0d9fba79f31319049
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - NP Classifier
  - SMART 2.0
  - atomic sort
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Annotation Table Merging Across Methods

## Summary

A tabular reconciliation skill for comparing chemical-class annotation calls from two annotation approaches — here SMART 2.0 (NMR-based CNN) and the 'atomic sort' method — on the same natural product set (swinholide A, samholides A–I, symplocolide A), producing a per-compound agreement/difference table.

## When to use

Use when two annotation methods (e.g. SMART 2.0 and atomic sort) have been run on the same compounds and you need a side-by-side concordance table with explicit difference notes per mismatched call.

## When NOT to use

- Only one annotation method was applied and there is no second method's calls to reconcile against
- The compounds lack a shared identifier key, so SMART 2.0 and atomic sort rows cannot be matched compound-by-compound

## Inputs

- SMART 2.0 annotation results for the cyclic-peptide/macrolide compounds (from SI tables or deposited annotation files)
- atomic sort method assignments for the same compounds (from the published reference/deposit)
- Compound identifier list covering swinholide A, samholides A–I, symplocolide A and related derivatives

## Outputs

- comparison_table.tsv with columns compound_id, SMART_2.0_annotation, atomic_sort_annotation, agreement_flag, difference_notes
- Agreement count summary (number/fraction of concordant annotation calls)

## How to apply

Retrieve the SMART 2.0 annotation results for the analyzed compounds (swinholide A, samholides A–I, symplocolide A and related derivatives) from the article's SI tables or deposited annotation files, and the corresponding atomic sort assignments from its published reference. Load both sets into pandas DataFrames keyed on compound identifier. For each compound compute a boolean agreement flag between the SMART 2.0 and atomic sort calls, and for every disagreement record the exact differing class/assignment fields in a difference_notes column. Assemble a table with columns compound_id, SMART_2.0_annotation, atomic_sort_annotation, agreement_flag, difference_notes, write it as comparison_table.tsv, and summarize agreement counts to quantify concordance between the two methods.

## Related tools

- **SMART 2.0** (NMR-based machine learning annotation tool whose calls form one side of the comparison table)
- **atomic sort** (The recently appearing comparator method whose assignments form the other side of the comparison table)
- **NP Classifier** (Programmatic chemical-class classifier (deployed via docker/docker-compose, /classify?smiles=<>) that can supply class assignments consistent with the annotation fields being compared) — https://github.com/mwang87/NP-Classifier

## Examples

```
sm = pd.read_csv('smart2_annotations.tsv', sep='\t'); at = pd.read_csv('atomic_sort_annotations.tsv', sep='\t'); m = sm.merge(at, on='compound_id'); m['agreement_flag'] = m['SMART_2.0_annotation'] == m['atomic_sort_annotation']; m.to_csv('comparison_table.tsv', sep='\t', index=False)
```

## Evaluation signals

- comparison_table.tsv exists and parses as well-formed TSV
- The table contains exactly one row per analyzed compound
- Every row with agreement_flag=False carries a non-empty difference_notes entry specifying the differing class/assignment fields
- Agreement counts are reported and assessed qualitatively against the paper's framing of SMART 2.0 and atomic sort as alternative approaches for characterizing structurally novel cyclic peptides

## Limitations

- Concordance is qualitative: the paper presents SMART 2.0 and atomic sort as alternative approaches, so agreement flags summarize but do not adjudicate which call is correct
- Comparison fidelity depends on the deposited/SI annotation files covering the same compound set (swinholide A, samholides A–I, symplocolide A derivatives) under matching identifiers

## Evidence

- [abstract] comparison against atomic sort on cyclic peptides: [evidence span withheld — anchor status: not_found]
- [abstract] compound scope of the comparison: "as well as the annotation of swinholide A, samholides A–I, and several new derivatives"
- [readme] programmatic classification endpoint for class fields: "Classify programmatically 

```/classify?smiles=<>```

You can also provide cached flag to the params to get the cached version so make it faster"
