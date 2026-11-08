---
name: entropy-based-peak-clarity-scoring
description: Decision support for scoring a per-atom modification-site likelihood
  array Ω from ModiFinder against a known true modification site, when a single metric
  balancing proximity and ambiguity of the prediction is needed.
when_to_use_negative:
- When the input is raw MS/MS peak data before ModiFinder scoring — the metric requires
  the already-computed per-atom likelihood array Ω, not spectra
- When no known true modification site γ is available (this is a supervised evaluation
  metric for benchmarking, not for discovery on unlabeled analogs)
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: ModiFinder
  role: Produces the per-atom likelihood array Ω being evaluated; the Average-distance
    score was adopted as ModiFinder's main evaluation metric
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:24f1883c7cde9d90f9559e5d2006316bf4069a1f232ee76d76123125a98739f6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - ModiFinder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# entropy-based-peak-clarity-scoring

## Summary

Computes ModiFinder's Average-distance evaluation score, a likelihood-weighted exponential decay metric over per-atom modification-site scores that balances proximity cover (high-scoring atoms near the true site) against ambiguity cover (informativeness/entropy of the likelihood array Ω).

## When to use

Decision support for scoring a per-atom modification-site likelihood array Ω from ModiFinder against a known true modification site, when a single metric balancing proximity and ambiguity of the prediction is needed.

## When NOT to use

- When the input is raw MS/MS peak data before ModiFinder scoring — the metric requires the already-computed per-atom likelihood array Ω, not spectra
- When no known true modification site γ is available (this is a supervised evaluation metric for benchmarking, not for discovery on unlabeled analogs)

## Inputs

- Per-atom modification-site likelihood array Ω from ModiFinder for the known compound (e.g., GNPS accession CCMSLIB00010113829 plus its analog structure)
- 2D molecular graph of the known compound with atom identities and the true modification site γ
- Compound identifier / structure of the analog pair (known and unknown)

## Outputs

- Average-distance evaluation score per compound pair
- Per-atom contributions to the score and normalized graph distances d(i,γ)/Φ
- evaluation_report.json with computed scores and intermediate per-atom values

## How to apply

Normalize the per-atom likelihood array Ω to sum to 1, zero out any atom scores below 0.5, raise all remaining values to the power of 4, and renormalize to sum 1 — these are ModiFinder's final post-processing parameters. Compute per-atom graph distances d(i,γ) on the 2D molecular graph of the known compound and normalize by the diameter Φ (the greatest shortest path between any two nodes). Score each atom by the likelihood-weighted exponential decay term exp(-d/Φ)/Φ(-d/Φ) summed over atoms: concentrated, proximity-correct likelihoods yield high scores while diffuse (ambiguous) likelihoods dilute their contributions. Validate the metric on illustrative peaked vs. diffuse distributions (SI Fig. SI-5 / SI Fig. SI-16) to confirm sharper, proximity-concentrated distributions score higher, then write scores and per-atom contributions to evaluation_report.json.

## Related tools

- **ModiFinder** (Produces the per-atom likelihood array Ω being evaluated; the Average-distance score was adopted as ModiFinder's main evaluation metric) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base

## Examples

```
import networkx as nx; D = nx.shortest_path_length(G, source=gamma); phi = max(max(sp.values()) for sp in [nx.shortest_path_length(G, source=n) for n in G]); score = sum(w[i]*math.exp(-d/phi)/phi for i, d in D.items())
```

## Evaluation signals

- Ω sums to 1 after each normalization; values below 0.5 are zeroed and all values raised to the power 4 before the final renormalization
- Graph distances normalized by the molecular-graph diameter Φ (greatest shortest path between any two nodes), so scores are comparable across compounds of different size
- Peaked likelihood distributions concentrated near the true site score higher than diffuse distributions with equivalent mass (verified on SI Fig. SI-5 examples)
- Sum of likelihood-weighted exp(-d/Φ)/Φ(-d/Φ) terms reproduces the reported Average-distance score when recomputed independently
- Computed scores and per-atom contributions present in evaluation_report.json

## Limitations

- Diffuse likelihood arrays are penalized even when mass is concentrated in a plausible region — the metric trades ambiguity cover against proximity cover by design
- The 0.5 threshold and power-of-4 exponent are ModiFinder's tuned post-processing values; re-tuning them changes score comparability with published benchmarks
- Metric operates on 2D molecular-graph distances, not 3D structural distances

## Evidence

- [results] The Average-distance evaluation was introduced and adopted as the main evaluation metric for ModiFinder: [evidence span withheld — anchor status: not_found]
- [readme] ModiFinder is a tool for site localization of structural modifications using MS/MS data.: [evidence span withheld — anchor status: not_found]
