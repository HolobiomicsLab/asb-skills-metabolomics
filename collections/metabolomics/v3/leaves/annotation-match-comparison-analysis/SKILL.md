---
name: annotation-match-comparison-analysis
description: Use when two or more nontargeted LC-MS datasets (feature tables with
  annotations) have been independently processed and you must quantify how many annotated
  features two aligners recover identically and how many spurious mismatches each
  introduces.
when_to_use_negative:
- Datasets were not independently processed or are fewer than two — there is nothing
  to align or compare
- Features lack Annotation_IDs (Method-unique annotation labels), since cross-referencing
  matched annotations against the reference depends on them
- You need drift correction or redundant-feature clustering (Blueshift/Gravity tasks),
  not alignment-quality benchmarking
tools:
- name: metabCombiner
  role: Candidate aligner run on DS1–DS4 in intersection mode to compare against the
    Eclipse reference alignment
- name: Eclipse
  role: Reference aligner whose clique-mode MSAligner results define the benchmark
    annotation set the candidate tool is scored against
  repo: github.com/broadinstitute/bmxp
- name: bmxp
  role: Provides the shared schema (Feature Metadata indexed on Compound_ID, Injection
    Metadata on injection_id) that input tables must conform to; installable via pip
    install bmxp
  repo: github.com/broadinstitute/bmxp
- name: Python
  role: Implementation and scripting environment for loading tables, running aligners,
    cross-referencing Annotation_IDs, and writing the comparison CSV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:c98c03cd7a89819824de742aa35068801083ddf81a9388c83a8173fc27f3fa4c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Eclipse
  - Python
  - bmxp
  - metabCombiner
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# annotation-match-comparison-analysis

## Summary

Benchmark an LC-MS dataset-alignment tool against a reference aligner (here metabCombiner vs Eclipse clique-mode) by cross-referencing recovered Annotation_IDs and tallying overlapping annotations matched versus spurious annotation-mismatch hits, including order-dependence checks for multi-dataset combination sequences.

## When to use

Use when two or more nontargeted LC-MS datasets (feature tables with annotations) have been independently processed and you must quantify how many annotated features two aligners recover identically and how many spurious mismatches each introduces.

## When NOT to use

- Datasets were not independently processed or are fewer than two — there is nothing to align or compare
- Features lack Annotation_IDs (Method-unique annotation labels), since cross-referencing matched annotations against the reference depends on them
- You need drift correction or redundant-feature clustering (Blueshift/Gravity tasks), not alignment-quality benchmarking

## Inputs

- Feature tables for independently processed LC-MS datasets (DS1–DS4 human plasma), with Feature Metadata (Compound_ID, RT, MZ, Annotation_ID) and Feature Abundances
- Reference alignment results from Eclipse MSAligner clique mode over the same datasets
- Alignment scripts (Supplementary Code S5/S6) specifying the metabCombiner four-dataset combination sequence

## Outputs

- metabcombiner_vs_eclipse_comparison.csv with per-tool matched-annotation counts, spurious annotation-mismatch hit counts, and order-dependence outcomes
- Tally of overlapping annotations recovered relative to the reference (target: 362 of 381)
- Order-dependence report from permuted dataset combination runs

## How to apply

Load the feature tables (e.g. DS1–DS4) and conform headers to the bmxp shared schema — Feature Metadata indexed on Compound_ID, Injection Metadata on injection_id — merging Feature Metadata with Feature Abundances since Eclipse requires this merged form. Run the candidate tool (metabCombiner) over the dataset sequence in intersection mode, then run or load the reference aligner's results (Eclipse MSAligner clique mode, imported via bmxp.eclipse). Cross-reference the candidate's aligned features against the reference via Annotation_ID, tallying overlapping annotations recovered and annotation-mismatch (spurious) hits. Re-run the combination with permuted dataset orders to expose order dependence, since metabCombiner results varied with dataset sequence while Eclipse's graph-based approach targets complex n > 2 matching. Write per-tool match counts, spurious-hit counts, and order-dependence outcomes to a CSV such as metabcombiner_vs_eclipse_comparison.csv, and validate against the published benchmark values (362 of 381 overlapping annotations matched; 26 spurious hits for metabCombiner vs 3 for Eclipse).

## Related tools

- **metabCombiner** (Candidate aligner run on DS1–DS4 in intersection mode to compare against the Eclipse reference alignment)
- **Eclipse** (Reference aligner whose clique-mode MSAligner results define the benchmark annotation set the candidate tool is scored against) — github.com/broadinstitute/bmxp
- **bmxp** (Provides the shared schema (Feature Metadata indexed on Compound_ID, Injection Metadata on injection_id) that input tables must conform to; installable via pip install bmxp) — github.com/broadinstitute/bmxp
- **Python** (Implementation and scripting environment for loading tables, running aligners, cross-referencing Annotation_IDs, and writing the comparison CSV)

## Examples

```
import bmxp; from bmxp.eclipse import MSAligner
```

## Evaluation signals

- Candidate matches 362 of 381 overlapping annotations — parity with Eclipse's clique-mode recovery
- Spurious annotation-mismatch hits reproduce published counts: metabCombiner 26 vs Eclipse 3
- Permuted dataset-order runs of metabCombiner yield different results, confirming the reported order dependence
- Output CSV contains per-tool match counts, spurious mismatch-hit counts, and order-dependence outcomes for all dataset combinations
- Input feature tables validate against bmxp.FMDATA/bmxp.IMDATA expected column headers (Compound_ID, injection_id, Annotation_ID)

## Limitations

- metabCombiner results depend on the order in which datasets are combined, so a single run understates or overstates its error rate
- metabCombiner produced 26 spurious annotation-mismatch hits versus Eclipse's 3 on DS1–DS4 in intersection mode
- Existing alignment software 'does not fully meet our needs' for n > 2 datasets — comparisons are only meaningful on the same aligned annotation space

## Evidence

- [abstract] Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets.: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [intro] Eclipse: align two or more same-method nontargeted LCMS datasets: "Eclipse: align two or more same-method nontargeted LCMS datasets"
- [methods] Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances.: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] All BMXP modules use a shared schema and file formats with our prefered columns headers.: "All BMXP modules use a shared schema and file formats with our prefered columns headers."
