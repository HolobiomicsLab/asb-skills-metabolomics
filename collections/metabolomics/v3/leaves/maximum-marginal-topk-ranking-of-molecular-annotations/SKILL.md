---
name: maximum-marginal-topk-ranking-of-molecular-annotations
description: Use when a ready-to-identify dataset contains candidate structures with
  MS/MS scores and ROASMI retention-order probabilities P_ij, and you need top-k ranked
  annotations per peak, especially for isomers with similar fragmentation or peaks
  lacking informative spectra.
when_to_use_negative:
- Input dataset lacks ROASMI-predicted retention-order probabilities P_ij (e.g. no
  trained ROASMI model or no ready-to-identify set) — the integration graph cannot
  be built.
- The chromatographic system is not reversed-phase LC at eluent pH ≈ 2.7 — ROASMI
  only applies to RPLC, and HILIC generalizability is future work.
- A candidate pair is marked indistinguishable by ROASMI (close retention with high
  variance) — such pairs must be excluded rather than ranked.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: ROASMI
  role: Supplies predicted retention-order probabilities P_ij that are assigned to
    directed edges of the fully connected candidate graph before integration
  repo: https://github.com/FangYuan717/ROASMI
- name: msms_rt_score_integration
  role: Implements the Eric Bach et al. probabilistic framework that uses random spanning-tree
    ensembles to compute maximum-marginal top-k ranking scores combining MS/MS and
    retention-order evidence
  repo: https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- name: chemprop
  role: Upstream molecular embedding module (D-MPNN) whose extended implementation
    within ROASMI produces the molecular representation feeding retention-score prediction
  repo: https://github.com/chemprop/chemprop
- name: descriptastorus
  role: Supports descriptor generation for molecular features used by ROASMI's embedding
    pipeline
  repo: https://github.com/bp-kelley/descriptastorus
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:f7abb77aa298b9daf50e42205a8a2f65bb8b005804eef371b079b873770ab199
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - ROASMI
  - chemprop
  - descriptastorus
  - msms_rt_score_integration
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# maximum-marginal-topk-ranking-of-molecular-annotations

## Summary

Integrates ROASMI-predicted retention-order probabilities with MS/MS spectral scores to rank candidate molecular structures for each detected peak, using a probabilistic framework over fully connected graphs of candidates with random spanning-tree ensembles.

## When to use

Use when a ready-to-identify dataset contains candidate structures with MS/MS scores and ROASMI retention-order probabilities P_ij, and you need top-k ranked annotations per peak, especially for isomers with similar fragmentation or peaks lacking informative spectra.

## When NOT to use

- Input dataset lacks ROASMI-predicted retention-order probabilities P_ij (e.g. no trained ROASMI model or no ready-to-identify set) — the integration graph cannot be built.
- The chromatographic system is not reversed-phase LC at eluent pH ≈ 2.7 — ROASMI only applies to RPLC, and HILIC generalizability is future work.
- A candidate pair is marked indistinguishable by ROASMI (close retention with high variance) — such pairs must be excluded rather than ranked.

## Inputs

- Ready-to-identify dataset CSV with candidate structures and MS/MS spectral scores per detected peak (e.g. data/identify_toy_set.csv)
- ROASMI-predicted retention-order probabilities P_ij for each pair of candidate structures (from the ROASMI retention prediction module)
- Scalar parameter D controlling the retention-vs-spectra weight balance (supplied via args.py / IdentifyArgs)

## Outputs

- identified_annotations.csv containing ranked candidate annotations per peak, with candidate identity, integrated score, and per-peak scores

## How to apply

