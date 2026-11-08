---
name: mass-based-database-searching
description: Use when an LC-MS/MS analyte has a precursor m/z and/or molecular formula
  and a candidate shortlist is needed for fingerprint prediction-based metabolite
  annotation (e.g. MetFID/MetFrag-style ranking).
when_to_use_negative:
- The analyte already has an annotated reference MS/MS spectrum available, so direct
  spectral-library matching (against HMDB, METLIN, GNPS) suffices instead of candidate
  retrieval and fingerprint ranking.
- No precursor m/z or molecular formula is available for the analyte, since both MetaboQuest
  mass search and PubChemPy formula queries require at least one of these.
- A large unsupervised survey where candidate lists cannot be narrowed — the article
  notes formula information shrank lists from 18 to 9, so applying this without any
  constraining metadata yields unmanageable candidate counts.
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: MetaboQuest
  role: Web service queried with precursor m/z (10 ppm tolerance, H+ adduct assumption)
    to retrieve the candidate list
  repo: https://omicscraft.com/MetaboQuest
- name: PubChemPy
  role: Python package used to query the PubChem database by exact molecular formula
    to obtain candidate compounds
- name: OpenBabel
  role: Computes the concatenated MACCS, FP3, and FP4 fingerprints (528 binary entries)
    for every retrieved candidate
- name: MetFID
  role: Downstream CNN fingerprint predictor that consumes the retrieved candidate
    shortlist for similarity ranking
  repo: https://github.com/aohongyu/MetFID
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:f130865085af694cda907ec6445b8bf88c076e8c119d7e44da99d3d18f5a0922
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetFID
  - MetaboQuest
  - OpenBabel
  - PubChemPy
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mass-based-database-searching

## Summary

Retrieves a list of candidate compounds from a compound database by querying with the precursor m/z (within a tolerance) and/or the exact molecular formula, producing the shortlist that downstream fingerprint-similarity ranking will operate on.

## When to use

Use when an LC-MS/MS analyte has a precursor m/z and/or molecular formula and a candidate shortlist is needed for fingerprint prediction-based metabolite annotation (e.g. MetFID/MetFrag-style ranking).

## When NOT to use

- The analyte already has an annotated reference MS/MS spectrum available, so direct spectral-library matching (against HMDB, METLIN, GNPS) suffices instead of candidate retrieval and fingerprint ranking.
- No precursor m/z or molecular formula is available for the analyte, since both MetaboQuest mass search and PubChemPy formula queries require at least one of these.
- A large unsupervised survey where candidate lists cannot be narrowed — the article notes formula information shrank lists from 18 to 9, so applying this without any constraining metadata yields unmanageable candidate counts.

## Inputs

- Precursor m/z value parsed from a testing MS/MS spectrum (e.g. 758.5697)
- Molecular formula of the analyte (e.g. C42H72NO10P)
- Ionization/adduct assumption (e.g. H+ adduct)
- Mass tolerance setting (10 ppm in the MetFID workflow)

## Outputs

- Candidate compound shortlist (e.g. 12 candidates for m/z 758.5697 at ≤ 10 ppm)
- For each candidate: InChIKey identifier
- For each candidate: concatenated MACCS, FP3, FP4 fingerprint as a 528-digit binary vector (computed with OpenBabel)
- candidate_compounds.csv listing InChIKeys and fingerprints for downstream fingerprint-similarity ranking

## How to apply

Parse the precursor m/z and molecular formula from each testing MS/MS spectrum, then query MetaboQuest with the precursor m/z at 10 ppm tolerance assuming the H+ adduct, or query PubChem through the PubChemPy package using the exact molecular formula. In the MetFID paper this step shrank average candidate lists from 18 to 9 compounds when formula information was available, and a mass search at m/z 758.5697 with ≤ 10 ppm tolerance returned 12 candidates. For every candidate returned, use OpenBabel to calculate the concatenated MACCS, FP3, and FP4 fingerprints assembled as a 528-digit binary vector, and record each candidate's InChIKey together with its fingerprint. The rationale is that a small, chemically plausible candidate set is required before the fingerprint predictor and ranking step can sensibly select the top hit; without retrieval, ranking is meaningless. Validation: the retrieved candidates must match the query — exact formula for PubChem, precursor m/z within 10 ppm for MetaboQuest — and every candidate must carry a valid InChIKey and 528-entry fingerprint before writing to candidate_compounds.csv.

## Related tools

- **MetaboQuest** (Web service queried with precursor m/z (10 ppm tolerance, H+ adduct assumption) to retrieve the candidate list) — https://omicscraft.com/MetaboQuest
- **PubChemPy** (Python package used to query the PubChem database by exact molecular formula to obtain candidate compounds)
- **OpenBabel** (Computes the concatenated MACCS, FP3, and FP4 fingerprints (528 binary entries) for every retrieved candidate)
- **MetFID** (Downstream CNN fingerprint predictor that consumes the retrieved candidate shortlist for similarity ranking) — https://github.com/aohongyu/MetFID

## Examples

```
python3 main.py  # MetFID run that ingests testing_compound.txt precursor m/z/formula and inchikey_list.txt candidates, producing testing_compound_prediction.txt in _files/
```

## Evaluation signals

- For PubChemPy retrieval, every returned candidate's molecular formula equals the queried formula exactly.
- For MetaboQuest mass search, every returned candidate's precursor mass lies within 10 ppm of the queried m/z (e.g. 758.5697 at ≤ 10 ppm should yield 12 candidates).
- Every candidate entry has a syntactically valid InChIKey (e.g. 14-character hyphen-separated block format like XSJLXZULZZXNJI-UHFFFAOYSA-N).
- Every candidate entry has a 528-entry binary fingerprint vector produced by OpenBabel (MACCS + FP3 + FP4).
- The candidate file (candidate_compounds.csv) exists, is non-empty, and contains one row per retrieved candidate with InChIKey and fingerprint columns.

## Limitations

- Candidate retrieval depends on adduct assumption (H+); a wrong adduct guess will miss the true compound in the retrieved list.
- PubChemPy formula queries and MetaboQuest mass searches rely on third-party databases, so coverage gaps or stale database mirrors will silently shrink the candidate set.
- A molecular formula query narrows the list but cannot distinguish isomers, so formula-only retrieval may still leave multiple equally plausible candidates requiring downstream fingerprint ranking.

## Evidence
