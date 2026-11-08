---
name: fragmentation-graph-transition-modelling
description: Use when predicting ESI-MS/MS spectra from a compound structure in CFM-ID
  4.0, or when modifying ring-cleavage transition probabilities so unstable intermediates
  do not emit peaks.
when_to_use_negative:
- When experimental reference spectra already exist for the compound — matching against
  the reference spectral library (RSL) is more reliable than in silico prediction
- When the input spectrum was collected on an instrument other than a QTOF — CFM-ID
  4.0 was trained and evaluated on QTOF ESI-MS/MS data and does not yet handle other
  instrument types
- When the goal is EI-MS/MS prediction using hand-coded fragmentation rules only —
  the learned fragmentation-graph transition model targets ESI-MS/MS, while EI predictions
  still rely on the rule-based predictor
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: CFM-ID 4.0
  role: Implements the fragmentation-graph enumeration and learned transition-probability
    estimation for ESI-MS/MS spectral prediction; also provides the rule-based (MSRB)
    predictor layer
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: ClassyFire
  role: Identifies the specific chemical classes of the input compound that are suitable
    for MSRB (rule-based) fragmentation, gating whether hand-coded rules augment the
    learned graph model
- name: Docker
  role: Distribution vehicle for the CFM-ID 4.0 runtime (wishartlab/cfmid image),
    enabling reproducible execution of the fragmentation-graph prediction pipeline
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:61f468816942604c3780ca3113a69e53791e7ea934800e8deef967e1e91fb5aa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - ClassyFire
  - Docker
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# fragmentation-graph-transition-modelling

## Summary

CFM-ID 4.0 models ESI-MS/MS fragmentation as a fragmentation graph whose nodes are theoretically possible fragments and whose edges carry learned transition probabilities derived from molecular-topology features, so that predicted peak intensities emerge from probabilistic bond-dissociation transitions rather than hand-tuned heuristics.

## When to use

Use when predicting ESI-MS/MS spectra from a compound structure in CFM-ID 4.0, or when modifying ring-cleavage transition probabilities so unstable intermediates do not emit peaks.

## When NOT to use

- When experimental reference spectra already exist for the compound — matching against the reference spectral library (RSL) is more reliable than in silico prediction
- When the input spectrum was collected on an instrument other than a QTOF — CFM-ID 4.0 was trained and evaluated on QTOF ESI-MS/MS data and does not yet handle other instrument types
- When the goal is EI-MS/MS prediction using hand-coded fragmentation rules only — the learned fragmentation-graph transition model targets ESI-MS/MS, while EI predictions still rely on the rule-based predictor

## Inputs

- Chemical structure of the query compound (e.g. SMILES or MOL file parsed into an adjacency matrix)
- Instrument energy level settings (low/medium/high collision energy) for ESI-MS/MS prediction
- Optionally, a chemical class annotation (via ClassyFire) to trigger rule-based MSRB fragmentation where applicable

## Outputs

- Predicted ESI-MS/MS peak list (m/z and intensity) at each requested energy level
- Fragmentation graph annotated with fragments, transitions, self-transitions, and learned transition probabilities
- Set of terminal fragments including two-step ring-cleavage products, with intermediate f_inter peaks suppressed (persistence probability = 0)

## How to apply

First, enumerate all theoretically possible fragments of the input structure combinatorially and structure them as a fragmentation graph, where edges are fragment transitions and self-transitions encode persistence of a fragment. Then, for each transition, extract topological features from the adjacency-matrix representation of the parent and child fragments (bond types, bond energies, atom environments) and feed these to the learned CFM model to estimate the transition probability for each instrument energy level (e.g. low/medium/high collision energies in the QTOF-trained model). For ring cleavages specifically, implement the two-step sequential model: break one bond on the ring to create an intermediate fragment f_inter, then break a second bond on the same ring to yield the final fragment f1; set the persistence (self-transition) probability of f_inter to zero so the chemically unstable intermediate never contributes a peak to the predicted spectrum. Finally, construct the predicted spectrum by summing peak intensities over the transition probabilities across all energy levels, and validate that intermediate-derived peaks are absent while legitimate two-bond ring-cleavage products remain present. The rationale is that encoding bond-topology information lets parameters learned from single-bond cleavages be shared with ring cleavages, eliminating the need for concatenated feature vectors (FV0+FV1) used for simultaneous two-bond breaks in earlier CFM-ID versions.

## Related tools

- **CFM-ID 4.0** (Implements the fragmentation-graph enumeration and learned transition-probability estimation for ESI-MS/MS spectral prediction; also provides the rule-based (MSRB) predictor layer) — https://bitbucket.org/wishartlab/cfm-id-code/
- **ClassyFire** (Identifies the specific chemical classes of the input compound that are suitable for MSRB (rule-based) fragmentation, gating whether hand-coded rules augment the learned graph model)
- **Docker** (Distribution vehicle for the CFM-ID 4.0 runtime (wishartlab/cfmid image), enabling reproducible execution of the fragmentation-graph prediction pipeline) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
docker run -it wishartlab/cfmid cfm-id --predict  'CC(=O)OC(C(=O))ONC(=O)C' --out predicted_spectrum
```

## Evaluation signals

- No predicted peak should map to a ring-cleavage intermediate fragment f_inter (its persistence/self-transition probability is fixed to zero and no intermediate-derived peak appears in the output spectrum)
- All legitimate two-bond ring-cleavage products should appear as predicted fragments/peaks for toy ring structures
- Predicted spectra from the topology-learned model should outperform CFM-ID 2.0/3.0 baselines on the same test set (the paper reports significantly improved accuracy for both spectral prediction and compound identification)
- Transition probabilities must be estimated for every edge (transition) and self-transition in the fragmentation graph before spectrum construction — unannotated edges indicate an incomplete graph build
- On the CASMI 2016 category-3 benchmark, the modified model should retain or improve top-1 identification accuracy relative to the unmodified CFM-ID 4.0

## Limitations

- Setting intermediate persistence to zero is a hard-coded constraint; it prevents unstable intermediates from producing peaks but means any real-world case where an intermediate is actually stable enough to be observed would be missed
- Deeper fragmentation graphs introduce extra computational complexity, which the authors acknowledge as room for improvement in runtime
- The learned transition parameters were trained on QTOF ESI-MS/MS data (Metlin, plus HMDB/FooDB/ContaminantDB superclass distributions), so transfer to other instrument types (e.g. ion trap, Orbitrap) is not yet supported
- The adjacency-matrix representation captures topological bond features but may not encode full 3D conformational effects on bond lability

## Evidence
