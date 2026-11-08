---
name: metabcombiner-intersection-mode-alignment
description: Use when evaluating LC-MS multi-dataset alignment tools — i.e., when
  feature tables from independently processed same-method datasets must be aligned
  and compared on overlapping Annotation_IDs and spurious matches.
when_to_use_negative:
- The datasets were processed by different LC-MS methods — Eclipse is scoped to aligning
  'same-method nontargeted LCMS datasets', and the comparison is not meaningful otherwise.
- The input is raw instrument data (.raw or .mzML) rather than processed feature tables
  — read those with Chroma first.
- The goal is drift correction or redundant-feature clustering rather than cross-dataset
  feature/annotation alignment (use Blueshift or Gravity instead).
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: metabCombiner
  role: Alignment tool under test; run in intersection mode across DS1–DS4 per Supplementary
    Code S5/S6
- name: Eclipse
  role: Provides the reference clique-mode alignment (MSAligner) against which metabCombiner's
    matches and spurious hits are scored
  repo: https://github.com/broadinstitute/bmxp
- name: bmxp
  role: Shared schema (bmxp.FMDATA, bmxp.IMDATA) used to conform feature/injection
    column headers before alignment
  repo: github.com/broadinstitute/bmxp
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:cf465836f41757bf7bb61982bd1e9c1ed56d6e02278c2a5fc443a8b2c133e927
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/broadinstitute/bmxp
  tools:
  - Eclipse
  - Python
  - bmxp
  - metabCombiner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# metabcombiner-intersection-mode-alignment

## Summary

Benchmark metabCombiner's intersection-mode alignment against Eclipse MSAligner for aligning multiple nontargeted LC-MS metabolomics datasets (DS1–DS4), quantifying recovered overlapping annotations and spurious annotation-mismatch hits.

## When to use

Use when evaluating LC-MS multi-dataset alignment tools — i.e., when feature tables from independently processed same-method datasets must be aligned and compared on overlapping Annotation_IDs and spurious matches.

## When NOT to use

- The datasets were processed by different LC-MS methods — Eclipse is scoped to aligning 'same-method nontargeted LCMS datasets', and the comparison is not meaningful otherwise.
- The input is raw instrument data (.raw or .mzML) rather than processed feature tables — read those with Chroma first.
- The goal is drift correction or redundant-feature clustering rather than cross-dataset feature/annotation alignment (use Blueshift or Gravity instead).

## Inputs

- DS1–DS4 feature tables (human plasma, same LC-MS method) conformable to the bmxp shared schema (Feature Metadata + Feature Abundances, Injection Metadata)
- Eclipse MSAligner clique-mode alignment results for the same datasets (reference)
- Supplementary Code S5/S6 metabCombiner scripts

## Outputs

- metabcombiner_vs_eclipse_comparison.csv with per-tool match counts, spurious annotation-mismatch counts, and order-dependence outcomes

## How to apply

Load the DS1–DS4 feature tables and conform column headers to the bmxp shared schema (Feature Metadata indexed on Compound_ID, Injection Metadata on injection_id), merging Feature Metadata with Feature Abundances since some modules (Blueshift, Eclipse) require this merged form. Run metabCombiner on DS1–DS4 in intersection mode following Supplementary Code S5/S6, combining the datasets through a four-dataset sequence. Load Eclipse MSAligner clique-mode results for the same datasets as the reference alignment, then cross-reference metabCombiner's aligned features against the Eclipse reference via Annotation_ID, tallying overlapping annotations recovered and spurious annotation-mismatch hits. Re-run the metabCombiner combination with permuted dataset orders to expose its order dependence, since its graph-free sequential combination can yield different results depending on dataset order. Write the per-tool match counts, spurious mismatch counts, and order-dependence outcomes to metabcombiner_vs_eclipse_comparison.csv.

## Related tools

- **metabCombiner** (Alignment tool under test; run in intersection mode across DS1–DS4 per Supplementary Code S5/S6)
- **Eclipse** (Provides the reference clique-mode alignment (MSAligner) against which metabCombiner's matches and spurious hits are scored) — https://github.com/broadinstitute/bmxp
- **bmxp** (Shared schema (bmxp.FMDATA, bmxp.IMDATA) used to conform feature/injection column headers before alignment) — github.com/broadinstitute/bmxp

## Examples

```
import bmxp; bmxp.FMDATA['Compound_ID'] = 'Feature_ID'; bmxp.IMDATA['injection_id'] = 'Filename'
```

## Evaluation signals

- metabCombiner matches 362 of 381 overlapping annotations, achieving parity with Eclipse's clique mode on DS1–DS4 (Section 4.4 values).
- Spurious annotation-mismatch hits total 26 for metabCombiner versus 3 for Eclipse — metabCombiner must not be reported as matching Eclipse's specificity.
- Re-running metabCombiner with permuted dataset orders produces different alignment outcomes, demonstrating order dependence.
- Per-tool counts are cross-referenced via Annotation_ID (method-unique annotation label) and written to metabcombiner_vs_eclipse_comparison.csv.
- Feature tables carry conformant bmxp headers (Compound_ID, injection_id) and merged Feature Metadata + Feature Abundances.

## Limitations

- metabCombiner's intersection-mode results depend on the order in which the four datasets are combined (order dependence, shown by permutation).
- metabCombiner produces many more spurious annotation-mismatch hits than Eclipse's clique mode (26 vs 3) on DS1–DS4.
- metabCombiner handled the n > 2 dataset matching less gracefully than Eclipse's graph-based approach for complex multi-dataset scenarios.
- bmxp version numbers for the modules used as reference are not documented.

## Evidence

- [abstract] Eclipse uses a novel graph-based approach to handle complex matching scenarios arising from n > 2 datasets: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [intro] Eclipse aligns same-method nontargeted LCMS datasets: "Align two or more same-method nontargeted LCMS datasets"
- [methods] Some modules require merging Feature Metadata + Feature Abundances: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] Shared schema index defaults: "Feature Metadata `bmxp.FMDATA` - Describes the feature. Index default is `Compound_ID`"
