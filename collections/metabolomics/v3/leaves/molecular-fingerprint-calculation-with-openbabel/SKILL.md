---
name: molecular-fingerprint-calculation-with-openbabel
description: Use when candidate compounds have been retrieved (e.g. via PubChem or
  MetaboQuest queries) and each candidate needs a fixed-length molecular fingerprint
  vector to compare with a predicted fingerprint.
when_to_use_negative:
- Input is already a table of fingerprint vectors — this skill computes fingerprints
  from structures, not similarity scores.
- The unknown compound's fingerprint is what is needed — use a fingerprint prediction
  model (e.g. MetFID CNN) rather than OpenBabel, since OpenBabel requires an explicit
  structure.
- Only structural similarity between known structures is needed without MS/MS context
  — general cheminformatics fingerprint tools suffice and the MetFID ranking pipeline
  adds unnecessary overhead.
edam_operation: http://edamontology.org/operation_0247
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: OpenBabel
  role: Computes MACCS, FP3, and FP4 fingerprints for candidate compounds, assembled
    into a 528-digit binary vector
  repo: https://github.com/aohongyu/MetFID
- name: PyFingerprint
  role: Python package exposing OpenBabel fingerprint types (fp2, fp3, fp4) via get_fingerprint
    for programmatic computation
  repo: https://github.com/hcji/PyFingerprint
- name: PubChemPy
  role: Queries PubChem by molecular formula to produce the candidate list that fingerprints
    are calculated for
- name: MetFID
  role: 'Downstream consumer: uses the 528-digit fingerprints to score candidate similarity
    against the CNN-predicted fingerprint'
  repo: https://github.com/aohongyu/MetFID
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:5cb70138947285c6ed4ee8bea1fbf3bd209f10e1e6b3ff920d8e3c3c865a3c81
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - MetFID
  - MetaboQuest
  - OpenBabel
  - PubChemPy
  - PyFingerprint
  edam_operation: http://edamontology.org/operation_0247
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# molecular-fingerprint-calculation-with-openbabel

## Summary

Compute concatenated MACCS, FP3, and FP4 fingerprints (528 binary entries) for every candidate compound retrieved from a database query, using OpenBabel as implemented in the MetFID pipeline. These fingerprint vectors enable Tanimoto-style similarity scoring against the fingerprint predicted from an MS/MS spectrum.

## When to use

Use when candidate compounds have been retrieved (e.g. via PubChem or MetaboQuest queries) and each candidate needs a fixed-length molecular fingerprint vector to compare with a predicted fingerprint.

## When NOT to use

- Input is already a table of fingerprint vectors — this skill computes fingerprints from structures, not similarity scores.
- The unknown compound's fingerprint is what is needed — use a fingerprint prediction model (e.g. MetFID CNN) rather than OpenBabel, since OpenBabel requires an explicit structure.
- Only structural similarity between known structures is needed without MS/MS context — general cheminformatics fingerprint tools suffice and the MetFID ranking pipeline adds unnecessary overhead.

## Inputs

- Candidate compound list (InChIKeys or structure files) retrieved from PubChem or MetaboQuest
- Molecular formula string (e.g. via PubChemPy) used to fetch candidates
- Candidate compound CSV (candidate_compounds.csv) with InChIKey and fingerprint columns

## Outputs

- 528-entry binary fingerprint vector per candidate compound (concatenated MACCS + FP3 + FP4)
- InChIKey-to-fingerprint mapping table for downstream similarity ranking
- candidate_compounds.csv containing candidate InChIKeys and fingerprints

## How to apply

After candidate retrieval (e.g. querying MetaboQuest at m/z = 758.5697 with ≤ 10 ppm tolerance, or PubChem via PubChemPy by exact molecular formula), pass each candidate compound to OpenBabel to calculate its molecular fingerprints. Assemble the MACCS, FP3, and FP4 fingerprints into a single vector comprising 528 binary entries, which is the representation MetFID trains and predicts against. Record each candidate's InChIKey together with its 528-digit fingerprint so downstream ranking can score fingerprint similarity between the predicted fingerprint and each candidate. The PyFingerprint package (get_fingerprint) exposes OpenBabel's fp2/fp3/fp4 types programmatically and can be used as the Python interface to OpenBabel. Validate that every candidate yields a 528-entry binary fingerprint and that fingerprints match the candidate's InChIKey.

## Related tools

- **OpenBabel** (Computes MACCS, FP3, and FP4 fingerprints for candidate compounds, assembled into a 528-digit binary vector) — https://github.com/aohongyu/MetFID
- **PyFingerprint** (Python package exposing OpenBabel fingerprint types (fp2, fp3, fp4) via get_fingerprint for programmatic computation) — https://github.com/hcji/PyFingerprint
- **PubChemPy** (Queries PubChem by molecular formula to produce the candidate list that fingerprints are calculated for)
- **MetFID** (Downstream consumer: uses the 528-digit fingerprints to score candidate similarity against the CNN-predicted fingerprint) — https://github.com/aohongyu/MetFID

## Examples

```
from PyFingerprint.fingerprint import get_fingerprint; fp = get_fingerprint(smi, 'fp3')
```

## Evaluation signals

- Every candidate compound in the retrieval output has a 528-entry binary fingerprint associated with its InChIKey.
- Fingerprint vector length is exactly 528 (MACCS + FP3 + FP4 concatenated) and contains only binary entries.
- Number of fingerprint records equals the number of candidate compounds returned by the retrieval step (e.g. 12 candidates for m/z 758.5697 at 10 ppm).
- fingerprints for the same InChIKey are deterministic across repeated runs of OpenBabel.
- The candidate list in candidate_compounds.csv matches the query: exact formula for PubChem or precursor m/z within 10 ppm for MetaboQuest.

## Limitations

- OpenBabel requires an explicit compound structure; it cannot compute fingerprints from MS/MS spectra alone — prediction of fingerprints from spectra is the MetFID CNN's job, not OpenBabel's.
- PyFingerprint requires Java SE Development Kit 11 and a dedicated conda environment, adding setup complexity.
- The 528-bit concatenated MACCS/FP3/FP4 representation is fixed; other fingerprint types (e.g. Morgan, ECFP6, Klekota-Roth) exposed by PyFingerprint are not used by MetFID's trained model.

## Evidence

- [readme] README confirms PyFingerprint exposes OpenBabel fp3/fp4 types via get_fingerprint: [evidence span withheld — anchor status: external_doc]
- [readme] README documents the PyFingerprint get_fingerprint call pattern used for fingerprint computation: [evidence span withheld — anchor status: external_doc]
- [other] Task card defines the retrieval-to-fingerprint output artifact: [evidence span withheld — anchor status: not_found]
