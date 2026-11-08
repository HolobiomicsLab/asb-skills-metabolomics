---
name: rank-at-k-metric-computation
description: Reference for evaluating molecular retrieval models on MassSpecGym when
  you have per-query ranked candidate lists and need rank@1 scores under the by-mass
  and by-formula candidate conditions.
when_to_use_negative:
- When the task is de novo molecule generation or spectrum simulation rather than
  retrieval/ranking of candidate structures — MassSpecGym defines these as separate
  challenges with different metrics.
- When aggregating rankings from constituent spectra for molecules that do not have
  exactly three spectra (the rank-aggregation comparison is limited to the 1,531 test
  molecules with exactly three spectra, one per collision energy level).
- When evaluating on custom candidate pools not derived from MassSpecGym's by-mass/by-formula
  retrieval conditions, since the published rank@1 baselines (35.99%/13.96% consensus,
  26.37%/11.10% individual) assume those conditions.
edam_operation: http://edamontology.org/operation_3452
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0219
tools:
- name: MassSpecGym
  role: Provides the benchmark dataset, predefined train/validation/test splits, the
    two retrieval challenge conditions (candidates by mass and by formula), and the
    evaluation metrics used for rank@1 computation via the RetrievalMassSpecGymModel
    abstract class.
  repo: https://github.com/pluskal-lab/MassSpecGym
- name: MVP
  role: The model being evaluated; produces ranked candidate lists via its four ranking
    views (mol-s, fp-s, mol-cs, fp-cs), which are then scored with rank@1 on MassSpecGym.
  repo: github.com/HassounLab/MVP
- name: Lightning
  role: Underlying framework used by MassSpecGym to structure the retrieval evaluation
    step and metric aggregation in RetrievalMassSpecGymModel.
- name: PyTorch
  role: Framework in which MVP and MassSpecGym models are implemented, generating
    the per-query ranked candidate scores that feed rank@1 computation.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:b91ae6c17cd47b1d8d72410f4acd327ffd40865e2027431d4f88589def99a41e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/MassSpecGym
  tools:
  - Lightning
  - MVP
  - MassSpecGym
  - PyTorch
  edam_operation: http://edamontology.org/operation_3452
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# rank-at-k-metric-computation

## Summary

Computes rank@1 retrieval metrics for molecular candidate ranking on the MassSpecGym benchmark, where a model scores a ranked list of candidate structures for each query MS/MS spectrum and the metric measures whether the ground-truth molecule is ranked first. It is the primary evaluation metric used to compare MVP against MIST, ESP, and JESTR for both candidates-by-mass and candidates-by-formula conditions.

## When to use

Reference for evaluating molecular retrieval models on MassSpecGym when you have per-query ranked candidate lists and need rank@1 scores under the by-mass and by-formula candidate conditions.

## When NOT to use

- When the task is de novo molecule generation or spectrum simulation rather than retrieval/ranking of candidate structures — MassSpecGym defines these as separate challenges with different metrics.
- When aggregating rankings from constituent spectra for molecules that do not have exactly three spectra (the rank-aggregation comparison is limited to the 1,531 test molecules with exactly three spectra, one per collision energy level).
- When evaluating on custom candidate pools not derived from MassSpecGym's by-mass/by-formula retrieval conditions, since the published rank@1 baselines (35.99%/13.96% consensus, 26.37%/11.10% individual) assume those conditions.

## Inputs

- Ranked candidate lists per query MS/MS spectrum (candidate molecular structures ordered by model score)
- Ground-truth molecular structure labels for each MassSpecGym test query
- Two candidate pools per query: candidates retrieved by mass and candidates retrieved by formula

## Outputs

- rank@1 percentage for each ranking view (mol-s, fp-s, mol-cs, fp-cs)
- Separate rank@1 values for candidates-by-mass and candidates-by-formula conditions
- A benchmark table aggregating rank@1 across views and conditions for model comparison (e.g. MVP vs MIST vs ESP vs JESTR)

## How to apply

