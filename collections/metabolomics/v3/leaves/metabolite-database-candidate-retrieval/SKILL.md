---
name: metabolite-database-candidate-retrieval
description: Use when an unknown analyte's precursor m/z (±10 ppm) or exact molecular
  formula is known and a candidate compound list with fingerprints is needed for downstream
  fingerprint-similarity ranking (e.g., MetFID annotation).
when_to_use_negative:
- The analyte lacks both a reliable precursor m/z and an exact molecular formula,
  so there is no valid query key for MetaboQuest or PubChem
- A curated candidate list (inchikey_list.txt blocks) already exists, making retrieval
  redundant
- The adduct or mass tolerance assumptions differ from those used here (H+ adduct,
  10 ppm), which would change which candidates are retrieved
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboQuest
  role: Retrieves a candidate list from the precursor m/z at 10 ppm tolerance assuming
    the H+ adduct
  repo: https://omicscraft.com/MetaboQuest
- name: PubChemPy
  role: Queries the PubChem database by exact molecular formula to narrow the candidate
    list
- name: OpenBabel
  role: Computes the MACCS, FP3, and FP4 fingerprints for every retrieved candidate
    compound
- name: PyFingerprint
  role: Python wrapper used to compute OpenBabel fp3/fp4 fingerprints for each candidate
    structure
  repo: https://github.com/hcji/PyFingerprint
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:bcdc9bd169308e090874d25459593c5ed5713c3ee9716a60fda787230c7fb871
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetFID
  - MetaboQuest
  - MoNA
  - OpenBabel
  - PubChemPy
  - PyFingerprint
  - TensorFlow
  - tabulate
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# metabolite-database-candidate-retrieval

## Summary

Retrieves candidate metabolite structures for an unknown LC-MS/MS analyte by querying MetaboQuest with the precursor m/z (10 ppm, H+ adduct) or PubChem via PubChemPy with the exact molecular formula, then computing each candidate's 528-bit MACCS+FP3+FP4 fingerprint keyed by InChIKey.

## When to use

Use when an unknown analyte's precursor m/z (±10 ppm) or exact molecular formula is known and a candidate compound list with fingerprints is needed for downstream fingerprint-similarity ranking (e.g., MetFID annotation).

## When NOT to use

- The analyte lacks both a reliable precursor m/z and an exact molecular formula, so there is no valid query key for MetaboQuest or PubChem
- A curated candidate list (inchikey_list.txt blocks) already exists, making retrieval redundant
- The adduct or mass tolerance assumptions differ from those used here (H+ adduct, 10 ppm), which would change which candidates are retrieved

## Inputs

- Precursor m/z value parsed from each testing MS/MS spectrum (e.g., 758.5697)
- Molecular formula for each analyte parsed from the testing MS/MS spectra
- Ionization mode and assumed adduct (H+)
- MetaboQuest query endpoint (https://omicscraft.com/MetaboQuest) and PubChem database via PubChemPy

## Outputs

- List of candidate compound InChIKeys retrieved by m/z (10 ppm) and/or exact-formula query
- A 528-entry binary fingerprint (MACCS + FP3 + FP4, computed by OpenBabel) for every candidate
- candidate_compounds.csv containing an InChIKey and its 528-digit fingerprint per candidate for downstream ranking

## How to apply

Parse the precursor m/z and molecular formula from the testing MS/MS spectra, then query MetaboQuest with the precursor m/z at 10 ppm tolerance assuming the H+ adduct (e.g., a search at m/z = 758.5697 with ≤ 10 ppm tolerance returned 12 candidates), and/or query PubChem through the PubChemPy package using the exact molecular formula (formula information shrank average candidate lists from 18 to 9). For every returned candidate, use OpenBabel via PyFingerprint to compute the concatenated MACCS, FP3, and FP4 fingerprint assembled as a 528-digit binary vector, and record each candidate's InChIKey alongside its fingerprint. Write the full list of candidate InChIKeys and fingerprints to candidate_compounds.csv for the downstream fingerprint-prediction ranking step. The rationale is to narrow the search space before ranking: the retrieved candidate set is the ranked pool whose top-1 rate MetFID's CNN (MetFID_CNN_40088_5618.h5) similarity scoring (delta(f',f)=1-sum|f'-f|/n) is evaluated against.

## Related tools

- **MetaboQuest** (Retrieves a candidate list from the precursor m/z at 10 ppm tolerance assuming the H+ adduct) — https://omicscraft.com/MetaboQuest
- **PubChemPy** (Queries the PubChem database by exact molecular formula to narrow the candidate list)
- **OpenBabel** (Computes the MACCS, FP3, and FP4 fingerprints for every retrieved candidate compound)
- **PyFingerprint** (Python wrapper used to compute OpenBabel fp3/fp4 fingerprints for each candidate structure) — https://github.com/hcji/PyFingerprint

## Examples

```
import pubchempy as pcp; candidates = pcp.get_compounds('C43H73NO14', 'formula')
```

## Evaluation signals

- Every retrieved candidate satisfies the query: exact molecular formula match for PubChem queries, and precursor m/z within 10 ppm of the query mass for MetaboQuest queries (H+ adduct assumed)
- candidate_compounds.csv exists and contains one InChIKey plus a 528-entry binary fingerprint for every retrieved candidate
- Adding the molecular-formula filter reduces the candidate list size (average 18 candidates without formula vs. 9 with formula in the paper)
- InChIKeys parse into valid first-block (skeleton) identifiers that OpenBabel/PyFingerprint can convert to structures without errors
- The retrieved candidate sets reproduce the paper's per-analyte counts when re-querying (e.g., 12 candidates for m/z = 758.5697 at ≤ 10 ppm)

## Limitations

- Candidate retrieval depends on database coverage: compounds absent from MetaboQuest/PubChem cannot be recovered, capping achievable top-k annotation accuracy
- The retrieval step fixes the adduct (H+) and tolerance (10 ppm); other adducts or wider tolerances are not covered by the paper's workflow
- Testing spectra in the repo workflow skip training-time merging across collision energies, so retrieval inputs must come from single-spectrum peak lists, not merged spectra

## Evidence
