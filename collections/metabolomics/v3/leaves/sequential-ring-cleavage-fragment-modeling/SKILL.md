---
name: sequential-ring-cleavage-fragment-modeling
description: Use when predicting ESI-MS/MS spectra of small molecules containing rings
  with CFM-ID, or when ablation-attributing spectral-prediction accuracy gains to
  topology, ring-cleavage, and rule-based components on QTOF Metlin data.
when_to_use_negative:
- Spectra collected on instruments other than QTOF — the article notes CFM-ID 4.0
  is trained/evaluated on QTOF ESI-MS/MS and handling other instruments is future
  work
- EI-MS/MS prediction tasks where the sequential ring-cleavage ESI remodeling and
  Metlin QTOF training sets are not the relevant framework
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: CFM-ID 4.0
  role: Implements the two-step sequential ring-cleavage fragmentation graph model
    and trains its parameters on Metlin QTOF ESI-MS/MS data
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: ClassyFire
  role: Identifies the chemical classes routed to MSRB (rule-based) fragmentation
    versus the learned sequential ring-cleavage model
- name: msrb-fragmenter
  role: Standalone source for the extended rule-based fragmenter compared against
    the learned ring-cleavage component in the ablation
  repo: https://bitbucket.org/wishartlab/msrb-fragmenter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:c4ac7e1f444525280933289b22234a4416d3841909b67f7662e0cb75b31ca45e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 2.0
  - CFM-ID 3.0
  - CFM-ID 4.0
  - ClassyFire
  - Metlin
  - msrb-fragmenter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# sequential-ring-cleavage-fragment-modeling

## Summary

Models ESI-MS/MS ring cleavage in CFM-ID 4.0 as a two-step sequential process of simple chemical bond dissociations (rather than a single multigraph break), with a maximum of 2 ring breaks per fragment. Ablation attributes ~35% of CFM-ID 4.0's ESI-MS/MS prediction gain over prior versions to this component, second to molecular topology features (~65%).

## When to use

Use when predicting ESI-MS/MS spectra of small molecules containing rings with CFM-ID, or when ablation-attributing spectral-prediction accuracy gains to topology, ring-cleavage, and rule-based components on QTOF Metlin data.

## When NOT to use

- Spectra collected on instruments other than QTOF — the article notes CFM-ID 4.0 is trained/evaluated on QTOF ESI-MS/MS and handling other instruments is future work
- EI-MS/MS prediction tasks where the sequential ring-cleavage ESI remodeling and Metlin QTOF training sets are not the relevant framework

## Inputs

- Metlin QTOF ESI-MS/MS training/testing spectra ([M+H]+ and [M-H]-, collision energies 10/20/40 eV)
- Experimental peak lists for Dice/Dot Product evaluation
- Candidate molecular structures (from which adjacency/topology features and fragments are derived)

## Outputs

- Predicted ESI-MS/MS spectra from the fragmentation graph of transitions
- Component-contribution (ablation) shares: topology ~65%, ring cleavage ~35%, rule-based ~1%
- Ring-break settings (fragmentation graph depth 2, max ring breaks 2) configuration for CFM-ID 4.0

## How to apply

Within CFM-ID 4.0, generate all theoretically possible fragments combinatorially into a fragmentation graph with fragment transitions and self-transitions, and model ring cleavage as a sequence of simple bond dissociations with max ring breaks set to 2 and fragmentation graph depth 2, instead of the older one-step ring-cleavage treatment of CFM-ID 2.0/3.0. Train the variant on the Metlin 2015/2019 QTOF ESI-MS/MS sets ([M+H]+ and [M-H]- at 10/20/40 eV) under 10-fold cross-validation, using a fully connected NN with two 128-node hidden layers. Isolate this component's contribution by training a baseline without connectivity-matrix topology features, then adding topology features, then adding sequential ring cleavage, then the MSRB rules. Compute Dice coefficient, Precision, Recall, and re-weighted Stein Dot Product between predicted and experimental spectra at a peak match tolerance of m/z difference < 0.01 Da or 10 ppm (whichever first). For rule-based classes (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides), ClassyFire is used to identify classes suitable for MSRB fragmentation.

## Related tools

- **CFM-ID 4.0** (Implements the two-step sequential ring-cleavage fragmentation graph model and trains its parameters on Metlin QTOF ESI-MS/MS data) — https://bitbucket.org/wishartlab/cfm-id-code/
- **ClassyFire** (Identifies the chemical classes routed to MSRB (rule-based) fragmentation versus the learned sequential ring-cleavage model)
- **msrb-fragmenter** (Standalone source for the extended rule-based fragmenter compared against the learned ring-cleavage component in the ablation) — https://bitbucket.org/wishartlab/msrb-fragmenter

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- Ablation delta: adding sequential ring cleavage on top of topology features should shift Dice/Dot Product metrics by an amount attributing ~35% of the total CFM-ID 4.0 gain (topology ~65%, rule-based ~1%)
- Peak matching against experimental spectra must use tolerance m/z < 0.01 Da or 10 ppm, whichever comes first
- Ring-break parameters must be fragmentation graph depth 2 and max ring breaks 2; a network with two 128-node hidden layers is used
- MSRB module should predict fewer than 3% of all MS/MS spectra, confirming learned (not rule-based) modeling dominates ring fragmentation
- Reproduced component shares should be within a few percentage points of the reported values

## Limitations

- Rule-based (MSRB) features contribute only ~1% of improvement, so this component should not be credited with large gains
- Deeper fragmentation graphs introduce extra computational complexity (runtime not benchmarked exactly)
- Model is trained/evaluated on QTOF ESI-MS/MS; adaptation to other instrument types is acknowledged as future work

## Evidence