After a trained model (e.g. MVP, evaluated via test.py with params_formSpec.yaml or params_binnedSpec.yaml) ranks molecular candidates for each test spectrum, compute the rank of the ground-truth molecule within the ranked candidate list and record the fraction of queries where it is ranked first (rank@1). Do this separately for the two candidate-retrieval conditions: candidates retrieved by mass and candidates retrieved by formula, and separately for each ranking view the model exposes (mol-s, fp-s, mol-cs, fp-cs) so the effect of the consensus-spectrum view can be isolated. For MVP, results show rank@1 of 35.99% (consensus view, candidates by mass) and 13.96% (consensus view, candidates by formula), versus 26.37% and 11.10% for the individual-spectrum view; when rank-aggregating over constituent spectra, restrict the evaluation to test molecules that have exactly three spectra (1,531 cases) so aggregation is comparable across queries. Validate the computed numbers against these published values and report all metrics per view and per candidate condition when comparing against baselines like MIST, ESP, or JESTR.

## Related tools

- **MassSpecGym** (Provides the benchmark dataset, predefined train/validation/test splits, the two retrieval challenge conditions (candidates by mass and by formula), and the evaluation metrics used for rank@1 computation via the RetrievalMassSpecGymModel abstract class.) — https://github.com/pluskal-lab/MassSpecGym
- **MVP** (The model being evaluated; produces ranked candidate lists via its four ranking views (mol-s, fp-s, mol-cs, fp-cs), which are then scored with rank@1 on MassSpecGym.) — github.com/HassounLab/MVP
- **Lightning** (Underlying framework used by MassSpecGym to structure the retrieval evaluation step and metric aggregation in RetrievalMassSpecGymModel.)
- **PyTorch** (Framework in which MVP and MassSpecGym models are implemented, generating the per-query ranked candidate scores that feed rank@1 computation.)

## Examples

```
python test.py --param_pth params_formSpec.yaml  # rank candidates, then compute rank@1 per view (mol-s/fp-s/mol-cs/fp-cs) for candidates by mass and by formula
```

## Evaluation signals

- Reproduced rank@1 within tolerance of the published MVP values: 35.99% (consensus spectrum view, candidates by mass) and 13.96% (consensus spectrum view, candidates by formula).
- Reproduced rank@1 within tolerance of 26.37% and 11.10% for the individual-spectrum (mol-s) view on candidates by mass and by formula, respectively.
- Per-query rank values computed for both candidate conditions (by mass and by formula) so that relative improvements (e.g. MVP outperforming MIST by 145.83% and JESTR by 130.71% for rank@1 with candidates by mass) can be verified.
- Rank-aggregation evaluation restricted to test molecules with exactly three spectra (1,531 cases) before computing aggregated rank@1.
- A single benchmark table is produced listing rank@1 for each ranking view under each candidate condition, matching the structure of the published comparison.

## Limitations

- The metric only captures whether the correct molecule is ranked first; other retrieval metrics (e.g. median rank, recall@k) are reported alongside in the full benchmark but rank@1 alone is sensitive to small model-score changes near the top of the list.
- Published numbers assume MassSpecGym's predefined splits and candidate pools; results are not directly comparable to runs on custom splits or custom candidate sets.
- For the rank-aggregation comparison, results are limited to the subset of test molecules with exactly three spectra (1,531 cases), so aggregate metrics are not computed over the full test set.
- MVP reports that losses that prioritize ranking molecular candidates are not currently included, which may cap achievable rank@1 improvements.

## Evidence

- [results] MVP achieves 35.99% and 13.96% rank@1 using the consensus spectrum view for candidates by mass and formula, respectively, and 26.37% and 11.10% rank@1 using individual spectra: [evidence span withheld — anchor status: not_found]
- [readme] Molecule retrieval (MS/MS spectrum → ranked list of candidate molecular structures): [evidence span withheld — anchor status: external_doc]
- [results] we limit the evaluation to a subset of the test set for which a test molecule has exactly three spectra: "we limit the evaluation to a subset of the test set for which a test molecule has exactly three spectra"
- [intro] for candidates by mass, MVP outperforms MIST by 145.83% and JESTR by 130.71% for rank@1: "for candidates by mass, MVP outperforms MIST by 145.83% and JESTR by 130.71% for rank@1."
- [readme] MassSpecGym provides three challenges for benchmarking the discovery and identification of new molecules from MS/MS spectra: [evidence span withheld — anchor status: external_doc]
- [results] MVP with the mol-cs ranking view outperforms other tools across all metrics for both candidates by mass and formula: "MVP with the mol-cs ranking view outperforms other tools across all metrics for both candidates by mass and formula"
