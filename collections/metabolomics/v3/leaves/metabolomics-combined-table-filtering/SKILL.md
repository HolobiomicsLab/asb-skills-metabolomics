---
name: metabolomics-combined-table-filtering
description: Use when a scored metabCombiner combined table contains many conflicting
  or low-scoring feature pair alignments (e.g. 14024 FPAs) and the goal is a reduced
  final table of intersecting features.
when_to_use_negative:
- Datasets acquired from excessively disparate chromatographic techniques (e.g. an
  RPLC dataset vs a HILIC dataset), where elution order is not preserved and alignment
  itself is unsuitable
- Datasets not acquired in the same ionization mode, or samples without strong overlap
  in metabolomic composition
- Datasets whose abundance values have been normalized or transformed in a way that
  removes ranked abundance order information
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: metabCombiner
  role: R package providing labelRows(), which marks conflicting and low-scoring feature
    pair alignments for removal from the combined table
  repo: https://github.com/hhabra/metabCombiner
- name: R
  role: Runtime environment in which the metabCombiner package labelRows() step is
    executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c03693
    title: '<i>metabCombiner</i>: Paired Untargeted LC-HRMS Metabolomics Feature Matching
      and Concatenation of Disparately Acquired Data Sets'
  source_content_hash: sha256:5145bb2ab4ab887fe17b1e1fff279d51ecd5810aa0744cc8bfb3fb9860d91e3b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hhabra/metabCombiner
  repo_url: https://github.com/hhabra/metabCombiner
  tools:
  - R
  - metabCombiner
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolomics-combined-table-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduces metabCombiner's combined table of feature pair alignments (FPAs) by running labelRows() with score/rank thresholds and manual inspection, removing misaligned or conflicting rows so only credible intersecting feature measurements remain.

## When to use

Use when a scored metabCombiner combined table contains many conflicting or low-scoring feature pair alignments (e.g. 14024 FPAs) and the goal is a reduced final table of intersecting features.

## When NOT to use

- Datasets acquired from excessively disparate chromatographic techniques (e.g. an RPLC dataset vs a HILIC dataset), where elution order is not preserved and alignment itself is unsuitable
- Datasets not acquired in the same ionization mode, or samples without strong overlap in metabolomic composition
- Datasets whose abundance values have been normalized or transformed in a way that removes ranked abundance order information

## Inputs

- Scored metabCombiner combined table of feature pair alignments (FPAs) produced by fit_gam and calcScores
- Threshold parameters: minScore, rankX, rankY, and delta score tolerance

## Outputs

- Reduced combined table of intersecting feature measurements with conflicting/low-scoring FPAs removed
- Row labels (e.g. REMOVED or CONFLICT) marking alignments flagged by labelRows()

## How to apply

After the Feature Pair Alignment Scoring step (fit_gam and calcScores), run metabCombiner::labelRows() on the scored combined table — for the plasma20/plasma30 pair this means minScore = 0.5, rankX = 3, rankY = 2, and delta = 0.2 — so conflicting and low-scoring FPAs are automatically marked for removal (this removed 6765 of 14024 FPAs on the plasma data). Inspect the rows labeled by labelRows() (e.g. REMOVED or CONFLICT) and manually remove additional alignments that warrant it (~400 more on the plasma example), confirming the final reduced table contains fewer than 6900 FPAs. The rationale is that each feature may appear in multiple m/z-grouped pairwise alignments, and only the highest-scoring, rank-consistent alignments reflect true correspondence between identical features.

## Related tools

- **metabCombiner** (R package providing labelRows(), which marks conflicting and low-scoring feature pair alignments for removal from the combined table) — https://github.com/hhabra/metabCombiner
- **R** (Runtime environment in which the metabCombiner package labelRows() step is executed)

## Examples

```
reduced <- metabCombiner::labelRows(combined, minScore = 0.5, rankX = 3, rankY = 2, delta = 0.2)
```

## Evaluation signals

- Automatic removal count from labelRows() matches expectation (6765 of 14024 FPAs on the plasma20/plasma30 example with minScore=0.5, rankX=3, rankY=2, delta=0.2)
- Post-inspection manual removals total approximately 400 additional alignments
- Final reduced combined table row count is under 6900 FPAs
- Every removed row carries a labelRows() label (REMOVED or CONFLICT) or is documented by manual inspection
- Weighted per-compound matching accuracy of the retained alignments stays near the ~90% reported for the plasma pair

## Limitations

- The paper does not state the version of the metabCombiner R package used
- HILIC-acquired datasets are more difficult to align than RPLC datasets, so threshold outcomes may differ
- Preprocessing errors such as incomplete or multipeak integration, low signal-to-noise features, and missed peaks propagate into alignment errors that thresholds may not fully resolve
- A manual inspection pass (~400 removals) is required in addition to automatic labeling; results are not fully automated

## Evidence
