---
name: spectra-annotation-benchmark-tabulation
description: Use when benchmarking how many experimental peaks each tool annotates
  in high-resolution MS/MS spectra of NRPs or PK siderophores, or when comparing annotator
  accuracy across CID and HCD fragmentation conditions.
when_to_use_negative:
- The goal is de novo sequencing rather than benchmarking how many experimental peaks
  known-compound annotators match
- Input spectra are low-resolution MS/MS where ppm-level m/z error tolerances (5 ppm)
  are not meaningful
- Compounds lack definable building-block sequences and cannot be represented as NRP/PK
  building-block databases for CycloBranch
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: CycloBranch
  role: Primary annotator and comparator; annotated 26/29 peaks of pseudacyclin A
    (99% weighted ratio) and 21/25 of roseotoxin A (97%), reporting weighted ratio
    of matched peak intensities per hit
  repo: https://ms.biomed.cas.cz/cyclobranch/
- name: MassSpecBlocks
  role: Builds custom NRP/PK sequence and building-block databases used to annotate
    spectra in CycloBranch during the benchmark
  repo: github.com/privrja/MassSpecBlocks
- name: NRPro
  role: Comparison annotator run at 5 ppm tolerance; annotated 23 of 29 pseudacyclin
    A peaks and 22 of 25 roseotoxin A peaks
- name: Dereplicator
  role: Comparison annotator run at 0.005 Da tolerance; annotated 6 of 29 pseudacyclin
    A peaks and 8 of 25 roseotoxin A peaks
- name: Dereplicator+
  role: Comparison annotator run at 0.005 Da tolerance; annotated 14 of 29 pseudacyclin
    A peaks and 19 of 25 roseotoxin A peaks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:b4208a5a2600ec02e3a553bbeb267ced013c6ae8586fac0b322ce4ab60a66a3a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CycloBranch
  - Dereplicator
  - Dereplicator+
  - MassSpecBlocks
  - NRPro
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# spectra-annotation-benchmark-tabulation

## Summary

Tabulates head-to-head annotation-accuracy comparisons of tandem-MS dereplication tools (CycloBranch, NRPro, Dereplicator, Dereplicator+) across fragmentation conditions (CID vs HCD), computing absolute and relative accuracy deltas per tool into a tidy CSV comparison table with exact-recompute validation.

## When to use

Use when benchmarking how many experimental peaks each tool annotates in high-resolution MS/MS spectra of NRPs or PK siderophores, or when comparing annotator accuracy across CID and HCD fragmentation conditions.

## When NOT to use

- The goal is de novo sequencing rather than benchmarking how many experimental peaks known-compound annotators match
- Input spectra are low-resolution MS/MS where ppm-level m/z error tolerances (5 ppm) are not meaningful
- Compounds lack definable building-block sequences and cannot be represented as NRP/PK building-block databases for CycloBranch

## Inputs

- Reproduced per-condition accuracy tables (Annotator and Comparator accuracies) for the 44-spectrum CID set and the 44-spectrum HCD set
- High-resolution tandem mass spectra of NRPs and PK siderophores (e.g., pseudacyclin A, roseotoxin A)
- Theoretical spectra from databases built with MassSpecBlocks and exported to CycloBranch plain text format

## Outputs

- cid_hcd_accuracy_comparison.csv with columns: tool, CID accuracy, HCD accuracy, absolute difference, relative difference
- Per-tool annotated-peak counts and weighted ratios of matched peak intensities per compound hit

## How to apply

Run each annotator on the per-condition spectrum sets with unified parameters: 5 ppm precursor and fragment ion m/z error tolerances for CycloBranch and NRPro (0.005 Da for Dereplicator and Dereplicator+), a minimum relative peak intensity threshold of zero, and only b-ions and a-ions with one neutral loss of H2O or NH3 to unify configuration with NRPro. Load the reproduced per-condition accuracy tables (44-spectrum CID set, 44-spectrum HCD set) where annotation databases were built with CycloBranch/MassSpecBlocks. Parse accuracy values per tool (Annotator, Comparator) and per fragmentation condition into a tidy pandas dataframe. Compute per-tool deltas: absolute difference (CID minus HCD) and relative difference ((CID − HCD)/HCD), then assemble a table with columns tool, CID accuracy, HCD accuracy, absolute difference, and relative difference. Validation: recompute every accuracy cell and delta from the source per-condition tables; accept the comparison table only if all recomputed values match exactly.

## Related tools

- **CycloBranch** (Primary annotator and comparator; annotated 26/29 peaks of pseudacyclin A (99% weighted ratio) and 21/25 of roseotoxin A (97%), reporting weighted ratio of matched peak intensities per hit) — https://ms.biomed.cas.cz/cyclobranch/
- **MassSpecBlocks** (Builds custom NRP/PK sequence and building-block databases used to annotate spectra in CycloBranch during the benchmark) — github.com/privrja/MassSpecBlocks
- **NRPro** (Comparison annotator run at 5 ppm tolerance; annotated 23 of 29 pseudacyclin A peaks and 22 of 25 roseotoxin A peaks)
- **Dereplicator** (Comparison annotator run at 0.005 Da tolerance; annotated 6 of 29 pseudacyclin A peaks and 8 of 25 roseotoxin A peaks)
- **Dereplicator+** (Comparison annotator run at 0.005 Da tolerance; annotated 14 of 29 pseudacyclin A peaks and 19 of 25 roseotoxin A peaks)

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('cid_hcd_accuracy_table.csv'); df['abs_diff']=df.CID-df.HCD; df['rel_diff']=(df.CID-df.HCD)/df.HCD; df.to_csv('cid_hcd_accuracy_comparison.csv', index=False)"
```

## Evaluation signals

- Recomputed accuracy cells and CID-minus-HCD deltas from the source per-condition tables must match the comparison table exactly before it is accepted
- Annotated-peak counts must not exceed the total number of experimental peaks per compound (e.g., ≤29 for pseudacyclin A, ≤25 for roseotoxin A)
- Weighted ratios of matched peak intensities should fall in [0, 1] (e.g., 99% for pseudacyclin A and 97% for roseotoxin A under CycloBranch)
- Comparison CSV must contain exactly the columns tool, CID accuracy, HCD accuracy, absolute difference, relative difference for all four tools
- Absolute difference must equal CID accuracy minus HCD accuracy and relative difference must equal (CID − HCD)/HCD for each tool row

## Limitations

- Cross-tool comparability required unifying configuration (only b-ions and a-ions, one neutral loss of H2O or NH3, minimum relative peak intensity threshold of zero); without this unification, counts are not comparable
- PK siderophores contain uncommon building blocks that reverse the orientation of peptide bonds, complicating mass spectra interpretation and potentially depressing annotation counts
- NRPro users cannot create custom databases of building blocks and NRP sequences, constraining which compounds can be benchmarked with it

## Evidence

- [results] Unified search tolerances across tools: "The precursor and fragment ion m/z error tolerances were set up to 5ppm for CycloBranch and NRPro. The values used in Dereplicator and Dereplicator+ were 0.005 Da."
- [results] Peak intensity threshold for benchmarking: "we set up the minimum threshold of relative peak intensity to zero."
- [results] Ion series restriction for fair comparison: "we were looking only for b-ions, a-ions and allowed only one neutral loss of H2O or NH3."
- [abstract] Database construction path feeding the benchmark: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [readme] Tool ecosystem roles from README: [evidence span withheld — anchor status: not_found]
