---
name: graph-parameter-perturbation-reanalysis
description: Use when existing match tables from n>2 LC-MS dataset alignment yield
  too few matched features under strict clustering, and you want to test relaxed diameter/clique-size
  settings without recomputing subalignments.
when_to_use_negative:
- When the per-pair subalignments have not yet been run — this skill reuses existing
  match tables and cannot create them
- When strict clique clustering is required for downstream analyses sensitive to spurious
  matches — relaxing parameters increased spurious matches from 3 to 13 in the DS1–4
  case
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Eclipse
  role: Graph-based clustering of feature-match edges built from existing subalignment
    match tables with relaxed parameters (diameter=2, min clique size=3)
  repo: https://github.com/broadinstitute/bmxp
- name: bmxp
  role: Provides the shared schema (FMDATA/IMDATA, Compound_ID, injection_id) and
    BMXP column format used for inputs and the reclustered output
  repo: https://github.com/broadinstitute/bmxp
- name: Python
  role: Language for loading match tables, building the feature-match graph, and running
    Eclipse clustering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:17ca19b2e6a7759399d9febbc0715afd276e9199efa4ee314aec9ca728257848
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/broadinstitute/bmxp
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# graph-parameter-perturbation-reanalysis

## Summary

Re-clusters existing Eclipse subalignment match tables with relaxed graph parameters (diameter=2, minimum clique size=3) instead of strict clique mode, exploiting Eclipse's decoupling of matching from aggregation so per-pair subalignments need not be rerun.

## When to use

Use when existing match tables from n>2 LC-MS dataset alignment yield too few matched features under strict clustering, and you want to test relaxed diameter/clique-size settings without recomputing subalignments.

## When NOT to use

- When the per-pair subalignments have not yet been run — this skill reuses existing match tables and cannot create them
- When strict clique clustering is required for downstream analyses sensitive to spurious matches — relaxing parameters increased spurious matches from 3 to 13 in the DS1–4 case

## Inputs

- DS1–DS4 subalignment match tables (Eclipse MSAligner output)
- Merged Feature Metadata + Feature Abundances (bmxp shared schema, Compound_ID x injection_id)

## Outputs

- reclustered_matches.csv (BMXP column format)
- Summary counts: matched features (4530, +669/34%), overlapping annotated features (367/381, 96%, +5), spurious matches (13, +10)

## How to apply

Load the DS1–DS4 subalignment match tables plus the merged Feature Metadata + Feature Abundances that Eclipse requires, using the shared bmxp schema (Compound_ID as feature index, injection_id as injection index). Build the feature-match graph from the existing match tables as edges, skipping the per-pair MSAligner subalignments entirely. Re-run Eclipse's graph clustering in relaxed mode with diameter=2 and minimum clique size=3. Summarize total matched features, overlapping annotated features (numerator/denominator), and spurious matches, then compare against reference values (4530 matched features, 367/381 annotations, 13 spurious matches) and export the re-clustered table to reclustered_matches.csv in BMXP column format.

## Related tools

- **Eclipse** (Graph-based clustering of feature-match edges built from existing subalignment match tables with relaxed parameters (diameter=2, min clique size=3)) — https://github.com/broadinstitute/bmxp
- **bmxp** (Provides the shared schema (FMDATA/IMDATA, Compound_ID, injection_id) and BMXP column format used for inputs and the reclustered output) — https://github.com/broadinstitute/bmxp
- **Python** (Language for loading match tables, building the feature-match graph, and running Eclipse clustering)

## Examples

```
from bmxp.eclipse import MSAligner; # build graph from existing DS1-4 match tables, then cluster with diameter=2, min_clique=3; df.to_csv('reclustered_matches.csv')
```

## Evaluation signals

- Matched-feature count reaches ~4530 (+669, +34%) over strict-mode DS1–4 results
- Overlapping annotated features reach 367 of 381 (96%, +5)
- Spurious matches total 13 (+10), matching Supplementary Code S2 reference values
- reclustered_matches.csv uses valid BMXP shared-schema column headers (Compound_ID, injection_id)
- No per-pair subalignments rerun — feature-match graph constructed solely from existing match-table edges

## Limitations

- Relaxed clustering increased spurious matches (3 → 13 in DS1–4), a precision trade-off for recall
- Eclipse and bmxp modules use a shared schema whose column labels can be globally changed (e.g. Compound_ID → Feature_ID); schema must match before clustering
- XIC shape-based feature clustering is not yet implemented in Gravity, so RT/correlation-based edges remain the basis for related clustering steps

## Evidence

- [abstract] Eclipse uses a novel graph-based approach to handle complex matching scenarios arising from n > 2 datasets: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [results] Re-clustering with diameter=2 and min clique size=3 matched 4530 features and 96% of overlapping annotations with 13 spurious matches: [evidence span withheld — anchor status: not_found]
- [readme] Eclipse requires merging Feature Metadata + Feature Abundances and uses the shared bmxp schema: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] Schema defaults: Compound_ID as feature index and injection_id as injection index: "* `Compound_ID` - Index, Project-unique feature ID (a bit of a misnomer)"
- [readme] Shared schema columns can be changed globally before running Eclipse/Blueshift/Gravity: "These can be changed globally so that all packages will use the same terminology."
