---
name: fpa-count-verification-across-datasets
description: 'Use when auditing a metabCombiner combined table reduction: you must
  remove misaligned feature pair alignments (FPAs) via labelRows() thresholds and
  verify the resulting row counts against expected removal statistics.'
when_to_use_negative:
- The datasets were acquired from excessively disparate chromatographic techniques,
  such as an RPLC dataset paired with a HILIC dataset — the elution order is not preserved
  and the paper says such pairs are not suitable.
- The input datasets were acquired in different ionization modes (positive vs. negative),
  which the paper lists as a prerequisite for metabCombiner.
- The combined table is already reduced and only needs downstream batch-effect correction,
  not another FPA removal pass.
tools:
- name: metabCombiner R package
  role: Provides labelRows() for marking conflicting and low-scoring feature pair
    alignments for removal from the combined table
  repo: https://github.com/hhabra/metabCombiner
- name: R
  role: Language environment in which the metabCombiner package (including labelRows())
    is run
  repo: https://github.com/hhabra/metabCombiner
- name: fit_gam
  role: Upstream GAM spline-fitting step that supplies the scored plasma combined
    table whose FPAs are counted and reduced
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c03693
    title: '<i>metabCombiner</i>: Paired Untargeted LC-HRMS Metabolomics Feature Matching
      and Concatenation of Disparately Acquired Data Sets'
  source_content_hash: sha256:e3218c744f5a7447ef77199403f1ddf9e746d2f363554ac8b82858d54281bdd3
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
  - fit_gam
  - metabCombiner R package
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# FPA count verification across datasets

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verifies the metabCombiner 'Combined Table Reduction' step by tracking how many feature pair alignments (FPAs) survive labelRows() filtering — on the plasma20/plasma30 pair, 6765 of 14024 FPAs are removed automatically plus ~400 by inspection, leaving fewer than 6900 rows in the final combined table.

## When to use

Use when auditing a metabCombiner combined table reduction: you must remove misaligned feature pair alignments (FPAs) via labelRows() thresholds and verify the resulting row counts against expected removal statistics.

## When NOT to use

- The datasets were acquired from excessively disparate chromatographic techniques, such as an RPLC dataset paired with a HILIC dataset — the elution order is not preserved and the paper says such pairs are not suitable.
- The input datasets were acquired in different ionization modes (positive vs. negative), which the paper lists as a prerequisite for metabCombiner.
- The combined table is already reduced and only needs downstream batch-effect correction, not another FPA removal pass.

## Inputs

- Scored metabCombiner combined table of feature pair alignments (FPAs) from fit_gam and calcScores on the plasma20/plasma30 dataset pair
- labelRows() threshold parameters: minScore = 0.5, rankX = 3, rankY = 2, delta = 0.2

## Outputs

- Reduced combined table with fewer than 6900 FPAs (14024 minus 6765 automatic removals minus ~400 manual inspection removals)
- Row labels (e.g. REMOVED or CONFLICT) from labelRows() marking alignments for removal
- Verified removal counts: 6765 automatic and ~400 post-inspection

## How to apply

Start from the scored combined table produced by metabCombiner's fit_gam/calcScores workflow on the plasma20/plasma30 dataset pair (14024 initial FPAs). Run metabCombiner::labelRows() with minScore = 0.5, rankX = 3, rankY = 2, and delta = 0.2 to automatically mark conflicting and low-scoring FPAs for removal — these thresholds label rows REMOVED or CONFLICT based on their similarity score, within-dataset rank by m/z, mapped retention time, and relative abundance, and the delta score tolerance. Confirm the automatic removal count is 6765 of 14024 FPAs. Then inspect the flagged rows and remove approximately 400 additional alignments that warrant manual removal. Finally verify the reduced table contains fewer than 6900 FPAs before writing it out as the final reduced table; this count bookkeeping is what validates the reduction against the paper's Combined Table Reduction step.

## Related tools

- **metabCombiner R package** (Provides labelRows() for marking conflicting and low-scoring feature pair alignments for removal from the combined table) — https://github.com/hhabra/metabCombiner
- **R** (Language environment in which the metabCombiner package (including labelRows()) is run) — https://github.com/hhabra/metabCombiner
- **fit_gam** (Upstream GAM spline-fitting step that supplies the scored plasma combined table whose FPAs are counted and reduced)

## Examples

```
combined <- metabCombiner::labelRows(combined, minScore = 0.5, rankX = 3, rankY = 2, delta = 0.2)
```

## Evaluation signals

- Automatic removal count equals 6765 of the 14024 initial FPAs after labelRows() with minScore = 0.5, rankX = 3, rankY = 2, delta = 0.2.
- Post-inspection removal count is approximately 400 rows flagged by labelRows() but warranting manual removal.
- Final reduced combined table contains fewer than 6900 FPAs.
- Upstream quality checks that make the counts meaningful: mean absolute retention time prediction error of roughly 0.06 min and weighted per-compound matching accuracy of approximately 90% on the plasma dataset pair.

## Limitations

- The paper does not state the version of the metabCombiner R package used, so exact labelRows() behavior may vary across versions.
- Preprocessing errors such as incomplete or multipeak integration, low signal-to-noise features, and missed peaks propagate into the combined table and affect FPA counts.
- Datasets acquired from HILIC methods are more difficult to align than those by RPLC methods, so removal counts will not transfer across chromatographic regimes.
- Abundance values must not be normalized or transformed in a way that removes ranked abundance order information, or the rank-based thresholds (rankX = 3, rankY = 2) lose their meaning.

## Evidence

- [abstract] ranks alignments by m/z, mapped retention time, and relative abundance similarity: "ranks alignments by m/z, mapped retention time, and relative abundance similarity"
- [intro] We demonstrate each step on a pair of human plasma metabolomics datasets contained within the package.: "We demonstrate each step on a pair of human plasma metabolomics datasets contained within the package."
- [readme] metabCombiner takes peak-picked and conventionally aligned untargeted LC-MS datasets and determines the overlapping <mass-to-charge (m/z), retention time (rt)> features: "metabCombiner takes peak-picked and conventionally aligned untargeted LC-MS datasets and determines the overlapping <mass-to-charge (m/z), retention time (rt)>"
