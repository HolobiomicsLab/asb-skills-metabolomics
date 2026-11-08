---
name: site-localization-evaluation-scoring
description: Use when a ModiFinder run has produced a per-atom modification-site likelihood
  array and you need a single comparable quality score, or when benchmarking site-localization
  accuracy across depth settings or parameter sweeps.
when_to_use_negative:
- When no ground-truth modification site γ is known — the Average-distance metric
  scores against the true site and cannot be computed in oracle-free discovery runs
- When raw spectra have not yet been preprocessed and aligned (peaks <1% of base peak
  removed, Euclidean-norm normalized, 40 ppm substructure assignment) — the likelihood
  array Ω does not yet exist
- When the goal is analog identification rather than site localization — MS/MS library
  search or analog library search is the appropriate skill
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ModiFinder
  role: Provides the Compound class, generate_probabilities() likelihood arrays, and
    BasicEvaluationEngine.evaluate_single() with the default Average-distance method
    used for scoring
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: MAGMaAnnotationEngine performs breadth-first combinatorial fragmentation (depth
    2–4) to assign substructures to peaks feeding the scored likelihood arrays
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:5a11ae20fd73a9e48d636e3809b684587269cd95689ba6d466bef79f982f3b25
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - ModiFinder
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# site-localization-evaluation-scoring

## Summary

Scores ModiFinder modification-site predictions using the Average-distance metric, which balances proximity cover (high-scoring atoms near the true site) against ambiguity cover (informativeness of the normalized likelihood array Ω), serving as the paper's main evaluation method.

## When to use

Use when a ModiFinder run has produced a per-atom modification-site likelihood array and you need a single comparable quality score, or when benchmarking site-localization accuracy across depth settings or parameter sweeps.

## When NOT to use

- When no ground-truth modification site γ is known — the Average-distance metric scores against the true site and cannot be computed in oracle-free discovery runs
- When raw spectra have not yet been preprocessed and aligned (peaks <1% of base peak removed, Euclidean-norm normalized, 40 ppm substructure assignment) — the likelihood array Ω does not yet exist
- When the goal is analog identification rather than site localization — MS/MS library search or analog library search is the appropriate skill

## Inputs

- ModiFinder Compound object built from GNPS spectral library entries (e.g., CCMSLIB00010113829) plus the analog structure
- Per-atom modification-site prediction likelihood array Ω from ModiFinder.generate_probabilities()
- 2D molecular graph of the known compound (atoms, bonds, diameter Φ)

## Outputs

- Average-distance evaluation score per compound pair (e.g., MF-O score)
- evaluation_report.json with computed scores and per-atom contributions
- Per-pair depth sweep table (e.g., fragmentation_depth_sweep_results.csv)

## How to apply

Load the known compound (e.g., GNPS accession CCMSLIB00010113829 plus its analog structure) and its prediction likelihood array Ω via the ModiFinder package. Post-process Ω by zeroing any atom scores below 0.5, raising all values to the power of 4, and renormalizing to sum to 1 to emphasize high-scoring atoms. Compute per-atom graph distances d(i,γ) on the 2D molecular graph and normalize them by the diameter Φ (the greatest shortest path between any two nodes). Score the prediction as the likelihood-weighted sum of exp(-d/Φ)/Φ(-d/Φ) over atoms: peaked distributions concentrated near the true site score higher, while diffuse likelihoods dilute contributions, jointly capturing proximity cover and ambiguity cover. Use BasicEvaluationEngine.evaluate_single() with the default Average-distance method to compute the score, then aggregate per-pair results (e.g., across MAGMaAnnotationEngine fragmentation depths 2–4) and write them with intermediate per-atom contributions to an evaluation report.

## Related tools

- **ModiFinder** (Provides the Compound class, generate_probabilities() likelihood arrays, and BasicEvaluationEngine.evaluate_single() with the default Average-distance method used for scoring) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (MAGMaAnnotationEngine performs breadth-first combinatorial fragmentation (depth 2–4) to assign substructures to peaks feeding the scored likelihood arrays)

## Examples

```
from modafinder import BasicEvaluationEngine; score = BasicEvaluationEngine().evaluate_single(compound, probabilities)  # default Average-distance method
```

## Evaluation signals

- Normalized Ω sums to 1, atoms with scores below 0.5 are zeroed, and remaining values are raised to the power of 4 before scoring
- Graph distances d(i,γ) are normalized by the molecular-graph diameter Φ (greatest shortest path between any two nodes)
- Peaked, proximity-concentrated likelihood distributions score strictly higher than diffuse distributions of equal mass (SI Fig. SI-5 illustrative examples)
- Depth sweep recapitulates the reported trend: 19% increase in annotated shifted peaks from MAGMa depth 2 to depth 4 and higher MF-O scores at depths 3–4 versus depth 2
- 8,033 of 12,909 benchmark pairs (62%) survive the explainable-shifted-peak filter, giving an expected evaluable subset size

## Limitations

- The metric balances proximity and ambiguity cover but diffuse likelihoods are penalized by design, so genuinely ambiguous cases receive low scores even when the true site ranks high
- Oracle mode is required for evaluation: substructures assigned to shifted peaks that lack the true modification site must be filtered out, which is only possible when the ground-truth site is known
- Score interpretation is qualitative — the paper relies on illustrative examples (SI Fig. SI-5) of different evaluation score values rather than absolute thresholds

## Evidence

- [results] The Average-distance evaluation was introduced and adopted as the main evaluation metric for ModiFinder: [evidence span withheld — anchor status: not_found]
- [readme] ModiFinder is a tool for site localization: [evidence span withheld — anchor status: external_doc]
