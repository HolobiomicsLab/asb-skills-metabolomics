---
name: average-distance-evaluation-score-computation
description: Use when you have run ModiFinder's MF-R likelihood computation on a known/unknown
  analog pair and need a scalar score (e.g. the 0.86→0.93 Kirromycin and 0.56→0.72
  Naphthomycin B case studies) to quantify modification-site localization accuracy.
when_to_use_negative:
- When no ground-truth modification site is known for the analog pair — the Average-distance
  Evaluation Score requires a verified true site and is unusable for discovery-mode
  queries
- When evaluating ModiFinder performance using proximity cover or ambiguity cover
  criteria instead of the distance-based score
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ModiFinder
  role: Provides MF-R likelihood computation, BasicEvaluationEngine, and Compound/Spectrum
    loaders used to obtain and evaluate the Average-distance Evaluation Score
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: Combinatorial fragmentation engine underlying ModiFinder's peak-to-substructure
    assignment feeding the scored likelihood distribution
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:eb625098bb7bca136d0ed8c76c1c0a0646c143b2d2dc99847fa6161008440c6e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - ModiFinder
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# average-distance-evaluation-score-computation

## Summary

Computes ModiFinder's Average-distance Evaluation Score, which quantifies how closely an MS/MS-derived modification-site likelihood distribution concentrates on the true modification atom(s) of a known molecule, enabling baseline-vs-refined comparison of localization quality.

## When to use

Use when you have run ModiFinder's MF-R likelihood computation on a known/unknown analog pair and need a scalar score (e.g. the 0.86→0.93 Kirromycin and 0.56→0.72 Naphthomycin B case studies) to quantify modification-site localization accuracy.

## When NOT to use

- When no ground-truth modification site is known for the analog pair — the Average-distance Evaluation Score requires a verified true site and is unusable for discovery-mode queries
- When evaluating ModiFinder performance using proximity cover or ambiguity cover criteria instead of the distance-based score

## Inputs

- ModiFinder atom-level modification-site likelihood distribution over the known molecule (from MF-R run)
- Ground-truth modification site annotation for the known molecule (e.g. N-methylation of Kirromycin; methylation of Naphthomycin B)
- Preprocessed MS/MS spectra (peaks <1% of base peak removed, intensities normalized to Euclidean norm 1)
- Peak-to-substructure assignment maps (initial or expert-refined)

## Outputs

- Average-distance Evaluation Score (scalar, higher = better localization)
- Pre- vs post-refinement score comparison (e.g. Kirromycin 0.86→0.93; Naphthomycin B 0.56→0.72)
- Counts of high-scoring modification sites (e.g. Naphthomycin B reduced from 24 to 2)

## How to apply

After MF-R (MAGMa combinatorial fragmentation at depth 2, 40 ppm m/z tolerance) produces the atom-level likelihood distribution for the known molecule, evaluate it against the ground-truth modification site (e.g. N-methylation of Kirromycin, methylation of Naphthomycin B) using ModiFinder's BasicEvaluationEngine. The Average-distance Evaluation Score measures the average distance between high-scoring atoms and the true modification site, so scores near 1 indicate correct localization. Preprocess spectra first per ModiFinder defaults: remove peaks below 1% of the base peak and normalize intensities to Euclidean norm 1. In the Kirromycin/Goldinodox and Naphthomycin A/B case studies, recompute the score after expert peak-to-fragment refinement (restricting 112.04/178.05 m/z and 133.07/147.08 m/z peaks to candidate substructures) to demonstrate ambiguity reduction; final post-processing sets likelihood values below 0.5 to 0 and raises values to the power of 4 before scoring.

## Related tools

- **ModiFinder** (Provides MF-R likelihood computation, BasicEvaluationEngine, and Compound/Spectrum loaders used to obtain and evaluate the Average-distance Evaluation Score) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (Combinatorial fragmentation engine underlying ModiFinder's peak-to-substructure assignment feeding the scored likelihood distribution)

## Examples

```
from modifinder import Compound, Spectrum, BasicEvaluationEngine; score = BasicEvaluationEngine().evaluate(known, unknown, true_site)
```

## Evaluation signals

- Known benchmark pairs reproduce the published baseline scores: Kirromycin-Goldinodox ≈ 0.86, Naphthomycin A-B ≈ 0.56
- After expert peak-map refinement, scores rise to ≈ 0.93 (Kirromycin) and ≈ 0.72 (Naphthomycin B)
- High-scoring modification sites for Naphthomycin B drop from 24 to 2 after refinement
- Score is bounded (higher values indicate likelihood mass concentrated near the true site); verify no negative or out-of-range values
- Spectra used for scoring show peaks <1% of base peak removed and Euclidean-normalized intensities

## Limitations

- Score requires a known true modification site, so it only validates benchmark pairs rather than scoring novel discoveries
- Raw MF-R runs can leave many atoms with high scores (ambiguity); refinement of peak-to-fragment maps depends on domain expertise about gas-phase fragmentation
- Benchmark pair eligibility requires precursor mass < 2000 Da and precursor mass difference < 50%, limiting generalization

## Evidence

- [abstract] ModiFinder synthesizes this information together and scores the likelihood for each atom in the known molecule to be the modification site.: "ModiFinder synthesizes this information together and scores the likelihood for each atom in the known molecule to be the modification site."
- [methods] all the peaks with intensities less than 1% of the base peak are removed: "all the peaks with intensities less than 1% of the base peak are removed, and the peaks are normalized to sum to a Euclidean norm of 1"
- [methods] final post-processing of likelihood scores: "any value below 0.5 is set to 0 and to further highlight the differences, especially in the high-scoring atoms, all the values are raised to the power of 4"
- [intro] an evaluation of ModiFinder's performance and limitations in identifying the modification site: [evidence span withheld — anchor status: not_found]
- [methods] If the theoretical m/z of a substructure falls within a specified m/z tolerance to the empirical m/z of a peak, then that substructure is assigned to that peak.: "If the theoretical m/z of a substructure falls within a specified m/z tolerance to the empirical m/z of a peak, then that substructure is assigned to that peak. Here, 40 ppm was chosen"
