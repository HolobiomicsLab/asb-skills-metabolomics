---
name: probabilistic-graph-integration-of-retention-order-and-msms-scores
description: Use when you have per-peak MS/MS candidate scores and pairwise ROASMI
  P_ij retention-order probabilities and need to rank candidate structures for each
  detected peak, e.g. to distinguish isomers or annotate peaks lacking usable MS/MS
  spectra.
when_to_use_negative:
- When your data is on HILIC chromatography — ROASMI currently applies only to RPLC
  systems, so retention scores would be invalid for this integration.
- When candidate pairs fall outside the C, H, N, O, P, and S elemental restrictions
  of ROASMI — retention predictions are out of domain.
- When a candidate pair's chromatographic peaks are too close for consistent retention
  order across experiments (ROASMI marks them indistinguishable); including them in
  ranking is unreliable.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3361
tools:
- name: ROASMI
  role: Provides the pairwise retention-order probabilities P_ij (via the RankNet
    retention prediction module) that serve as edge weights in the graph integration
    framework
  repo: https://github.com/FangYuan717/ROASMI
- name: aalto-ics-kepaco/msms_rt_score_integration
  role: Implements the Eric Bach et al. probabilistic framework that integrates retention
    and MS/MS scores using random spanning-tree ensembles to infer maximum-marginal
    top-k rankings
  repo: https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- name: chemprop
  role: Underlying molecular embedding module (D-MPNN) used to learn compound structures
    for ROASMI's retention score predictions
  repo: https://github.com/chemprop/chemprop
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:40975a8caa8db29775f56ee951f07d01e58c757da462725c890eb52f60bf6519
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - ROASMI
  - aalto-ics-kepaco/msms_rt_score_integration
  - chemprop
  - msms_rt_score_integration
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# probabilistic-graph-integration-of-retention-order-and-msms-scores

## Summary

Combines ROASMI-predicted retention-order probabilities P_ij with per-peak MS/MS spectral scores by modeling score integration as inference on a fully connected graph over candidate structures, using random spanning-tree ensembles to infer maximum-marginal top-k rankings. This lets ROASMI's RankNet retention scores refine annotations for peaks lacking usable MS/MS spectra or with coexisting isomers.

## When to use

Use when you have per-peak MS/MS candidate scores and pairwise ROASMI P_ij retention-order probabilities and need to rank candidate structures for each detected peak, e.g. to distinguish isomers or annotate peaks lacking usable MS/MS spectra.

## When NOT to use

- When your data is on HILIC chromatography — ROASMI currently applies only to RPLC systems, so retention scores would be invalid for this integration.
- When candidate pairs fall outside the C, H, N, O, P, and S elemental restrictions of ROASMI — retention predictions are out of domain.
- When a candidate pair's chromatographic peaks are too close for consistent retention order across experiments (ROASMI marks them indistinguishable); including them in ranking is unreliable.

## Inputs

- data/identify_toy_set.csv (ready-to-identify dataset containing candidate structures with MS/MS spectral scores for each detected peak)
- ROASMI-predicted pairwise retention-order probabilities P_ij for each pair of candidate structures
- Scalar parameter D controlling retention-vs-MS/MS score weight balance (supplied via args.py IdentifyArgs)

## Outputs

- identified_annotations.csv (ranked annotations per peak: top-k candidate identity, integrated score, per-peak scores)
- Top-k candidate structure rankings for each detected peak

## How to apply

Load the ready-to-identify dataset (e.g. data/identify_toy_set.csv) holding MS/MS scores per candidate structure per peak plus ROASMI's pairwise P_ij retention-order probabilities. Construct a fully connected graph over candidate structures for each peak, assigning each node its MS/MS score and each directed edge its ROASMI P_ij probability. Run the Eric Bach et al. probabilistic integration framework (aalto-ics-kepaco/msms_rt_score_integration) using random spanning-tree ensembles to infer the maximum probability of ranking top-k candidates, sweeping the scalar parameter D supplied via args.py (IdentifyArgs) to control the retention-vs-spectra weight balance. Aggregate the spanning-tree results to obtain posterior ranking scores and select top-k annotations per peak, then exclude any candidate pairs that ROASMI marks as indistinguishable (e.g. 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid, which the model cannot separate). Write ranked annotations (top-k candidate identity, integrated score, per-peak scores) to identified_annotations.csv.

## Related tools

- **ROASMI** (Provides the pairwise retention-order probabilities P_ij (via the RankNet retention prediction module) that serve as edge weights in the graph integration framework) — https://github.com/FangYuan717/ROASMI
- **aalto-ics-kepaco/msms_rt_score_integration** (Implements the Eric Bach et al. probabilistic framework that integrates retention and MS/MS scores using random spanning-tree ensembles to infer maximum-marginal top-k rankings) — https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- **chemprop** (Underlying molecular embedding module (D-MPNN) used to learn compound structures for ROASMI's retention score predictions) — https://github.com/chemprop/chemprop

## Examples

```
python code/Identify.py --pred data/identify_toy_set.csv --D 0.5
```

## Evaluation signals

- Confirm the script (code/Identify.py) runs to completion without error
- Verify identified_annotations.csv contains ranked annotations for every peak in the input set
- Check that top-k candidate scores reflect the D-parameterized retention-vs-spectra balance, with the posterior score for each peak's top candidate being a maximum-marginal probability
- Confirm candidate pairs ROASMI marks as indistinguishable (e.g. very close retention with high variance) are excluded from the final ranked output
- Cross-check that the ranked top-1 annotation for each peak is consistent with the underlying graph edge probabilities (P_ij) and MS/MS node scores

## Limitations

- Candidate pairs with chromatographic peaks too close together may not have consistent retention order across experiments, making the pairwise P_ij unreliable for distinguishing them
- ROASMI only applies to RPLC systems; the integration cannot be extended to HILIC without retrained retention models
- The model is limited to conventional compounds within the elemental restrictions of C, H, N, O, P, and S
- Some molecular pairs (e.g. 4-hydroxybenzoic acid and 2,5-dihydroxybenzoic acid) are indistinguishable by ROASMI and must be excluded from ranked output

## Evidence

- [methods] Probabilistic graph integration of retention order and MS/MS scores: "we integrate predicted retention scores with MS/MS scores by running: python code/Identify.py"
- [readme] Spanning-tree based integration: [evidence span withheld — anchor status: not_found]
- [discussion] Indistinguishable pairs exclusion: "Such pairs should be excluded from the subsequent small molecule identification process."
- [other] Ranking integration framework source: "which is available in the [GitHub repository](https://github.com/aalto-ics-kepaco/msms_rt_score_integration)"
