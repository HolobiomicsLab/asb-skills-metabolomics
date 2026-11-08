---
name: run-order-statistical-hypothesis-testing
description: Use when comparing two XCMS grouping results at different RT window resolutions
  (e.g. bw near RT sampling resolution vs. expected maximal RT drift) to flag peak
  groups whose retention-time sample-index range is inconsistent with sample run order.
when_to_use_negative:
- Only one XCMS grouping result is available — the detection criterion requires a
  higher-resolution alignment result to compute the feature p-values.
- Input is already realigned or realignment has already been applied by ncGTW — detection
  should precede the ncGTW realignment stage.
- Data are not LC-MS feature-level peak groups with run-order information (the discrete-uniform
  null is defined over sample indices/run order).
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: ncGTW
  role: Provides misalignDetect(), the order-statistics p-value test and disjoint-neighbor
    criterion that flags misaligned feature groups from XCMS output.
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Supplies the two grouping results (different RT window/bw settings) that serve
    as paired inputs to the misalignment test.
- name: R
  role: Runtime environment; ncGTW is an R package available via Bioconductor.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:bf4e532dc4fd21070432ef3175da19f35b8e559f0775afc336fc608e92af9685
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - R
  - XCMS
  - ncGTW
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# run-order-statistical-hypothesis-testing

## Summary

Statistical misalignment detection from the ncGTW package: each LC-MS feature's sample-index (run-order) range is tested under a discrete-uniform null via order-statistics p-values, and features with sufficiently small p-values plus disjoint neighbor subsets are flagged for realignment.

## When to use

Use when comparing two XCMS grouping results at different RT window resolutions (e.g. bw near RT sampling resolution vs. expected maximal RT drift) to flag peak groups whose retention-time sample-index range is inconsistent with sample run order.

## When NOT to use

- Only one XCMS grouping result is available — the detection criterion requires a higher-resolution alignment result to compute the feature p-values.
- Input is already realigned or realignment has already been applied by ncGTW — detection should precede the ncGTW realignment stage.
- Data are not LC-MS feature-level peak groups with run-order information (the discrete-uniform null is defined over sample indices/run order).

## Inputs

- Higher-resolution XCMS grouping result (e.g. xcmsSmallWin, RT window near RT sampling resolution)
- Lower-resolution XCMS grouping result (e.g. xcmsLargeWin, RT window set to expected maximal RT drift)
- m/z tolerance parameter in ppm (vignette example: ppm = 15)

## Outputs

- excuGroups table of peak groups (features) detected as misaligned
- Optional EIC plots per flagged group from plotGroup, colored by sample run order

## How to apply

Produce two XCMS grouping results on the same peak table: a higher-resolution grouping with RT window near the RT sampling resolution (inverse of scan frequency) and a lower-resolution grouping with RT window set to the expected maximal RT drift. Run ncGTW's misalignDetect() on both, with a ppm tolerance (the vignette example uses ppm = 15); the function computes an order-statistics p-value per feature under a discrete-uniform null hypothesis that alignment is accurate, then combines the two criteria — small p-value and disjoint sample subsets among neighbors in the same m/z bin — to decide which features are misaligned. The rationale is that a correctly aligned feature's detected peaks should spread across the full run order, so an inappropriately narrow sample-index range is statistically improbable. Collect the flagged features in the returned excluGroups table and optionally verify with plotGroup EIC plots colored by sample run order. In the paper's vignette example, exactly two peak groups are detected as misaligned.

## Related tools

- **ncGTW** (Provides misalignDetect(), the order-statistics p-value test and disjoint-neighbor criterion that flags misaligned feature groups from XCMS output.) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Supplies the two grouping results (different RT window/bw settings) that serve as paired inputs to the misalignment test.)
- **R** (Runtime environment; ncGTW is an R package available via Bioconductor.)

## Examples

```
excuGroups <- misalignDetect(xcmsLargeWin, xcmsSmallWin, ppm = 15)
```

## Evaluation signals

- The returned excluGroups table contains exactly two peak groups for the ncGTW vignette example dataset, matching the reported count.
- Each flagged feature has an order-statistics p-value (discrete-uniform null) sufficiently small under the higher-resolution alignment result.
- Flagged features satisfy the disjoint-sample-subsets criterion among neighboring features in the same m/z bin.
- plotGroup EIC plots of flagged groups show RT drift correlated with sample run order.
- Written artifact (excuGroups.csv) parses with exactly two rows and expected columns.

## Limitations

- The false positive rate can exceed the theoretical 0.05 threshold on real data (higher in the Rotterdam dataset); ncGTW instead accepts these and screens them at a later post-alignment stage via correlation coefficient and overlapping area scores.
- Exact p-value thresholds and ncGTW parameter settings are not exhaustively listed in the paper; users must consult package defaults.
- Detection is defined against XCMS-style grouping; misalignment not visible as a narrow sample-index range in the two grouping results may be missed.

## Evidence
