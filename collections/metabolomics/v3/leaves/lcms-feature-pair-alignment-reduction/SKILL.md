---
name: lcms-feature-pair-alignment-reduction
description: Use when metabCombiner has produced a scored combined table after fit_gam
  and calcScores, and conflicting or low-scoring FPAs must be pruned to form the final
  reduced table before downstream statistics.
when_to_use_negative:
- The input datasets were acquired with excessively disparate chromatographic techniques
  (e.g. an RPLC dataset vs a HILIC dataset) or in different ionization modes — the
  article states such pairs are not suitable for metabCombiner alignment.
- Abundance values have been normalized or transformed in a way that removes ranked
  abundance order information, which labelRows() ranking relies on.
- The combined table has not yet been scored by calcScores after anchor-based spline
  RT mapping, so no score/rank values exist to threshold.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3383
tools:
- name: metabCombiner
  role: Provides labelRows(), the row-reduction step that marks conflicting and low-scoring
    FPAs in the scored combined table for removal
  repo: https://github.com/hhabra/metabCombiner
- name: R
  role: Runtime environment in which the metabCombiner package (and labelRows()) is
    executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c03693
    title: '<i>metabCombiner</i>: Paired Untargeted LC-HRMS Metabolomics Feature Matching
      and Concatenation of Disparately Acquired Data Sets'
  source_content_hash: sha256:94fb3d5907e67c008a8a8fdc6bf17e6868b3e56bfc84ada21574d2de3e940cfd
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lcms-feature-pair-alignment-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduces a scored metabCombiner combined table by running labelRows() with score/rank/delta thresholds (minScore=0.5, rankX=3, rankY=2, delta=0.2) to automatically mark conflicting and low-scoring feature pair alignments (FPAs) for removal, followed by a short manual inspection pass, yielding the final reduced combined table (e.g. from 14024 FPAs to under 6900 rows on the plasma20/plasma30 pair).

## When to use

Use when metabCombiner has produced a scored combined table after fit_gam and calcScores, and conflicting or low-scoring FPAs must be pruned to form the final reduced table before downstream statistics.

## When NOT to use

- The input datasets were acquired with excessively disparate chromatographic techniques (e.g. an RPLC dataset vs a HILIC dataset) or in different ionization modes — the article states such pairs are not suitable for metabCombiner alignment.
- Abundance values have been normalized or transformed in a way that removes ranked abundance order information, which labelRows() ranking relies on.
- The combined table has not yet been scored by calcScores after anchor-based spline RT mapping, so no score/rank values exist to threshold.

## Inputs

- Scored metabCombiner combined table (FPAs with scores and ranks) from a fit_gam + calcScores run on a paired untargeted LC-HRMS dataset pair (e.g. plasma20/plasma30)
- labelRows() threshold parameters: minScore = 0.5, rankX = 3, rankY = 2, delta = 0.2

## Outputs

- Reduced combined table with conflicting/low-scoring FPAs removed (fewer than 6900 rows from 14024 initial FPAs on the plasma pair)
- Label annotations (REMOVED or CONFLICT) per FPA supporting manual inspection

## How to apply

Load the scored combined table produced by metabCombiner's fit_gam and calcScores workflow on the dataset pair (e.g. plasma20/plasma30), then run metabCombiner::labelRows() with minScore = 0.5, rankX = 3, rankY = 2, and delta = 0.2 so rows are ranked by m/z, mapped retention time, and relative abundance similarity and low-scoring or conflicting alignments are automatically flagged for removal. Inspect the flagged rows (labeled REMOVED or CONFLICT) and manually remove any additional alignments that warrant it — on the plasma data this was roughly 400 rows after 6765 automatic removals out of 14024 initial FPAs. The rationale is that groupings by m/z generate many candidate FPAs per feature, so only the best-scoring, non-conflicting alignment per feature should survive into the combined table. Confirm the final table contains fewer than 6900 FPAs and write it out as the reduced combined table before any batch-effect correction and statistical analysis.

## Related tools

- **metabCombiner** (Provides labelRows(), the row-reduction step that marks conflicting and low-scoring FPAs in the scored combined table for removal) — https://github.com/hhabra/metabCombiner
- **R** (Runtime environment in which the metabCombiner package (and labelRows()) is executed)

## Examples

```
combined <- metabCombiner::labelRows(combined, minScore = 0.5, rankX = 3, rankY = 2, delta = 0.2)
```

## Evaluation signals

- Automatic removal count equals 6765 of the 14024 initial FPAs on the plasma20/plasma30 dataset pair.
- Post-inspection manual removal count is approximately 400 additional alignments.
- Final reduced combined table contains fewer than 6900 FPAs, consistent with the paper's Combined Table Reduction step.
- All surviving FPAs meet minScore >= 0.5 and the rankX <= 3 / rankY <= 2 / delta <= 0.2 constraints.
- Rows removed are those labeled REMOVED or CONFLICT by labelRows().

## Limitations

- Datasets acquired from HILIC methods are more difficult to align than those by RPLC methods, so threshold behavior may differ.
- The manual inspection step (about 400 further removals) is subjective and not fully automated by labelRows().
- The paper does not state the version of the metabCombiner R package used, so exact labelRows() defaults or behavior may vary.
- Inputs must be biologically similar samples with strongly overlapping metabolomic composition and mostly preserved elution order.

## Evidence
