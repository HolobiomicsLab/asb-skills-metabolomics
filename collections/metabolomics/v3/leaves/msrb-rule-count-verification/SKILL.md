---
name: msrb-rule-count-verification
description: Use when auditing or reproducing CFM-ID 4.0's SMIRKS-based fragmentation
  rule set — e.g. verifying that the MSRB library contains 313 rules spanning the
  reported chemical classes and adduct types before reuse.
when_to_use_negative:
- When the goal is to predict spectra with the MSRB predictor rather than verify its
  rule library — run CFM-ID's predictor instead of parsing rule counts
- When analyzing MS/MS spectra from non-QTOF instruments, which CFM-ID 4.0 does not
  yet handle
- When the fragments/classes of interest lie outside the covered classes (acylcarnitines,
  acylcholines, flavonols, flavones, flavanones, flavonoid glycosides, lipids) — the
  313-rule library does not cover other chemical classes
tools:
- name: CFM-ID 4.0
  role: Source of the extended hand-written rule-based (MSRB) predictor whose SMIRKS
    rule library is being audited
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: ClassyFire
  role: Assigns chemical class labels used to route which compounds are suitable for
    MSRB fragmentation
- name: Docker
  role: Docker image of CFM-ID 4.0 (wishartlab/cfmid) providing a runnable environment
    containing the rule-based predictor
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:f17cdef0fecc3d88cc2d6e80fd11e237547c179ba319265b2b57cece06f24a39
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
yes_rate: 0.7142857142857143
n_claims: 7
---

# msrb-rule-count-verification

## Summary

Audits CFM-ID 4.0's extended hand-written SMIRKS-based (MSRB) fragmentation rule library by parsing the msrb-fragmenter source and Supplemental Table S1, then verifying the rule count equals 313 across 11 ClassyFire chemical classes and 3 adduct types.

## When to use

Use when auditing or reproducing CFM-ID 4.0's SMIRKS-based fragmentation rule set — e.g. verifying that the MSRB library contains 313 rules spanning the reported chemical classes and adduct types before reuse.

## When NOT to use

- When the goal is to predict spectra with the MSRB predictor rather than verify its rule library — run CFM-ID's predictor instead of parsing rule counts
- When analyzing MS/MS spectra from non-QTOF instruments, which CFM-ID 4.0 does not yet handle
- When the fragments/classes of interest lie outside the covered classes (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides, lipids) — the 313-rule library does not cover other chemical classes

## Inputs

- msrb-fragmenter Bitbucket repository source (SMIRKS rule definitions)
- CFM-ID 4.0 Supplemental Table S1 listing SMIRKS-based fragmentation rules
- Experimental QTOF ESI-MS/MS spectra at 10, 20, and 40 eV collision energies (source of scaled relative intensities)

## Outputs

- msrb_rules_table.csv with columns: SMIRKS pattern, chemical class, adduct type, relative intensity
- Verified rule count (expected: 313) with class/adduct coverage breakdown (11 chemical classes, 3 adduct types)

## How to apply

Retrieve the msrb-fragmenter source from Bitbucket and the Supplemental Table S1 listing SMIRKS-based fragmentation rules, then parse each rule's fragmentation reaction pattern, chemical class, and adduct type. Attach to each rule the average scaled relative intensity learned from experimental QTOF ESI-MS/MS spectra acquired at 10, 20, and 40 eV collision energies. Confirm each rule is routed by a ClassyFire class label, since CFM-ID 4.0 uses ClassyFire to identify which chemical classes are suitable for MSRB fragmentation. Compile the rules into a table (SMIRKS pattern, chemical class, adduct type, relative intensity) and count them, verifying the total equals 313 hand-made rules over 11 chemical classes and 3 adduct types as stated in the paper.

## Related tools

- **CFM-ID 4.0** (Source of the extended hand-written rule-based (MSRB) predictor whose SMIRKS rule library is being audited) — https://bitbucket.org/wishartlab/cfm-id-code/
- **ClassyFire** (Assigns chemical class labels used to route which compounds are suitable for MSRB fragmentation)
- **Docker** (Docker image of CFM-ID 4.0 (wishartlab/cfmid) providing a runnable environment containing the rule-based predictor) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
git clone https://bitbucket.org/wishartlab/msrb-fragmenter; python -c "import pandas as pd; t=pd.read_csv('msrb_rules_table.csv'); print(len(t), t['chemical class'].nunique(), t['adduct type'].nunique())"
```

## Evaluation signals

- Compiled rule count equals 313 hand-made MSRB rules
- Class routing covers exactly 11 ClassyFire chemical classes and 3 adduct types
- Covered classes include acylcarnitines, acylcholines, flavonols, flavones, flavanones, and flavonoid glycosides as listed in the article
- Every row in msrb_rules_table.csv has a non-empty SMIRKS pattern, chemical class, adduct type, and average scaled relative intensity
- Independent recount from Supplemental Table S1 reproduces the paper's total

## Limitations

- Relative intensities are averages scaled from QTOF spectra at only three collision energies (10, 20, 40 eV)
- The hand-written rule library covers a limited set of chemical classes — new classes beyond the 11 would require additional rules
- Rules must be extracted from the supplemental table and repository; a complete machine-readable rule list may not be published directly in the article

## Evidence
