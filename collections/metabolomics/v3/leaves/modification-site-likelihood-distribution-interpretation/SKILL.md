---
name: modification-site-likelihood-distribution-interpretation
description: Use when a ModiFinder prediction has produced a per-atom likelihood array
  Ω for the known compound and you need to collapse it into a single evaluation score
  that rewards peaked, site-concentrated distributions over diffuse, ambiguous ones.
when_to_use_negative:
- You only need ModiFinder's qualitative top-ranked modification-site prediction,
  not a quantitative evaluation of the likelihood distribution
- Input peaks have not yet passed ModiFinder's pipeline (intensity filtering <1% base
  peak, Euclidean-norm-1 normalization, 40 ppm substructure assignment), so no valid
  Ω exists yet
- The structure has no calculable 2D graph diameter Φ (e.g., disconnected or trivial
  graph), making normalized distances undefined
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ModiFinder
  role: Produces the per-atom likelihood array Ω that this skill post-processes and
    scores; provides the Python package used to load the Compound and prediction
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:1532021ef9138e767462411533303a50338091751737dfbc18e3c949ab63c7dc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - ModiFinder
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# modification-site-likelihood-distribution-interpretation

## Summary

Interprets ModiFinder's per-atom modification-site likelihood array Ω by post-processing it (threshold 0.5, power of 4, renormalization) and scoring it with the Average-distance metric, which balances proximity cover (high-scoring atoms near the true site) against ambiguity cover (informativeness of a diffuse likelihood array).

## When to use

Use when a ModiFinder prediction has produced a per-atom likelihood array Ω for the known compound and you need to collapse it into a single evaluation score that rewards peaked, site-concentrated distributions over diffuse, ambiguous ones.

## When NOT to use

- You only need ModiFinder's qualitative top-ranked modification-site prediction, not a quantitative evaluation of the likelihood distribution
- Input peaks have not yet passed ModiFinder's pipeline (intensity filtering <1% base peak, Euclidean-norm-1 normalization, 40 ppm substructure assignment), so no valid Ω exists yet
- The structure has no calculable 2D graph diameter Φ (e.g., disconnected or trivial graph), making normalized distances undefined

## Inputs

- ModiFinder per-atom modification-site likelihood array Ω over the known (S-known) compound graph
- Known compound structure (e.g., GNPS library compound CCMSLIB00010113829) and its analog structure
- 2D molecular graph of the known compound for shortest-path distance computation

## Outputs

- Average-distance evaluation score for the likelihood array
- Per-atom weighted contributions to the score
- evaluation_report.json containing computed scores and intermediate per-atom contributions

## How to apply

Load the compound (e.g., Compound from GNPS accession CCMSLIB00010113829 plus its analog structure) and its prediction likelihood array Ω with the ModiFinder package. Post-process Ω exactly as the paper specifies: normalize to sum 1, set any atom score below 0.5 to 0, raise all remaining values to the power of 4, and renormalize to sum 1 — this sharpens differences among high-scoring atoms. Compute per-atom graph distances d(i,γ) on the 2D molecular graph and normalize by the diameter Φ (the greatest shortest path between any two nodes). Score the distribution with the likelihood-weighted exponential decay term exp(-d/Φ), summed over atoms; peaked distributions concentrated near the true site score higher, while diffuse likelihoods dilute their own contributions. Compare scores against the illustrative peaked vs. diffuse examples in SI Fig. SI-5 / SI Fig. SI-16 to sanity-check the range, then write scores and per-atom contributions to evaluation_report.json.

## Related tools

- **ModiFinder** (Produces the per-atom likelihood array Ω that this skill post-processes and scores; provides the Python package used to load the Compound and prediction) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base

## Examples

```
omega = omega/omega.sum(); omega[omega < 0.5] = 0; omega = omega**4; omega /= omega.sum(); score = (omega * np.exp(-d/Phi)).sum()
```

## Evaluation signals

- Ω post-processing invariants: array sums to 1 before and after the power-of-4 step; no atom retains a raw score below 0.5
- A sharply peaked Ω concentrated on atoms near the true modification site yields a strictly higher Average-distance score than the diffuse variant of the same array
- All distance terms d(i,γ) are bounded in [0, 1] after normalization by the graph diameter Φ
- evaluation_report.json is produced with both the aggregate score and intermediate per-atom contributions present
- Scores on the SI Fig. SI-5 illustrative examples fall within the range depicted for those example distributions

## Limitations

- The 0.5 threshold and power-of-4 exponent are paper-specific post-processing choices; applying them to arrays from other tools may distort the distribution
- Diffuse (ambiguous) likelihoods are penalized by design — a genuinely spread-out modification signal across symmetric sites will score low even if partially correct
- Evaluation assumes the true modification site γ is known (oracle-style benchmarking); it cannot be computed for genuinely unknown sites

## Evidence

- [other] Average-distance balances proximity cover and ambiguity cover: [evidence span withheld — anchor status: not_found]
- [readme] ModiFinder purpose: [evidence span withheld — anchor status: not_found]
