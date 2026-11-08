---
name: tandem-ms-evaluation-set-bookkeeping
description: Use when reproducing or extending benchmark comparisons of tandem MS
  annotation tools (CycloBranch, NRPro, Dereplicator, Dereplicator+) on matched CID/HCD
  spectrum sets, where per-condition accuracy deltas and exact recomputation checks
  are needed.
when_to_use_negative:
- The goal is single-spectrum dereplication or de novo NRP analysis rather than cross-tool
  benchmark bookkeeping on an evaluation set
- Input spectra have no matched CID/HCD pair structure or no per-tool accuracy values
  to reconcile across conditions
- Custom building block databases must be created in NRPro — the article notes NRPro
  users cannot create custom databases of building blocks and NRP sequences
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CycloBranch
  role: Annotation engine that produces the per-hit peak annotation counts and weighted
    ratios of matched peak intensities recorded in the evaluation tables
- name: MassSpecBlocks
  role: Builds the custom NRP/PK sequence and building block databases (exported as
    plain text) that CycloBranch uses to annotate the CID/HCD evaluation spectra
  repo: github.com/privrja/MassSpecBlocks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:e5dfb0dacf2f07189903eaafccd466092ee882dc989c54689d97b98f8f63d64e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CycloBranch
  - MassSpecBlocks
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# tandem-ms-evaluation-set-bookkeeping

## Summary

A bookkeeping skill for curating and validating evaluation sets of high-resolution tandem MS/MS spectra of NRPs and PK siderophores, tracking annotation/comparison accuracy per tool per fragmentation condition (CID vs HCD), and assembling reproducible comparison tables.

## When to use

Use when reproducing or extending benchmark comparisons of tandem MS annotation tools (CycloBranch, NRPro, Dereplicator, Dereplicator+) on matched CID/HCD spectrum sets, where per-condition accuracy deltas and exact recomputation checks are needed.

## When NOT to use

- The goal is single-spectrum dereplication or de novo NRP analysis rather than cross-tool benchmark bookkeeping on an evaluation set
- Input spectra have no matched CID/HCD pair structure or no per-tool accuracy values to reconcile across conditions
- Custom building block databases must be created in NRPro — the article notes NRPro users cannot create custom databases of building blocks and NRP sequences

## Inputs

- Reproduced per-condition accuracy tables (Annotator and Comparator accuracies) for the 44-spectrum CID set and 44-spectrum HCD set
- High-resolution tandem mass spectra of NRPs and PK siderophores (e.g., pseudacyclin A, roseotoxin A, desferrioxamine B) annotated with CycloBranch/MassSpecBlocks-built databases
- Tool configuration records: 5 ppm m/z tolerances (CycloBranch, NRPro) vs 0.005 Da (Dereplicator, Dereplicator+), minimum relative peak intensity 0

## Outputs

- Tidy pandas dataframe of accuracy values per tool and per fragmentation condition
- cid_hcd_accuracy_comparison.csv with columns: tool, CID accuracy, HCD accuracy, absolute difference, relative difference
- Validation report confirming all recomputed accuracy cells and deltas match the reproduced tables exactly

## How to apply

Load the reproduced per-condition accuracy tables reporting Annotator and Comparator accuracies on the 44-spectrum CID set and the 44-spectrum HCD set, annotated with CycloBranch using MassSpecBlocks-built databases at 5 ppm precursor and fragment ion m/z error tolerances and a minimum relative peak intensity threshold of zero (with only b-ions and a-ions and at most one neutral loss of H2O or NH3 to unify configuration with NRPro; Dereplicator and Dereplicator+ use 0.005 Da). Parse accuracy values per tool and per fragmentation condition into a tidy pandas dataframe. Compute per-tool deltas: absolute difference (CID accuracy minus HCD accuracy) and relative difference ((CID − HCD)/HCD). Assemble a comparison table with columns tool, CID accuracy, HCD accuracy, absolute difference, and relative difference, and write it to cid_hcd_accuracy_comparison.csv. Accept the table only if every recomputed accuracy cell and delta matches the reproduced source tables exactly.

## Related tools

- **CycloBranch** (Annotation engine that produces the per-hit peak annotation counts and weighted ratios of matched peak intensities recorded in the evaluation tables)
- **MassSpecBlocks** (Builds the custom NRP/PK sequence and building block databases (exported as plain text) that CycloBranch uses to annotate the CID/HCD evaluation spectra) — github.com/privrja/MassSpecBlocks

## Examples

```
import pandas as pd; df = pd.concat([pd.read_csv('cid_acc.csv').assign(cond='CID'), pd.read_csv('hcd_acc.csv').assign(cond='HCD')]); df['abs_diff']=df['cid_acc']-df['hcd_acc']; df.to_csv('cid_hcd_accuracy_comparison.csv', index=False)
```

## Evaluation signals

- Every recomputed accuracy cell and CID−HCD delta matches the reproduced per-condition tables exactly; the comparison table is rejected otherwise
- Comparison CSV contains exactly the columns tool, CID accuracy, HCD accuracy, absolute difference, and relative difference
- Tolerance settings are recorded and consistent: 5 ppm for CycloBranch/NRPro, 0.005 Da for Dereplicator/Dereplicator+, minimum relative peak intensity threshold of zero
- Peak-annotation sanity anchors hold (e.g., CycloBranch 26/29 peaks for pseudacyclin A at weighted ratio 99%; 21/25 for roseotoxin A at 97%)
- Matched-hit records include the weighted ratio of matched peak intensities as reported by CycloBranch

## Limitations

- Cross-tool accuracy comparisons are not strictly uniform: Dereplicator and Dereplicator+ use 0.005 Da tolerances while CycloBranch and NRPro use 5 ppm
- A minimum relative peak intensity threshold of zero inflates the count of experimental peaks to be annotated
- In some MassSpecBlocks decompositions the peptide bond connecting Ac to Ile on the branch was not automatically marked because the resulting block Ac would contain only three nonhydrogen atoms, so generated sequences may need manual review before being exported to CycloBranch

## Evidence

- [results] Tolerances recorded per tool: "The precursor and fragment ion m/z error tolerances were set up to 5ppm for CycloBranch and NRPro. The values used in Dereplicator and Dereplicator+ were 0.005 Da."
- [results] Intensity threshold setting: "we set up the minimum threshold of relative peak intensity to zero."
- [results] Ion-type configuration: "we were looking only for b-ions, a-ions and allowed only one neutral loss of H2O or NH3."
- [abstract] Database construction pipeline: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [readme] Tool scope from README: "MassSpecBlocks: Database of Sequences and Building Blocks of Microbial Metabolites for Mass Spectra Analysis is an open-source web application"
