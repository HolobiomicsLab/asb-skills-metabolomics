---
name: intermediate-fragment-persistence-weight-zeroing
description: Use when implementing or modifying the CFM-ID 4.0 fragmentation graph
  for ESI-MS/MS spectral prediction, specifically to remodel ring cleavages as sequential
  one-bond breaks while suppressing unstable intermediate fragment peaks.
when_to_use_negative:
- When working with legacy CFM-ID 2.0/3.0 code that already implements ring cleavage
  as a single simultaneous two-bond break with concatenated FV0+FV1 feature vectors
  — this skill replaces that design, so it is redundant there.
- When predicting EI-MS/MS spectra or handling MS spectra from instruments other than
  QTOF instruments — the article notes CFM-ID 4.0 is built around QTOF ESI-MS/MS data.
- When the fragment of interest is a stable ring-derived product (f1) rather than
  a transient single-bond-break intermediate — zeroing its persistence weight would
  incorrectly remove legitimate peaks.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CFM-ID 4.0
  role: The spectral prediction engine whose fragmentation-graph generation code is
    modified to add the sequential ring-cleavage model and zero the intermediate fragment's
    persistence probability
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: Docker
  role: Provides the containerized CFM-ID 4.0 execution environment for running the
    modified predictor
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:44afafedd4d57f43b8b8e6e92fc3f74824626c4734d2cc3394fa0f36b11bb974
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - Docker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# intermediate-fragment-persistence-weight-zeroing

## Summary

A CFM-ID 4.0 technique that models ring cleavage in ESI-MS/MS as two sequential single-bond transitions, assigning a persistence (self-transition) probability of zero to the chemically unstable intermediate fragment f_inter so it never contributes a predicted spectral peak.

## When to use

Use when implementing or modifying the CFM-ID 4.0 fragmentation graph for ESI-MS/MS spectral prediction, specifically to remodel ring cleavages as sequential one-bond breaks while suppressing unstable intermediate fragment peaks.

## When NOT to use

- When working with legacy CFM-ID 2.0/3.0 code that already implements ring cleavage as a single simultaneous two-bond break with concatenated FV0+FV1 feature vectors — this skill replaces that design, so it is redundant there.
- When predicting EI-MS/MS spectra or handling MS spectra from instruments other than QTOF instruments — the article notes CFM-ID 4.0 is built around QTOF ESI-MS/MS data.
- When the fragment of interest is a stable ring-derived product (f1) rather than a transient single-bond-break intermediate — zeroing its persistence weight would incorrectly remove legitimate peaks.

## Inputs

- Molecular structure (for CFM-ID, a chemical structure from which an adjacency-matrix-based representation of bond topology is derived)
- CFM-ID 4.0 source code from https://bitbucket.org/wishartlab/cfm-id-code/ or the Docker image https://hub.docker.com/r/wishartlab/cfmid
- Toy ring molecules for validation of the modified fragmentation graph

## Outputs

- Predicted ESI-MS/MS spectrum in which intermediate ring-cleavage fragments produce no peaks
- Fragmentation graph with sequential ring-cleavage transitions and zero-weight self-transitions on f_inter

## How to apply

Locate the fragment-graph generation code in the CFM-ID 4.0 source (bitbucket.org/wishartlab/cfm-id-code) and replace the legacy simultaneous two-bond ring cleavage, which concatenated two bond-break feature vectors (FV0+FV1), with a two-step sequential transition: first break one bond on the ring to create intermediate fragment f_inter, then break a second bond on that same ring to yield the final fragment f1. Because the single-bond-break intermediate is chemically unstable, set its persistence (self-transition) probability to zero so it is never emitted as a peak in the predicted MS spectrum. The rationale is that this lets bond-cleavage parameters learned from ordinary single-bond cleavages be shared with ring cleavages, removing the need for concatenated feature vectors. After modification, run the predictor on small toy ring molecules to confirm the intermediate does not appear as a peak while correctly cleaved ring products do.

## Related tools

- **CFM-ID 4.0** (The spectral prediction engine whose fragmentation-graph generation code is modified to add the sequential ring-cleavage model and zero the intermediate fragment's persistence probability) — https://bitbucket.org/wishartlab/cfm-id-code/
- **Docker** (Provides the containerized CFM-ID 4.0 execution environment for running the modified predictor) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- For each toy ring structure, no predicted peak maps to an intermediate fragment f_inter
- The persistence (self-transition) probability of f_inter is exactly zero in the modified fragmentation graph
- All legitimate two-bond ring-cleavage products (f1) are present as predicted fragments/peaks
- The model now uses single-bond-cleavage parameters shared with ordinary bond cleavages instead of concatenated FV0+FV1 feature vectors for ring cleavages

## Limitations

- Deeper fragmentation graphs introduce extra computational complexity, and the authors note there is still room for improvement in both accuracy and runtime.
- CFM-ID 4.0 is trained and evaluated on QTOF ESI-MS/MS spectra; the authors state they plan to adapt it to handle spectra from instruments other than QTOF, so results on non-QTOF data are not supported.
- The zeroed persistence weight is specific to the chemically unstable single-bond ring-cleavage intermediate; it must not be applied to stable final fragments.

## Evidence
