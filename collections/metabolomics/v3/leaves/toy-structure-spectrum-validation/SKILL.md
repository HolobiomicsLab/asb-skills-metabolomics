---
name: toy-structure-spectrum-validation
description: Use when verifying that a modified or newly configured CFM-ID 4.0 fragmentation
  graph correctly suppresses peaks from unstable ring-cleavage intermediates before
  trusting predicted ESI-MS/MS spectra of ring-containing compounds.
when_to_use_negative:
- The molecule has no rings — single-bond (acyclic) cleavage validation does not exercise
  the sequential ring-cleavage intermediate suppression path
- You need spectra from instruments other than QTOF — the article notes CFM-ID must
  still 'adapt CFM-ID to handle MS spectra collected using instruments other than
  QTOF instruments'
- You are validating learned bond-cleavage parameter accuracy against experimental
  spectra rather than checking the structural invariant that intermediate fragments
  produce no peak
tools:
- name: CFM-ID 4.0
  role: Predictor whose modified fragmentation graph is exercised; generates theoretically
    possible fragments combinatorially as a fragmentation graph with transition and
    self-transition probabilities
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: Docker
  role: Execution environment used to run the CFM-ID 4.0 predictor without rebuilding
    from source
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:876f21e7b747a804a2a722af960e6e58daf19235bba56f0c40d9857aa489ac3d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - Docker
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# toy-structure-spectrum-validation

## Summary

A validation recipe for CFM-ID 4.0's sequential ring-cleavage model: run the predictor on small toy ring molecules and confirm that the chemically unstable intermediate fragment (whose persistence/self-transition probability is fixed to zero) produces no predicted MS peak, while legitimate two-bond ring-cleavage products do appear.

## When to use

Use when verifying that a modified or newly configured CFM-ID 4.0 fragmentation graph correctly suppresses peaks from unstable ring-cleavage intermediates before trusting predicted ESI-MS/MS spectra of ring-containing compounds.

## When NOT to use

- The molecule has no rings — single-bond (acyclic) cleavage validation does not exercise the sequential ring-cleavage intermediate suppression path
- You need spectra from instruments other than QTOF — the article notes CFM-ID must still 'adapt CFM-ID to handle MS spectra collected using instruments other than QTOF instruments'
- You are validating learned bond-cleavage parameter accuracy against experimental spectra rather than checking the structural invariant that intermediate fragments produce no peak

## Inputs

- Small toy ring-containing molecule structures (e.g., SMILES or MOL files) with a single ring for controlled testing
- CFM-ID 4.0 fragmentation graph containing fragment transitions and self-transitions for each toy structure
- Predicted ESI-MS/MS peak lists (predicted spectra) from the modified CFM-ID 4.0 predictor

## Outputs

- Per-structure pass/fail validation report mapping each predicted peak to its originating fragment
- Confirmation that the intermediate fragment f_inter has persistence weight = 0 and contributes no spectral peak
- List of correctly predicted two-bond ring-cleavage fragment peaks for each toy ring molecule

## How to apply

Set up the CFM-ID 4.0 environment from bitbucket.org/wishartlab/cfm-id-code or the wishartlab/cfmid Docker image. Select small toy molecules containing a single ring and run the predictor (QTOF ESI-MS/MS setting) to generate the fragmentation graph with fragment transitions and self-transitions. Inspect the graph: the sequential two-step ring cleavage should first break one ring bond to create the intermediate fragment f_inter, whose persistence (self-transition) probability must be set to zero, then break a second bond on the same ring to yield the final fragment f1. Compare each predicted spectrum's peaks against the fragment assignments; the validation passes only if no peak maps to an intermediate fragment while all legitimate two-bond ring-cleavage products are present. Rationale: CFM-ID 4.0 models ring cleavage as two sequential one-bond transitions so that parameters learned from single-bond cleavages can be shared, and the zero-persistence convention prevents the chemically unstable intermediate from emitting a spectral peak.

## Related tools

- **CFM-ID 4.0** (Predictor whose modified fragmentation graph is exercised; generates theoretically possible fragments combinatorially as a fragmentation graph with transition and self-transition probabilities) — https://bitbucket.org/wishartlab/cfm-id-code/
- **Docker** (Execution environment used to run the CFM-ID 4.0 predictor without rebuilding from source) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
docker pull wishartlab/cfmid  # then run the CFM-ID 4.0 predictor on toy ring SMILES and check that no peak maps to the zero-persistence intermediate fragment
```

## Evaluation signals

- For every toy ring structure, no predicted peak in the output spectrum maps to an intermediate fragment (intermediate persistence weight = 0 and no intermediate-derived peak present)
- All legitimate two-bond ring-cleavage products are present as predicted fragments/peaks in the output spectrum
- The fragmentation graph shows two sequential one-bond ring transitions (first break creates f_inter, second break yields f1) instead of a single concatenated FV0+FV1 two-bond event
- The same predictor produces correct single-bond cleavage peaks for acyclic controls, confirming shared single-bond parameters remain intact
- Toy-molecule validation is repeated across several ring chemistries before scaling to real compounds

## Limitations

- Deeper fragmentation graphs (more sequential transition steps) 'introduce extra computational complexity'
- CFM-ID 4.0 currently targets QTOF ESI-MS/MS data; other instrument types are not handled
- The validation only checks the structural invariant on toy molecules — it does not quantify accuracy against experimental spectra, and the authors note 'there is still room for further improvement, both in terms of accuracy and runtime'

## Evidence

- [full_text] Set the persistence (self-transition) probability of the intermediate fragment f_inter to zero so the unstable intermediate never contributes a peak: [evidence span withheld — anchor status: not_found]
