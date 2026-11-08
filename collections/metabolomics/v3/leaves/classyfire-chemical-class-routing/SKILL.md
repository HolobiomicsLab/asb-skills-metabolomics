---
name: classyfire-chemical-class-routing
description: Use when predicting ESI-MS/MS spectra for a compound that may belong
  to a rule-covered class (e.g. acylcarnitines, acylcholines, flavonols, flavones,
  flavanones, flavonoid glycosides, lipids) and you must decide whether MSRB rule-based
  prediction applies.
when_to_use_negative:
- The compound's ClassyFire class is not among the 11 classes covered by the extended
  MSRB rule set (e.g. outside acylcarnitines, acylcholines, flavonoids, lipids) —
  use CFM-ID 4.0's combinatorial/topology-based predictor instead.
- The goal is EI-MS/MS prediction or general compound identification where the improved
  ESI-MS/MS MSRB routing and its 3 adduct types are not applicable.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: CFM-ID 4.0
  role: Hosts the MSRB predictor; uses ClassyFire output to route compounds to the
    correct chemical-class-specific SMIRKS fragmentation rules
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: ClassyFire
  role: Classifies the query compound so CFM-ID can determine which chemical classes
    are suitable for MSRB fragmentation
- name: msrb-fragmenter
  role: Standalone source for the SMIRKS-based rule fragmentation module whose rules
    are routed by ClassyFire class labels
  repo: https://bitbucket.org/wishartlab/msrb-fragmenter
- name: Docker
  role: Distribution mechanism for running CFM-ID 4.0 (including the MSRB routing)
    locally
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:c0443e9f032e0e0de32ee06a2f76f7573406ee251725b0ec517e161ea086dea8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - ClassyFire
  - Docker
  - msrb-fragmenter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# classyfire-chemical-class-routing

## Summary

Uses ClassyFire to identify the specific chemical classes of an input compound and route it to the appropriate hand-written SMIRKS-based (MSRB) fragmentation rules in CFM-ID 4.0, which cover 11 chemical classes and 3 adduct types with 313 hand-made rules encoding fragmentation reactions and average scaled relative intensities.

## When to use

Use when predicting ESI-MS/MS spectra for a compound that may belong to a rule-covered class (e.g. acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides, lipids) and you must decide whether MSRB rule-based prediction applies.

## When NOT to use

- The compound's ClassyFire class is not among the 11 classes covered by the extended MSRB rule set (e.g. outside acylcarnitines, acylcholines, flavonoids, lipids) — use CFM-ID 4.0's combinatorial/topology-based predictor instead.
- The goal is EI-MS/MS prediction or general compound identification where the improved ESI-MS/MS MSRB routing and its 3 adduct types are not applicable.

## Inputs

- Input compound structure (e.g. SMILES/InChI) submitted to ClassyFire for chemical class assignment
- ClassyFire class label for the query compound
- Adduct type of the precursor ion (one of the 3 adduct types covered by the rule set)
- SMIRKS-based MSRB rule library (Supplemental Table S1 / msrb-fragmenter rules) with per-rule relative intensities

## Outputs

- Routing decision: whether the compound is eligible for MSRB rule-based fragmentation
- List of matched SMIRKS fragmentation rules applicable to the compound's chemical class and adduct type
- Rule table (e.g. msrb_rules_table.csv) with columns for SMIRKS pattern, chemical class, adduct type, and relative intensity

## How to apply

First run ClassyFire on the input structure to obtain its chemical class label, since CFM-ID 4.0 uses ClassyFire to identify the specific chemical classes suitable for MSRB fragmentation. Check whether the assigned class matches one of the 11 classes covered by the extended SMIRKS rule set (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides, and lipids) and whether the adduct type is one of the 3 supported adduct types. If the class matches, route the compound to the MSRB fragmenter and apply the SMIRKS rules from Supplemental Table S1, each of which encodes a fragmentation reaction pattern together with an average scaled relative intensity derived from experimental QTOF spectra at 10, 20, and 40 eV collision energies. If the class is not covered, fall back to CFM-ID 4.0's combinatorial/topology-based (C2MS) predictor instead of forcing MSRB rules. Verify the routing by confirming the rule count and class/adduct coverage against the published total of 313 rules.

## Related tools

- **CFM-ID 4.0** (Hosts the MSRB predictor; uses ClassyFire output to route compounds to the correct chemical-class-specific SMIRKS fragmentation rules) — https://bitbucket.org/wishartlab/cfm-id-code/
- **ClassyFire** (Classifies the query compound so CFM-ID can determine which chemical classes are suitable for MSRB fragmentation)
- **msrb-fragmenter** (Standalone source for the SMIRKS-based rule fragmentation module whose rules are routed by ClassyFire class labels) — https://bitbucket.org/wishartlab/msrb-fragmenter
- **Docker** (Distribution mechanism for running CFM-ID 4.0 (including the MSRB routing) locally) — https://hub.docker.com/r/wishartlab/cfmid

## Evaluation signals

- The total number of compiled SMIRKS rules equals 313, covering 11 chemical classes and 3 adduct types as reported in the article.
- Class routing matches: each rule in the table is assigned one of the 11 ClassyFire chemical classes reported for the MSRB predictor.
- Every routed rule carries an average scaled relative intensity derived from experimental QTOF spectra at 10, 20, and 40 eV collision energies.
- Compounds whose ClassyFire class is outside the covered set are NOT routed to MSRB rules (fallback to the topology-based predictor is triggered).
- The rule table schema contains the required columns: SMIRKS pattern, chemical class, adduct type, and relative intensity.

## Limitations

- The extended MSRB rule set covers only 11 chemical classes and 3 adduct types; compounds outside these classes get no rule-based ESI-MS/MS prediction.
- CFM-ID 4.0's MSRB predictor was developed and evaluated for QTOF ESI-MS/MS data; the authors note plans to adapt CFM-ID to handle spectra from instruments other than QTOF instruments.
- Deeper fragmentation graphs introduce extra computational complexity, and there is acknowledged room for further improvement in accuracy and runtime.

## Evidence

- [supplementary] 313 MSRB rules with scaled relative intensities: [evidence span withheld — anchor status: not_found]