Load the ready-to-identify set (e.g. data/identify_toy_set.csv) containing candidate structures with MS/MS spectral scores per detected peak plus ROASMI-predicted retention-order probabilities P_ij for each candidate pair. Build a fully connected graph over all candidates per peak where each node carries its MS/MS score and each directed edge carries the ROASMI P_ij retention-order probability. Run the Eric Bach et al. probabilistic integration framework (aalto-ics-kepaco/msms_rt_score_integration) with random spanning-tree ensembles to infer the maximum probability of ranking the top-k candidates, sweeping the scalar retention-vs-spectra weight parameter D supplied in args.py (IdentifyArgs). Aggregate the spanning-tree results to obtain posterior ranking scores, select top-k candidate annotations per peak, and exclude candidate pairs ROASMI marks as indistinguishable (e.g. 81% probability with 9% variance for close-retention pairs). Write ranked annotations to identified_annotations.csv with candidate identity, integrated score, and per-peak scores.

## Related tools

- **ROASMI** (Supplies predicted retention-order probabilities P_ij that are assigned to directed edges of the fully connected candidate graph before integration) — https://github.com/FangYuan717/ROASMI
- **msms_rt_score_integration** (Implements the Eric Bach et al. probabilistic framework that uses random spanning-tree ensembles to compute maximum-marginal top-k ranking scores combining MS/MS and retention-order evidence) — https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- **chemprop** (Upstream molecular embedding module (D-MPNN) whose extended implementation within ROASMI produces the molecular representation feeding retention-score prediction) — https://github.com/chemprop/chemprop
- **descriptastorus** (Supports descriptor generation for molecular features used by ROASMI's embedding pipeline) — https://github.com/bp-kelley/descriptastorus

## Examples

```
python code/Identify.py --pred_path data/identify_toy_set.csv
```

## Evaluation signals

- The Identify.py script runs to completion without errors on the ready-to-identify dataset.
- identified_annotations.csv contains a ranked annotation entry for every detected peak present in the input identify_toy_set.csv.
- Candidate pairs flagged as indistinguishable by ROASMI (e.g. retention-order probability ≈ 81% with variance ≈ 9%) are excluded from the final ranked output.
- The integrated score reflects the balance controlled by parameter D: as D is swept, the relative weight of retention-order vs MS/MS evidence in the top-k ranking shifts accordingly.
- Top-k results for coexisting isomers with similar fragmentation patterns show improved discrimination compared to MS/MS-only scoring.

## Limitations

- ROASMI applies only to conventional compounds within the elemental restrictions of C, H, N, O, P, and S, leaving a vast chemical space yet to be explored.
- ROASMI currently applies only to RPLC systems; constructing HILIC retention models with generalizability is future work.
- When chromatographic peaks of two compounds are close enough together, their retention order may not be consistent across experiments, making the pair indistinguishable.
- RT outliers resulting from slight changes in experimental conditions are unresolved and require in-depth mechanistic study.
- Machine learning approaches are imperfect and require integration with solid mechanistic understanding.

## Evidence

- [methods] Score integration is modeled as inference on a fully connected graph over candidate structures, using random spanning-tree ensembles to infer the maximum probability of ranking the top-k candidates for each peak, with the full vector z corresponding to the molecular candidates.: [evidence span withheld — anchor status: not_found]
- [methods] Retention and MS/MS scores are combined by running Identify.py after predicting retention scores for each compound in the ready-to-identify datasets.: "we integrate predicted retention scores with MS/MS scores by running: python code/Identify.py"
- [readme] The msms_rt_score_integration repository provides the scripts used for the probabilistic framework integrating mass spectrum and retention time information.: [evidence span withheld — anchor status: external_doc]
- [readme] The msms_rt_score_integration code is developed for Python >= 3.6 with dependencies including numpy, scipy, pandas, scikit-learn, joblib, matplotlib, seaborn, networkx and setuptools.: [evidence span withheld — anchor status: external_doc]
- [abstract] ROASMI helps distinguish coexisting isomers with similar fragmentation patterns and annotate detection peaks without informative MS/MS spectra.: "The application of ROASMI to four real-world datasets demonstrates its advantages in distinguishing coexisting isomers with similar fragmentation patterns and in annotating detection peaks without"
