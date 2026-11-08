---
name: spanning-tree-ensemble-posterior-computation
description: Use when candidate structures for a detected peak already have both MS/MS
  spectral scores and ROASMI retention-order probabilities P_ij, and you need a posterior-ranked
  top-k annotation list per peak.
when_to_use_negative:
- Input compounds lie outside the ROASMI applicability domain — only conventional
  compounds composed of C, H, N, O, P, S are covered; broader chemical space is unexplored
- Chromatography is not reversed-phase liquid chromatography (RPLC) at eluent pH ≈
  2.7 — ROASMI currently applies only to RPLC systems, and HILIC generalizability
  is future work
- You only need MS/MS spectral matching without retention-order information — this
  skill specifically integrates retention scores and is redundant if no ROASMI P_ij
  values are available
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: ROASMI
  role: Provides the predicted retention-order probabilities P_ij (and associated
    variance for distinguishing detectable pairs) that serve as edge weights on the
    fully connected graph over candidate structures
  repo: https://github.com/FangYuan717/ROASMI
- name: msms_rt_score_integration
  role: Implements the Eric Bach et al. probabilistic integration framework that performs
    random spanning-tree ensemble inference over the graph to compute maximum-marginal
    top-k ranking scores combining MS/MS and retention information
  repo: https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- name: chemprop
  role: Underlying molecular embedding module (directed message passing neural network)
    used by ROASMI to produce the chemical-space representations from which retention-order
    predictions are derived
  repo: https://github.com/chemprop/chemprop
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:6563596426b2a07b18c0327c15f44f8cab8ead1d73dc063db0f8128e8a640fd3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - ROASMI
  - chemprop
  - msms_rt_score_integration
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spanning-tree-ensemble-posterior-computation

## Summary

Integration of ROASMI predicted retention-order probabilities P_ij with MS/MS spectral scores as probabilistic inference on a fully connected graph over candidate molecular structures, using random spanning tree ensembles (Eric Bach et al. framework) to compute maximum-marginal top-k ranking scores for each detected peak.

## When to use

Use when candidate structures for a detected peak already have both MS/MS spectral scores and ROASMI retention-order probabilities P_ij, and you need a posterior-ranked top-k annotation list per peak.

## When NOT to use

- Input compounds lie outside the ROASMI applicability domain — only conventional compounds composed of C, H, N, O, P, S are covered; broader chemical space is unexplored
- Chromatography is not reversed-phase liquid chromatography (RPLC) at eluent pH ≈ 2.7 — ROASMI currently applies only to RPLC systems, and HILIC generalizability is future work
- You only need MS/MS spectral matching without retention-order information — this skill specifically integrates retention scores and is redundant if no ROASMI P_ij values are available

## Inputs

- data/identify_toy_set.csv - ready-to-identify dataset of candidate structures with MS/MS spectral scores for each detected peak
- ROASMI-predicted retention-order probabilities P_ij for each pair of candidate structures (from the ROASMI retention prediction module, output of code/ROASMI_predict.py)
- Scalar parameter D (retention-vs-spectra weight balance), set in args.py / IdentifyArgs

## Outputs

- identified_annotations.csv - final ranked annotations with top-k candidate identity, integrated score, and per-peak scores
- Top-k candidate annotations per detected peak
- Exclusion of candidate pairs marked indistinguishable by ROASMI variance analysis

## How to apply

Load the ready-to-identify dataset 'data/identify_toy_set.csv' with candidate structures carrying MS/MS spectral scores per detected peak plus ROASMI-predicted retention-order probabilities P_ij from the retention prediction module. For each peak, construct a fully connected graph over candidate structures where nodes carry MS/MS scores and directed edges carry ROASMI P_ij retention-order probabilities. Run the Eric Bach et al. probabilistic integration framework (aalto-ics-kepaco/msms_rt_score_integration) which uses random spanning tree ensembles to infer the maximum probability of ranking the top-k candidates, sweeping the scalar parameter D supplied in args.py (IdentifyArgs) to control the retention-vs-spectra weight balance. Aggregate spanning-tree results into posterior ranking scores and select the top-k candidate annotations per peak, then exclude candidate pairs ROASMI marks as indistinguishable (very close retention with high variance, e.g. 81% probability and 9% variance or closer) from the final ranked output written to identified_annotations.csv.

## Related tools

- **ROASMI** (Provides the predicted retention-order probabilities P_ij (and associated variance for distinguishing detectable pairs) that serve as edge weights on the fully connected graph over candidate structures) — https://github.com/FangYuan717/ROASMI
- **msms_rt_score_integration** (Implements the Eric Bach et al. probabilistic integration framework that performs random spanning-tree ensemble inference over the graph to compute maximum-marginal top-k ranking scores combining MS/MS and retention information) — https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- **chemprop** (Underlying molecular embedding module (directed message passing neural network) used by ROASMI to produce the chemical-space representations from which retention-order predictions are derived) — https://github.com/chemprop/chemprop

## Examples

```
python code/Identify.py
```

## Evaluation signals

- The script code/Identify.py runs to completion on the input identify_toy_set.csv without errors
- identified_annotations.csv contains ranked annotations for every peak present in the input set
- Each peak's annotation includes a top-k candidate identity, an integrated score, and per-peak scores consistent with the graph-based integration framework
- Candidate pairs that ROASMI flags as indistinguishable (e.g. very close retention with high variance) are excluded from the final ranked output
- Varying the scalar parameter D produces the expected trade-off in ranking between retention-order and MS/MS spectral information (posterior scores shift as D is swept)

## Limitations

- ROASMI applies only to conventional compounds within elemental restrictions of C, H, N, O, P, and S; broader chemical space unexplored
- ROASMI currently applies only to RPLC systems (eluent pH ≈ 2.7); HILIC retention model construction with generalizability remains future work
- Model cannot distinguish certain molecular pairs with close retention (e.g., 4-hydroxybenzoic acid and 2,5-dihydroxybenzoic acid) — such indistinguishable pairs must be excluded from downstream identification
- Retention order of very close chromatographic peaks may not be consistent across experiments; RT outliers from slight experimental condition changes are unresolved and require further mechanistic study
- The Eric Bach et al. integration framework was developed and tested in a Linux environment; Windows and MacOS are not officially supported

## Evidence

- [methods] Graph-based integration framework with spanning tree ensembles: "we integrate predicted retention scores with MS/MS scores by running: python code/Identify.py"
- [readme] Spanning tree ensemble computation in the Bach framework: [evidence span withheld — anchor status: external_doc]
- [discussion] Indistinguishable pair exclusion criterion: "when the chromatographic peaks of two compounds are close enough together, their order of retention may not be consistent across experiments."
- [discussion] ROASMI variance as distinguishing signal: "ROASMI predicts that peak 1 elutes before peak 2 with an 81% probability and a variance of 9%."
