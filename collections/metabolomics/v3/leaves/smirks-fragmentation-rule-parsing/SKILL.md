---
name: smirks-fragmentation-rule-parsing
description: Use when building or auditing an ESI-MS/MS in-silico rule-based predictor
  that needs SMIRKS fragmentation rules with chemical-class routing and relative intensities,
  e.g. reproducing CFM-ID 4.0's extended MSRB rule set.
when_to_use_negative:
- Predicting spectra for compounds outside the covered chemical classes — CFM-ID's
  MSRB rules target specific ClassyFire classes (acylcarnitines, acylcholines, flavonoids,
  lipids); generic bond-topology (C2MS) prediction should be used instead.
- The input spectra were collected on non-QTOF instruments — the rules encode average
  scaled intensities from QTOF data at 10, 20, and 40 eV.
- You need machine-learned fragmentation parameters rather than hand-written rules
  — CFM-ID 4.0's main accuracy gains come from topology-learned parameters, not the
  MSRB library.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CFM-ID 4.0
  role: Source of the extended hand-written rule-based (MSRB) predictor whose SMIRKS
    rules are parsed and verified
  repo: https://bitbucket.org/wishartlab/msrb-fragmenter
- name: ClassyFire
  role: Routes each SMIRKS rule to the specific chemical classes suitable for MSRB
    fragmentation
- name: Docker
  role: Runtime environment for executing CFM-ID 4.0 while validating parsed rules
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:2ccbb9b2f8b774df0adede767f6b7b22795b8cc5d59188f6ec49331a2a8873f7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - ClassyFire
  - Docker
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# smirks-fragmentation-rule-parsing

## Summary

Extract, annotate, and verify CFM-ID 4.0's hand-made SMIRKS-based fragmentation (MSRB) rule library — 313 rules covering 11 chemical classes and 3 adduct types — attaching average scaled relative intensities and ClassyFire class routing for each rule.

## When to use

Use when building or auditing an ESI-MS/MS in-silico rule-based predictor that needs SMIRKS fragmentation rules with chemical-class routing and relative intensities, e.g. reproducing CFM-ID 4.0's extended MSRB rule set.

## When NOT to use

- Predicting spectra for compounds outside the covered chemical classes — CFM-ID's MSRB rules target specific ClassyFire classes (acylcarnitines, acylcholines, flavonoids, lipids); generic bond-topology (C2MS) prediction should be used instead.
- The input spectra were collected on non-QTOF instruments — the rules encode average scaled intensities from QTOF data at 10, 20, and 40 eV.
- You need machine-learned fragmentation parameters rather than hand-written rules — CFM-ID 4.0's main accuracy gains come from topology-learned parameters, not the MSRB library.

## Inputs

- SMIRKS rule definitions from the msrb-fragmenter Bitbucket repository
- Supplemental Table S1 listing SMIRKS-based fragmentation rules
- Experimental QTOF ESI-MS/MS spectra at 10, 20, and 40 eV collision energies (source of scaled relative intensities)

## Outputs

- msrb_rules_table.csv with columns: SMIRKS pattern, chemical class, adduct type, average scaled relative intensity
- Verified rule count and class/adduct coverage summary (313 rules; 11 chemical classes; 3 adduct types)

## How to apply

Retrieve the MSRB source from the msrb-fragmenter Bitbucket repository and Supplemental Table S1 of the CFM-ID 4.0 article listing the SMIRKS-based fragmentation rules. Parse each SMIRKS rule definition for the covered modular classes (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides, and lipids) and record each rule's fragmentation reaction pattern. Attach to each rule the average scaled relative intensity derived from experimental QTOF spectra at 10, 20, and 40 eV collision energies. Confirm the compiled rule count equals 313 rules and that ClassyFire routing matches the 11 chemical classes and 3 adduct types reported in the article, then write the verified table (SMIRKS pattern, chemical class, adduct type, relative intensity) to msrb_rules_table.csv.

## Related tools

- **CFM-ID 4.0** (Source of the extended hand-written rule-based (MSRB) predictor whose SMIRKS rules are parsed and verified) — https://bitbucket.org/wishartlab/msrb-fragmenter
- **ClassyFire** (Routes each SMIRKS rule to the specific chemical classes suitable for MSRB fragmentation)
- **Docker** (Runtime environment for executing CFM-ID 4.0 while validating parsed rules) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
python parse_smirks_rules.py --source SupplementalTableS1 --repo https://bitbucket.org/wishartlab/msrb-fragmenter --output msrb_rules_table.csv
```

## Evaluation signals

- Compiled rule count equals exactly 313 hand-made MSRB rules.
- Rule set covers exactly 11 chemical classes and 3 adduct types as stated in the article.
- ClassyFire class routing reproduces the 11 specific chemical classes reported for MSRB fragmentation.
- Every parsed rule carries an average scaled relative intensity traceable to QTOF spectra at 10, 20, and 40 eV collision energies.
- msrb_rules_table.csv schema check: each row has non-empty SMIRKS pattern, chemical class, adduct type, and relative intensity.

## Limitations

- Rules cover only specific chemical classes (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides, lipids); compounds outside these classes fall back to CFM-ID's learned topology-based predictor.
- Relative intensities are averages scaled from QTOF data, so they may not transfer to non-QTOF instruments (CFM-ID's discussion notes plans to adapt to other instruments).
- The article reports runtime and computational-complexity concerns for deeper fragmentation, which also applies when applying many rules exhaustively.

## Evidence

- [results] generation of 313 hand-made MSRB rules encoding fragmentation reactions and average scaled relative intensities (at 10, 20, and 40 eV): [evidence span withheld — anchor status: not_found]
