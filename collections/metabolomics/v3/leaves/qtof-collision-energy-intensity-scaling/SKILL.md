---
name: qtof-collision-energy-intensity-scaling
description: Use when building or validating a hand-written SMIRKS rule library (MSRB)
  whose relative-intensity parameters must be sourced from QTOF spectra at 10/20/40
  eV, e.g. compiling msrb_rules_table.csv for CFM-ID 4.0-class predictors.
when_to_use_negative:
- Input spectra were collected on instruments other than QTOF — CFM-ID 4.0 explicitly
  notes the intent to later adapt to non-QTOF instruments, so scaled intensities from
  this library may not transfer
- The prediction task is purely combinatorial/learned (C2MS bond-topology model) rather
  than rule-based MSRB, since ClassyFire routing and SMIRKS rules only apply to classes
  suitable for MSRB fragmentation
tools:
- name: CFM-ID 4.0
  role: Hosts the enhanced hand-written rule-based (MSRB) predictor whose SMIRKS rules
    carry the QTOF-scaled intensities
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: ClassyFire
  role: Identifies the specific chemical classes suitable for MSRB fragmentation,
    routing each rule to its class label
- name: Docker
  role: Deployment vehicle for running the CFM-ID 4.0 MSRB predictor (wishartlab/cfmid
    image)
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:7e321a09fb21ce4ee4830a167d31121cc7c3886817674f2816715f14ccff87da
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - ClassyFire
  - Docker
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# qtof-collision-energy-intensity-scaling

## Summary

Curates and intensity-scales hand-written SMIRKS-based (MSRB) fragmentation rules so that each rule carries average scaled relative intensities derived from experimental QTOF ESI-MS/MS spectra acquired at 10, 20, and 40 eV collision energies, as implemented in CFM-ID 4.0's expanded rule library of 313 rules covering 11 chemical classes and 3 adduct types.

## When to use

Use when building or validating a hand-written SMIRKS rule library (MSRB) whose relative-intensity parameters must be sourced from QTOF spectra at 10/20/40 eV, e.g. compiling msrb_rules_table.csv for CFM-ID 4.0-class predictors.

## When NOT to use

- Input spectra were collected on instruments other than QTOF — CFM-ID 4.0 explicitly notes the intent to later adapt to non-QTOF instruments, so scaled intensities from this library may not transfer
- The prediction task is purely combinatorial/learned (C2MS bond-topology model) rather than rule-based MSRB, since ClassyFire routing and SMIRKS rules only apply to classes suitable for MSRB fragmentation

## Inputs

- SMIRKS-based fragmentation rule definitions (Supplemental Table S1 of the CFM-ID 4.0 paper / msrb-fragmenter repository)
- Experimental QTOF ESI-MS/MS spectra (e.g. Metlin-sourced) acquired at 10, 20, and 40 eV collision energies
- ClassyFire chemical class labels for each covered compound class

## Outputs

- msrb_rules_table.csv with columns for SMIRKS pattern, chemical class, adduct type, and average scaled relative intensity (10/20/40 eV)
- Verified rule inventory: 313 hand-made MSRB rules covering 11 chemical classes and 3 adduct types

## How to apply

Retrieve the MSRB source from the msrb-fragmenter Bitbucket repository and the article's Supplemental Table S1 listing the SMIRKS-based fragmentation rules. Parse each SMIRKS pattern for the covered modular classes (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides, and lipids) and record its fragmentation reaction pattern. Attach to each rule the average scaled relative intensity incorporated from experimental QTOF spectra at 10, 20, and 40 eV collision energies, since CFM-ID 4.0 was trained and tested on a much larger set of QTOF ESI-MS/MS spectra. Route each rule using the ClassyFire class label and adduct type, then count and verify the compiled set equals 313 hand-made MSRB rules spanning 11 chemical classes and 3 adduct types. Write the verified table with columns for SMIRKS pattern, chemical class, adduct type, and relative intensity to msrb_rules_table.csv.

## Related tools

- **CFM-ID 4.0** (Hosts the enhanced hand-written rule-based (MSRB) predictor whose SMIRKS rules carry the QTOF-scaled intensities) — https://bitbucket.org/wishartlab/cfm-id-code/
- **ClassyFire** (Identifies the specific chemical classes suitable for MSRB fragmentation, routing each rule to its class label)
- **Docker** (Deployment vehicle for running the CFM-ID 4.0 MSRB predictor (wishartlab/cfmid image)) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
df.to_csv('msrb_rules_table.csv', columns=['smirks','chemical_class','adduct_type','relative_intensity'], index=False)
```

## Evaluation signals

- Compiled rule count equals exactly 313 hand-made MSRB rules
- Class routing matches the 11 ClassyFire chemical classes and 3 adduct types reported in the article
- Every rule row in msrb_rules_table.csv carries an average scaled relative intensity traceable to QTOF spectra at 10, 20, and 40 eV
- CSV schema contains exactly the columns: SMIRKS pattern, chemical class, adduct type, relative intensity
- Each rule's fragmentation reaction pattern parses as valid SMIRKS and maps to one of the covered classes (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides, lipids)

## Limitations

- Intensity scaling is grounded in QTOF ESI-MS/MS data only; adaptation to other instrument types was still planned, not implemented
- Coverage is limited to the classes ClassyFire identifies as suitable for MSRB fragmentation — compounds outside the 11 covered classes fall back to the combinatorial/learned predictor
- Deeper fragmentation graphs add computational complexity, and the authors acknowledge room for further improvement in accuracy and runtime

## Evidence
