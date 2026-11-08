---
name: massspectrometry-benchmark-metric-computation
description: Use when comparing molecular-candidate ranking strategies on MS/MS benchmark
  data (e.g., MassSpecGym retrieval task) and you must compute rank@1/rank@k for aggregate-then-rank
  vs rank-then-aggregate designs.
when_to_use_negative:
- The test molecule has fewer or more than exactly three spectra — the rank-aggregation
  comparison is limited to the 1,531-case subset with exactly three spectra
- You are evaluating spectrum simulation or de novo generation challenges rather than
  the molecule-retrieval ranking task
- Spectra whose molecules appear in CFM-ID's training set must be excluded for a fair
  CFM-ID comparison — raw metric computation over the full test set would be unfair
  for that baseline
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0121
tools:
- name: MVP
  role: Pretrained model that ranks molecular candidates from the consensus spectrum
    (mol-cs view) and from individual constituent spectra for the two comparison strategies
  repo: github.com/HassounLab/MVP
- name: MassSpecGym
  role: Benchmark dataset providing the test split, candidate sets by mass and formula,
    and retrieval-challenge evaluation metrics
  repo: https://github.com/pluskal-lab/MassSpecGym
- name: lightning
  role: PyTorch Lightning framework underlying MassSpecGym's RetrievalMassSpecGymModel
    evaluation classes used to run test.py
  repo: https://github.com/Lightning-AI/pytorch-lightning
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:6e455ed3d332049ea879af13321a5099246814aacfcddbf49e1a3c309958c064
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MVP
  - MassSpecGym
  - lightning
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# massspectrometry-benchmark-metric-computation

## Summary

Computes candidate-retrieval metrics (rank@1, rank@k) on the MassSpecGym benchmark to compare annotation strategies, specifically consensus-spectrum (aggregate-then-rank) ranking versus per-spectrum rank-then-aggregate for molecules with multiple query spectra.

## When to use

Use when comparing molecular-candidate ranking strategies on MS/MS benchmark data (e.g., MassSpecGym retrieval task) and you must compute rank@1/rank@k for aggregate-then-rank vs rank-then-aggregate designs.

## When NOT to use

- The test molecule has fewer or more than exactly three spectra — the rank-aggregation comparison is limited to the 1,531-case subset with exactly three spectra
- You are evaluating spectrum simulation or de novo generation challenges rather than the molecule-retrieval ranking task
- Spectra whose molecules appear in CFM-ID's training set must be excluded for a fair CFM-ID comparison — raw metric computation over the full test set would be unfair for that baseline

## Inputs

- Preprocessed MassSpecGym test set (spectra + candidate sets by mass and by formula)
- MVP pretrained model checkpoint trained on MassSpecGym
- params_formSpec.yaml or params_binnedSpec.yaml configuration file with dataset paths
- Preprocessed fingerprints and consensus spectra files from MVP data_preprocess.py

## Outputs

- Structured CSV table reporting rank@1 (and rank@k) for consensus-spectrum annotation vs aggregated per-spectrum rankings, per candidate-set type (mass, formula)
- Per-molecule ranking lists from the mol-cs and rank-then-aggregate views

## How to apply

Load the preprocessed MassSpecGym test set and restrict to the subset where a test molecule has exactly three spectra (1,531 cases), since this is the only setting where consensus-spectrum vs rank-aggregation comparison is defined. For each query molecule, run candidate ranking with MVP's pretrained checkpoint via test.py (params_formSpec.yaml or params_binnedSpec.yaml) using the mol-cs consensus-spectrum ranking view, then run ranking on each of the three constituent spectra individually and aggregate per-spectrum rankings (e.g., average rank and reciprocal rank). Compute rank@1 and rank@k for both strategies on the identical molecule subset so the comparison is paired. Expected published reference values: mol-cs achieves 35.99% rank@1 by mass and 13.96% by formula, versus 26.37% and 11.10% for per-spectrum aggregation. Emit the comparison as a structured CSV table keyed by strategy and candidate-set type (mass vs formula).

## Related tools

- **MVP** (Pretrained model that ranks molecular candidates from the consensus spectrum (mol-cs view) and from individual constituent spectra for the two comparison strategies) — github.com/HassounLab/MVP
- **MassSpecGym** (Benchmark dataset providing the test split, candidate sets by mass and formula, and retrieval-challenge evaluation metrics) — https://github.com/pluskal-lab/MassSpecGym
- **lightning** (PyTorch Lightning framework underlying MassSpecGym's RetrievalMassSpecGymModel evaluation classes used to run test.py) — https://github.com/Lightning-AI/pytorch-lightning

## Examples

```
python test.py --param_pth params_formSpec.yaml
```

## Evaluation signals

- Rank@1 for mol-cs on the three-spectrum subset reproduces 35.99% (mass candidates) and 13.96% (formula candidates); rank-then-aggregate reproduces 26.37% and 11.10%
- The evaluated subset contains exactly 1,531 test molecules, each with exactly three constituent spectra
- Both strategies are scored on the identical paired molecule subset — no molecule appears in one strategy's evaluation but not the other's
- Output CSV separates results by candidate-set type (by mass vs by formula) and by ranking strategy (mol-cs vs average/reciprocal rank aggregation)
- Rankings generated via test.py run with params_formSpec.yaml or params_binnedSpec.yaml load a valid pretrained checkpoint without data-path errors

## Limitations

- Evaluation is restricted to the subset of test molecules with exactly three spectra, so results do not generalize to molecules with 1, 2, or >3 spectra
- Preprocessed MassSpecGym download URLs are empty in the MVP README ('wget' commands with no targets), so preprocessed data may need to be regenerated locally
- MVP's rank-aggregation baselines use only average rank and reciprocal rank; other aggregation schemes are not covered by this comparison

## Evidence

- [abstract] annotation using query consensus spectra significantly outperforms rank aggregation strategies based on constituent spectrum annotation: "annotation using query consensus spectra significantly outperforms rank aggregation strategies based on constituent spectrum annotation"
- [results] we limit the evaluation to a subset of the test set for which a test molecule has exactly three spectra: "we limit the evaluation to a subset of the test set for which a test molecule has exactly three spectra"
- [abstract] MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively: "MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively"
- [intro] an aggregate-then-rank strategy, which combines spectral information prior to candidate ranking, and a rank-then-aggregate: "an aggregate-then-rank strategy, which combines spectral information prior to candidate ranking, and a rank-then-aggregate"
- [readme] Our model is trained on [MassSpecGym dataset](https://github.com/pluskal-lab/MassSpecGym).: "Our model is trained on [MassSpecGym dataset](https://github.com/pluskal-lab/MassSpecGym)."
