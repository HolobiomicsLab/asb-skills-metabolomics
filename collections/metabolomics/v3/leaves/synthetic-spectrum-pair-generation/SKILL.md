---
name: synthetic-spectrum-pair-generation
description: Use when implementing or verifying a modified-spectrum scoring function
  (e.g. the ANN-SoLo shifted dot product) and you need ground-truth pairs with known
  modification-induced peak shifts to test recovery and ranking behavior.
when_to_use_negative:
- Scoring real experimental query spectra against a spectral library — use the full
  ANN-SoLo cascade search with FDR control instead of synthetic pairs
- Validating standard (unshifted) dot product scoring, where modification-induced
  shifts are out of scope
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ANN-SoLo
  role: Source of the shifted dot product scoring function being verified with synthetic
    spectrum pairs
  repo: https://github.com/bittremieux/ANN-SoLo
- name: Python
  role: Implementation language for generating synthetic pairs and the verification
    harness
- name: NumPy
  role: Array operations for peak shifting and intensity-product scoring
- name: C++
  role: Performance-critical implementation of the shifted dot product enumeration
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:828dcc94138e685d63dd727d8d0dfd0e46550adc360bd434090e1b45fef39865
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - C++
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# synthetic-spectrum-pair-generation

## Summary

Generates synthetic modified/unmodified spectrum pairs — a base spectrum plus a copy with peaks shifted by a known mass delta — to validate that a shifted dot product implementation recovers the expected shift and scores the true modified counterpart above decoys.

## When to use

Use when implementing or verifying a modified-spectrum scoring function (e.g. the ANN-SoLo shifted dot product) and you need ground-truth pairs with known modification-induced peak shifts to test recovery and ranking behavior.

## When NOT to use

- Scoring real experimental query spectra against a spectral library — use the full ANN-SoLo cascade search with FDR control instead of synthetic pairs
- Validating standard (unshifted) dot product scoring, where modification-induced shifts are out of scope

## Inputs

- Base unmodified peak list (m/z, intensity pairs)
- Known charge-normalized precursor mass difference (modification mass delta)
- Precursor charge state(s) for the synthetic pair

## Outputs

- Synthetic modified/unmodified spectrum pairs with known peak shifts
- shifted_dot_product_verification.json containing computed scores per pair and the recovered shift

## How to apply

Construct a base (unmodified) peak list, then create a synthetic modified copy by shifting fragment peaks by a known charge-normalized mass delta derived from the precursor mass difference. Run the shifted dot product, which linearly enumerates unshifted and shifted peak pairs scored by intensity product (fully penalizing shifted matches without annotation), then greedily sums matches so no peak is used twice. Verify the implementation recovers the expected shift and that the true modified counterpart receives a higher score than decoy pairs. Write per-pair scores and the recovered shift to a JSON verification file (shifted_dot_product_verification.json). This mirrors ANN-SoLo's rationale that explicitly accounting for modification-induced peak shifts yields a further 21% more identifications than the standard dot product.

## Related tools

- **ANN-SoLo** (Source of the shifted dot product scoring function being verified with synthetic spectrum pairs) — https://github.com/bittremieux/ANN-SoLo
- **Python** (Implementation language for generating synthetic pairs and the verification harness)
- **NumPy** (Array operations for peak shifting and intensity-product scoring)
- **C++** (Performance-critical implementation of the shifted dot product enumeration)

## Examples

```
python verify_shifted_dot_product.py --base-peaks base.npy --delta 80.0 --charge 2 --output shifted_dot_product_verification.json
```

## Evaluation signals

- Recovered shift from the shifted dot product equals the known synthetic mass delta
- True modified counterpart scores strictly higher than decoy pairs
- No single peak in either spectrum is matched more than once in the greedy sum
- Output JSON (shifted_dot_product_verification.json) contains per-pair scores and the recovered shift with expected schema
- Unshifted peak pairs are scored by intensity product while shifted matches without annotation are fully penalized

## Limitations

- Synthetic shifts by a single known mass delta do not capture real-world complexity such as multiple modifications or co-fragmenting peptides
- Fully penalized shifted matches without annotation may under-score valid but unannotated shifted matches
- Accurate FDR estimation for open searches (into which such scores feed) remains an unresolved research question

## Evidence

- [methods] Generate synthetic modified/unmodified spectrum pairs (a base spectrum plus a copy with peaks shifted by a known mass delta) and verify that the implementation recovers the expected shift: [evidence span withheld — anchor status: not_found]
- [results] A further 21% increase in identifications is achieved by using the shifted dot product instead of the standard dot product: [evidence span withheld — anchor status: not_found]
- [intro] the shifted dot product is used to accurately match modified query spectra to their unmodified library counterpart by taking into account peak shifts caused by a modification: "the shifted dot product is used to accurately match modified query spectra to their unmodified library counterpart by taking into account peak shifts caused by a modification"
- [readme] ANN-SoLo ... combined with ... the shifted dot product score to sensitively match modified spectra to their unmodified counterpart: "the shifted dot product score to sensitively match modified spectra to their unmodified counterpart"
- [discussion] It is a standing research question whether alternative approaches are needed for the accurate FDR estimation of open searches: "It is a standing research question whether alternative approaches are needed for the accurate FDR estimation of open searches"
