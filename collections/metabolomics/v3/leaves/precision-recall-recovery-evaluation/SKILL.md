---
name: precision-recall-recovery-evaluation
description: Use when validating a gap-filling procedure on draft metabolic reconstructions
  by measuring how well randomly deleted reactions are recovered, comparing community-aware
  versus individual gap-filling settings.
when_to_use_negative:
- Only sensitivity and precision are computable here; if you need a full confusion
  matrix (specificity, accuracy) the removed-reaction benchmark design cannot supply
  true negatives — choose a different validation design.
- Do not use this to benchmark draft-reconstruction generation itself (e.g., comparing
  KBase, RAVEN 2.0, CarveMe, AuReMe/Pathway Tools drafts by structural distance measures);
  this skill evaluates gap-filling recovery only.
- Not for gap filling of a single unperturbed model where no reaction-removal ground
  truth exists.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_2259
tools:
- name: COMMIT
  role: Performs the community-aware conditional gap filling whose reaction recovery
    is being benchmarked
  repo: https://github.com/pwendering/COMMIT
- name: COBRA toolbox
  role: MATLAB framework within which the gap-filling LPs and reaction perturbations
    are run (v3.0)
  repo: https://github.com/opencobra/cobratoolbox
- name: IBM CPLEX
  role: LP solver for the gap-filling optimization (v12.9, tested)
- name: Gurobi
  role: Alternative LP solver for the gap-filling optimization (v9.1.1, tested; glpk
    not recommended)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:fe9fadd30154345384959ed499af0c701b0e1d2c57a2e7b7d4481c65f7993a82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COBRA toolbox
  - COMMIT
  - Gurobi
  - IBM CPLEX
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# precision-recall-recovery-evaluation

## Summary

Benchmark for gap-filling quality in which reactions are randomly removed from published species models at fixed fractions (1%, 2%, 5%, 10%) with 50 repetitions per fraction, then recovered via community-aware (COMMIT) versus individual gap filling, and scored by precision and recall only, since true negatives cannot be determined.

## When to use

Use when validating a gap-filling procedure on draft metabolic reconstructions by measuring how well randomly deleted reactions are recovered, comparing community-aware versus individual gap-filling settings.

## When NOT to use

- Only sensitivity and precision are computable here; if you need a full confusion matrix (specificity, accuracy) the removed-reaction benchmark design cannot supply true negatives — choose a different validation design.
- Do not use this to benchmark draft-reconstruction generation itself (e.g., comparing KBase, RAVEN 2.0, CarveMe, AuReMe/Pathway Tools drafts by structural distance measures); this skill evaluates gap-filling recovery only.
- Not for gap filling of a single unperturbed model where no reaction-removal ground truth exists.

## Inputs

- Published draft genome-scale metabolic reconstructions for each community member (D. vulgaris, M. maripaludis, or soil community isolates)
- Consensus reconstructions for the two soil communities (Bulgarelli and Schlaeppi)
- MNXref-based gap-filling database Universal-model-MNXref-balanced.mat
- Removal fraction schedule (1%, 2%, 5%, 10%) and number of repetitions (50 per fraction)
- COMMIT penalty weights for transport (100), metabolic (50), directionality change (25), permeable transport (50), sequence support (25), uptake of secreted metabolites (1), exchange (10^5)

## Outputs

- Per-repetition precision values (fraction of added reactions matching removed reactions)
- Per-repetition recall/sensitivity values (fraction of removed reactions recovered)
- Aggregated precision and recall per removal fraction, community-aware versus individual gap filling
- reaction_recovery_precision_recall.csv

## How to apply

Load the consensus reconstructions (e.g., the two soil communities 'Bulgarelli' and 'Schlaeppi') and the MNXref-based gap-filling database (Universal-model-MNXref-balanced.mat) into MATLAB with the COBRA toolbox v3.0. For each removal fraction (1%, 2%, 5%, 10%), randomly delete that percentage of reactions from each published species model, repeating the random removal 50 times per fraction to sample the removal distribution. For every perturbed model, run COMMIT community-aware gap filling and individual gap filling without community composition as the baseline, using IBM CPLEX v12.9 or Gurobi v9.1.1 as the LP solver (glpk is not recommended). For every repetition and removal fraction, compute precision (fraction of added reactions that match removed reactions) and recall/sensitivity (fraction of removed reactions recovered); true negatives cannot be determined, so only these two metrics are computable. Aggregate the 50 repetitions per fraction, compare community-aware versus individual gap filling, and write the aggregated values to reaction_recovery_precision_recall.csv; the correct result is a slight improvement in both precision and recall for community-aware gap filling.

## Related tools

- **COMMIT** (Performs the community-aware conditional gap filling whose reaction recovery is being benchmarked) — https://github.com/pwendering/COMMIT
- **COBRA toolbox** (MATLAB framework within which the gap-filling LPs and reaction perturbations are run (v3.0)) — https://github.com/opencobra/cobratoolbox
- **IBM CPLEX** (LP solver for the gap-filling optimization (v12.9, tested))
- **Gurobi** (Alternative LP solver for the gap-filling optimization (v9.1.1, tested; glpk not recommended))

## Examples

```
git clone https://github.com/pwendering/COMMIT
```

## Evaluation signals

- Precision and recall are computable for every combination of removal fraction (1%, 2%, 5%, 10%) and repetition (50 per fraction), with only these two metrics since true negatives cannot be determined.
- Aggregated community-aware results show slightly higher precision AND recall than individual gap filling at each removal fraction.
- Results are written in the expected schema to reaction_recovery_precision_recall.csv.
- The LP solver in use is IBM CPLEX v12.9 or Gurobi v9.1.1 (not glpk), as glpk is explicitly not recommended.
- Each removed-reaction set is sampled randomly with 50 repetitions per fraction, matching the reported sampling design.

## Limitations

- Since the true negatives could not be determined, only sensitivity and precision can be computed — specificity/accuracy are undefined under this benchmark.
- COMMIT's improvement over individual gap filling in this test is slight in both precision and recall, so results may be sensitive to the random seeds or solver choice.
- Gap-filling penalty weights used in the study were tuned ad hoc until satisfying results were obtained for selected reconstructions, with no systematic selection procedure documented.
- glpk is not recommended as LP solver for this workflow; use IBM CPLEX or Gurobi.

## Evidence

- [discussion] Only sensitivity and precision could be computed since true negatives could not be determined: "Since the true negatives could not be determined, only sensitivity and precision were calculated as measures to assess the quality of model prediction."
- [intro] COMMIT reduced gap-filling reactions relative to individual gap filling: "COMMIT decreased the number of added reactions compared to the individual gap filling of the reconstructions"
- [readme] Repository scope: consensus reconstructions and community-dependent gap-filling for two soil communities: "This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions for two soil communities (Bulgarelli et al."
- [readme] COMMIT repo requirements: MATLAB, COBRA toolbox v3.0, LP solvers CPLEX/Gurobi: "Requirements for COMMIT
- Matlab (tested with versions R2017b and R2020b)
- [COBRA toolbox v3.0](https://github.com/opencobra/cobratoolbox)
- LP solver for the COBRA toolbox, tested with 
  - [IBM"
