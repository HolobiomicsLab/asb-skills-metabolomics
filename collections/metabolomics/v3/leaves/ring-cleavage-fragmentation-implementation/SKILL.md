---
name: ring-cleavage-fragmentation-implementation
description: Use when modifying or extending CFM-ID-style fragmentation graphs to
  model ring cleavage, or when predicted MS/MS spectra must exclude chemically unstable
  ring-cleavage intermediate fragments.
when_to_use_negative:
- The fragmentation of interest involves only acyclic single-bond cleavages, where
  the legacy one-bond transition model already suffices
- You need rule-based prediction for covered chemical classes (acylcarnitines, acylcholines,
  flavonols, flavones, flavanones, flavonoid glycosides) handled by CFM-ID 4.0's enhanced
  hand-written rule predictor rather than the learned ring-cleavage model
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: CFM-ID 4.0
  role: Competitive fragmentation modeling framework whose fragment-graph generation
    code is modified to implement the two-step sequential ring-cleavage transition
    with zero persistence probability for the intermediate
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: Docker
  role: Execution environment for running the modified CFM-ID 4.0 predictor (wishartlab/cfmid
    image)
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:fcf6950e5c26281632361f63caa7ab916a4b72fd991cc4563320a895cc347982
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - Docker
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ring-cleavage-fragmentation-implementation

## Summary

Implements CFM-ID 4.0's remodeling of ESI-MS/MS ring cleavage as two sequential one-bond transitions, with the unstable intermediate fragment's persistence (self-transition) probability fixed to zero so it never emits a predicted spectral peak.

## When to use

Use when modifying or extending CFM-ID-style fragmentation graphs to model ring cleavage, or when predicted MS/MS spectra must exclude chemically unstable ring-cleavage intermediate fragments.

## When NOT to use

- The fragmentation of interest involves only acyclic single-bond cleavages, where the legacy one-bond transition model already suffices
- You need rule-based prediction for covered chemical classes (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides) handled by CFM-ID 4.0's enhanced hand-written rule predictor rather than the learned ring-cleavage model

## Inputs

- Small molecule chemical structure (molecular topology / adjacency-matrix-based representation used by CFM-ID 4.0)
- CFM-ID 4.0 source code or wishartlab/cfmid Docker image
- Toy ring molecule structures for validation

## Outputs

- Modified fragment-graph generation with sequential ring-cleavage transitions (ring bond break 1 → f_inter, ring bond break 2 → f1)
- Predicted ESI-MS/MS spectrum in which the intermediate fragment f_inter produces no peak (persistence weight = 0)

## How to apply

Set up the CFM-ID 4.0 environment from the source at bitbucket.org/wishartlab/cfm-id-code or the Docker image hub.docker.com/r/wishartlab/cfmid. In the fragment-graph generation code, replace the legacy simultaneous two-bond ring cleavage (which concatenated two bond-break feature vectors FV0 and FV1) with a sequential model: first break a single bond on the ring to create intermediate fragment f_inter, then break a second bond on the same ring to yield the final fragment f1. Set the persistence (self-transition) probability of f_inter to zero so the unstable intermediate cannot produce a peak in the predicted spectrum. This lets bond-cleavage parameters learned from single-bond cleavages be shared with ring cleavages instead of requiring concatenated feature vectors. Validate on small toy ring molecules: confirm no predicted peak maps to an intermediate fragment while all legitimate two-bond ring-cleavage products still appear as predicted fragments.

## Related tools

- **CFM-ID 4.0** (Competitive fragmentation modeling framework whose fragment-graph generation code is modified to implement the two-step sequential ring-cleavage transition with zero persistence probability for the intermediate) — https://bitbucket.org/wishartlab/cfm-id-code/
- **Docker** (Execution environment for running the modified CFM-ID 4.0 predictor (wishartlab/cfmid image)) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
docker run -v $(pwd):/data wishartlab/cfmid cfm-predict /data/toy_ring.smi 0.001 /data/param_output.log /data/param_wlog.txt
```

## Evaluation signals

- For each toy ring structure, no predicted peak maps to an intermediate fragment (f_inter persistence weight = 0 and no intermediate-derived peak in the spectrum)
- All legitimate two-bond ring-cleavage products are present as predicted fragments for the toy ring molecules
- Predicted ring-cleavage peaks derive from fragments produced by two sequential one-bond transitions sharing single-bond-cleavage learned parameters, not from concatenated FV0+FV1 feature vectors
- Predicted ESI-MS/MS spectra on ring-containing compounds should match experimental Metlin QTOF data at least as well as the legacy simultaneous two-bond model

## Limitations

- Deeper fragmentation graphs introduce extra computational complexity
- Runtime limitations are acknowledged without exact benchmarks; there is room for further improvement in accuracy and runtime
- CFM-ID 4.0 is trained and evaluated on QTOF ESI-MS/MS spectra and does not yet handle spectra from other instruments

## Evidence

- [results] Finding from task card: intermediate persistence probability zero.: [evidence span withheld — anchor status: not_found]
