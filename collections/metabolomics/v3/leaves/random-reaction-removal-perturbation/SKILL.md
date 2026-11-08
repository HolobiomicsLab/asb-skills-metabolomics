---
name: random-reaction-removal-perturbation
description: Use when you need to benchmark a gap-filling approach (e.g., COMMIT vs.
  individual gap filling) by injecting controlled corruption—randomly removed reactions—into
  published species models and scoring recovery by precision and recall.
when_to_use_negative:
- The models are already complete and you only need to compare their reaction sets
  directly—no perturbation or gap-filling benchmark is needed.
- The question is whether a model can grow on a defined medium rather than how well
  gap filling recovers removed reactions—use growth/feasibility simulations instead.
- You only have a single-species draft model with no community composition available—COMMIT's
  community-aware advantage cannot be exercised, so individual gap filling is the
  appropriate target.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: COMMIT
  role: Performs the community-aware conditional gap filling on each perturbed model,
    then its added-reaction set is scored against the randomly removed reactions.
  repo: https://github.com/pwendering/COMMIT
- name: COBRA toolbox
  role: MATLAB toolbox (v3.0) providing the constraint-based modeling infrastructure
    for loading models and running gap filling.
  repo: https://github.com/opencobra/cobratoolbox
- name: IBM CPLEX
  role: LP solver (v12.9, tested) used to solve the gap-filling optimization for each
    perturbed model; glpk is not recommended.
- name: Gurobi
  role: Alternative LP solver (v9.1.1, tested) for the gap-filling optimization.
- name: RAVEN 2.0
  role: One of the reconstruction pipelines whose published species models serve as
    the starting models for the perturbation experiments.
  repo: https://github.com/SysBioChalmers/RAVEN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:5d5cccd9ca18abdaca6cd1384ed12e5b6ce639ee819899c03aa23a673a01b0c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COBRA toolbox
  - COMMIT
  - Gurobi
  - IBM CPLEX
  - RAVEN 2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# random-reaction-removal-perturbation

## Summary

A validation protocol that randomly deletes a known fraction of reactions (1%, 2%, 5%, 10%, repeated 50 times per fraction) from published species models, then measures how well a gap-filling method recovers the removed reactions using precision and recall. Used here to show that COMMIT's community-aware gap filling slightly outperforms individual gap filling on the D. vulgaris / M. maripaludis two-species community.

## When to use

Use when you need to benchmark a gap-filling approach (e.g., COMMIT vs. individual gap filling) by injecting controlled corruption—randomly removed reactions—into published species models and scoring recovery by precision and recall.

## When NOT to use

- The models are already complete and you only need to compare their reaction sets directly—no perturbation or gap-filling benchmark is needed.
- The question is whether a model can grow on a defined medium rather than how well gap filling recovers removed reactions—use growth/feasibility simulations instead.
- You only have a single-species draft model with no community composition available—COMMIT's community-aware advantage cannot be exercised, so individual gap filling is the appropriate target.

## Inputs

- Published/perturbed species models for the two-species community (Desulfovibrio vulgaris and Methanococcus maripaludis)
- Community composition for the two species (required for COMMIT community-aware gap filling)
- Random removal fractions: 1%, 2%, 5%, 10% of reactions per model
- 50 random removal repetitions per fraction (random seeds for reproducibility)
- MATLAB environment with COBRA toolbox v3.0 and LP solver (IBM CPLEX v12.9 or Gurobi v9.1.1)

## Outputs

- reaction_recovery_precision_recall.csv with per-repetition precision and recall values
- Aggregated precision/recall (sensitivity) values per removal fraction for community-aware vs. individual gap filling
- Comparison statistics showing the reported slight improvement in both metrics for community-aware gap filling

## How to apply

For each removal fraction (1%, 2%, 5%, 10%), randomly delete that percentage of reactions from each published species model, repeating the random removal 50 times per fraction to average over stochastic variation. For each perturbed model, run the community-aware COMMIT gap filling (code/matlab/gap_filling/run_iterative_gap_filling.m) and, in parallel, individual gap filling without community composition (code/matlab/gap_filling/gap_fill_individual_models.m), using MATLAB with the COBRA toolbox v3.0 and IBM CPLEX v12.9 or Gurobi v9.1.1 as the LP solver (glpk is not recommended). For every repetition and fraction, compute precision (fraction of added reactions that match removed reactions) and recall/sensitivity (fraction of removed reactions recovered); true negatives cannot be determined, so these two metrics are the only computable measures. Aggregate the 50 repetitions per fraction and compare community-aware versus individual gap filling, writing results to reaction_recovery_precision_recall.csv.

## Related tools

- **COMMIT** (Performs the community-aware conditional gap filling on each perturbed model, then its added-reaction set is scored against the randomly removed reactions.) — https://github.com/pwendering/COMMIT
- **COBRA toolbox** (MATLAB toolbox (v3.0) providing the constraint-based modeling infrastructure for loading models and running gap filling.) — https://github.com/opencobra/cobratoolbox
- **IBM CPLEX** (LP solver (v12.9, tested) used to solve the gap-filling optimization for each perturbed model; glpk is not recommended.)
- **Gurobi** (Alternative LP solver (v9.1.1, tested) for the gap-filling optimization.)
- **RAVEN 2.0** (One of the reconstruction pipelines whose published species models serve as the starting models for the perturbation experiments.) — https://github.com/SysBioChalmers/RAVEN

## Examples

```
git clone https://github.com/pwendering/COMMIT
```

## Evaluation signals

- Both precision and recall should show a slight improvement for COMMIT's community-aware gap filling over individual gap filling when aggregated over 50 repetitions per fraction.
- Only sensitivity (recall) and precision are computable—true negatives cannot be determined for randomly removed reactions—so the reported metrics must be limited to these two.
- The results table must span all four removal fractions (1%, 2%, 5%, 10%) with 50 repetitions each, covering both gap-filling modes.
- Gap filling must be run with a tested LP solver (IBM CPLEX v12.9 or Gurobi v9.1.1); runs with glpk are not considered reliable.
- Every recovered reaction added by gap filling must be matched against the exact list of randomly removed reactions for that repetition to compute precision.

## Limitations

- Since the true negatives could not be determined, only sensitivity and precision could be calculated as measures to assess the quality of model prediction.
- The COMMIT gap-filling penalty weights were tuned ad hoc (altered until satisfying results were obtained for selected reconstructions) rather than by a systematic selection procedure.
- The full specification of the COMMIT procedure is deferred to supplementary S1 Text and is not present in the main text.
- The improvement in precision and recall for community-aware gap filling is slight rather than large, so the validation may not detect meaningful differences in small perturbation sets without sufficient repetitions.

## Evidence

- [discussion] Only sensitivity and precision could be computed since true negatives could not be determined: "Since the true negatives could not be determined, only sensitivity and precision were calculated as measures to assess the quality of model prediction."
- [readme] COMMIT repository provides the community gap-filling code: "This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions"
- [intro] COMMIT decreased added reactions vs individual gap filling: "COMMIT decreased the number of added reactions compared to the individual gap filling of the reconstructions"
- [readme] Solver requirement for the COBRA toolbox gap-filling runs: "LP solver for the COBRA toolbox, tested with - [IBM CPLEX v12.9](https://www.ibm.com/analytics/cplex-optimizer)"
- [readme] COBRA toolbox is the MATLAB infrastructure for constraint-based modeling: [evidence span withheld — anchor status: not_found]
