---
name: graph-distance-normalization-by-diameter
description: Use when computing distance-based scores over atoms of a known compound's
  2D molecular graph (e.g. for ModiFinder's Average-distance evaluation) and you need
  distances comparable across molecules of different sizes.
when_to_use_negative:
- The prediction output is a ranked shortlist of candidate structures rather than
  an atom-level likelihood array Ω over the known compound's atoms.
- No known/analog structure graph is available, so per-atom 2D graph distances cannot
  be computed.
- The goal is MS/MS library matching or analog identification itself rather than scoring
  where the modification site is on an already-known compound.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: ModiFinder
  role: Produces the atom-level likelihood array Ω and supplies the Compound/analog
    structures whose 2D molecular graph distances are normalized by the diameter Φ
    for the Average-distance metric
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:94dcac8e31d272bc8d4bcf3b2e1a2d5ab493f7c1487e908ba3f4a07e59f819e2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - ModiFinder
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# graph-distance-normalization-by-diameter

## Summary

Normalizes per-atom graph distances on a 2D molecular graph by the molecule's diameter Φ (the greatest shortest path between any two nodes), enabling size-invariant scoring of modification-site predictions in ModiFinder's Average-distance evaluation metric.

## When to use

Use when computing distance-based scores over atoms of a known compound's 2D molecular graph (e.g. for ModiFinder's Average-distance evaluation) and you need distances comparable across molecules of different sizes.

## When NOT to use

- The prediction output is a ranked shortlist of candidate structures rather than an atom-level likelihood array Ω over the known compound's atoms.
- No known/analog structure graph is available, so per-atom 2D graph distances cannot be computed.
- The goal is MS/MS library matching or analog identification itself rather than scoring where the modification site is on an already-known compound.

## Inputs

- Known compound structure (2D molecular graph), e.g. Compound from GNPS accession CCMSLIB00010113829 plus its analog structure
- ModiFinder atom-level modification-site prediction likelihood array Ω (post-processed: values < 0.5 zeroed, values raised to the power of 4, renormalized to sum 1)

## Outputs

- Per-atom graph distances d(i,γ) normalized by the molecular diameter Φ
- Likelihood-weighted Average-distance evaluation score per compound, plus intermediate per-atom contributions written to evaluation_report.json

## How to apply

Given a known compound (e.g. from GNPS accession CCMSLIB00010113829) and its atom-level likelihood array Ω from ModiFinder, first normalize Ω to sum to 1, zero out any atom scores below 0.5, raise all values to the power of 4, and renormalize to sum to 1. Compute the per-atom graph distance d(i,γ) on the 2D molecular graph using shortest-path distances between atom nodes. Determine the molecular diameter Φ as the greatest shortest path between any two nodes of the graph, then normalize each distance by Φ so that scores are comparable across molecules of different sizes. The normalized distances feed the Average-distance score, the likelihood-weighted exponential decay term exp(-d/Φ) summed over atoms, which balances proximity cover (high-scoring atoms near the true modification site) against ambiguity cover (diffuse likelihoods dilute contributions). Validate on illustrative peaked vs. diffuse likelihood distributions (SI Fig. SI-5) that sharper, proximity-concentrated distributions yield higher scores.

## Related tools

- **ModiFinder** (Produces the atom-level likelihood array Ω and supplies the Compound/analog structures whose 2D molecular graph distances are normalized by the diameter Φ for the Average-distance metric) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base

## Examples

```
import networkx as nx; d = nx.shortest_path_length(G, source=site); Phi = max(nx.shortest_path_length(G, u, v) for u in G for v in G); d_norm = {i: d[i]/Phi for i in d}
```

## Evaluation signals

- Ω is normalized to sum to 1, values below 0.5 are set to 0, remaining values are raised to the power of 4, and the array is renormalized to sum 1 before distance weighting.
- The molecular diameter Φ equals the greatest shortest path between any two nodes of the 2D molecular graph; all d(i,γ) values are divided by Φ and thus fall in [0, 1].
- Peaked likelihood distributions concentrated near the true modification site score higher than diffuse distributions of the same compound (as illustrated in SI Fig. SI-5).
- Computed Average-distance scores and per-atom intermediate contributions are present in evaluation_report.json.
- Per-atom contributions are likelihood-weighted via exp(-d/Φ) so diffuse likelihoods dilute, rather than concentrate, the total score.

## Limitations

- Diameter normalization depends on accurate 2D molecular graph topology; the metric is only as good as the known compound structure used to build the graph.
- The Average-distance score balances proximity cover and ambiguity cover by design, so diffuse likelihoods dilute contributions and may under-reward predictions spread over multiple plausible sites.
- Illustrative interpretation of score values relies on worked examples (SI Fig. SI-5); absolute score thresholds are not defined independently of these examples.

## Evidence

- [methods] Average-distance evaluation metric: [evidence span withheld — anchor status: not_found]
- [results] metric rationale: [evidence span withheld — anchor status: not_found]
- [readme] tool availability: [evidence span withheld — anchor status: not_found]
