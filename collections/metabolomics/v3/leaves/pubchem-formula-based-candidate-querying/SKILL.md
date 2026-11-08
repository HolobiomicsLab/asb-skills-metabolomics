---
name: pubchem-formula-based-candidate-querying
description: Use when annotating an LC-MS analyte whose molecular formula is known
  and candidate lists must be filtered before ranking candidates using fingerprint
  prediction methods like MetFID.
when_to_use_negative:
- Use mass-based candidate retrieval (e.g. MetaboQuest at 10 ppm tolerance) when the
  exact molecular formula is unknown, since formula-based querying requires the exact
  formula.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_0218
tools:
- name: PubChemPy
  role: Python package used to query the PubChem database based on molecular formula
- name: MetaboQuest
  role: Alternative mass-based candidate retrieval tool (m/z 758.5697, 10 ppm tolerance,
    H+ adduct)
- name: OpenBabel
  role: Determines MACCS, FP3, FP4 fingerprints assembled into a 528-digit binary
    vector for each retrieved candidate
- name: MetFID
  role: Consumes retrieved candidate InChIKeys to rank by fingerprint similarity against
    the ANN-predicted fingerprint
  repo: https://github.com/aohongyu/MetFID
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:aaa08f26db79e055f294c96f382cb49c1030368154d9408fc161425aabb1623d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - MetFID
  - MetaboQuest
  - OpenBabel
  - PubChemPy
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# pubchem-formula-based-candidate-querying

## Summary

Narrow the search space for metabolite annotation by retrieving candidate compounds from PubChem via the PubChemPy package, using the exact molecular formula of each analyte.

## When to use

Use when annotating an LC-MS analyte whose molecular formula is known and candidate lists must be filtered before ranking candidates using fingerprint prediction methods like MetFID.

## When NOT to use

- Use mass-based candidate retrieval (e.g. MetaboQuest at 10 ppm tolerance) when the exact molecular formula is unknown, since formula-based querying requires the exact formula.

## Inputs

- molecular formula (exact, from MS/MS or formula prediction)
- PubChemPy API access to PubChem database

## Outputs

- list of candidate InChIKeys (one per formula-matched compound)
- candidate_compounds.csv with InChIKey and 528-digit fingerprint per candidate

## How to apply

Query the PubChem database based on molecular formula via the PubChemPy package to generate a candidate list. Each candidate's InChIKey is used to assemble a 528-digit binary fingerprint via OpenBabel (MACCS, FP3, FP4). MetFID then feeds the MS/MS spectrum to an ANN to predict the fingerprint, ranks candidates by fingerprint similarity (top-ranked is used for annotation). Formula filtering reduced average candidate list size from 18 to 9 in the paper.

## Related tools

- **PubChemPy** (Python package used to query the PubChem database based on molecular formula)
- **MetaboQuest** (Alternative mass-based candidate retrieval tool (m/z 758.5697, 10 ppm tolerance, H+ adduct))
- **OpenBabel** (Determines MACCS, FP3, FP4 fingerprints assembled into a 528-digit binary vector for each retrieved candidate)
- **MetFID** (Consumes retrieved candidate InChIKeys to rank by fingerprint similarity against the ANN-predicted fingerprint) — https://github.com/aohongyu/MetFID

## Examples

```
python3 main.py  # in MetFID repo, with inchikey_list.txt populated from PubChemPy formula-query results
```

## Evaluation signals

- Every candidate returned has an InChIKey and a 528-entry binary fingerprint in candidate_compounds.csv.
- PubChemPy queries must return compounds matching the exact molecular formula (no formula mismatch).
- Fingerprint vector length must be exactly 528 entries (MACCS + FP3 + FP4 concatenated).
- Candidate list sizes should reflect formula filtering (average list size reduced from 18 to 9 in the paper's evaluation).

## Limitations

- Requires an exact molecular formula; formula errors propagate directly to the candidate list.
- Formula-based filtering only narrows the search; downstream ranking quality still depends on the fingerprint prediction model.

## Evidence

- [results] formula information shrank average candidate lists from 18 to 9: [evidence span withheld — anchor status: not_found]
- [readme] MetFID requires PyFingerprint, PubChemPy, OpenBabel, and a pre-trained CNN model: [evidence span withheld — anchor status: not_found]
