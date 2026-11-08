---
name: inchikey-compound-identifier-handling
description: Reference for pipelines that must merge MS/MS spectra from the same compound
  across collision energies or align predicted fingerprints with candidate compound
  lists drawn from MetaboQuest/PubChem.
when_to_use_negative:
- When the compound identity is only a molecular formula or m/z value and no InChIKey
  has been resolved yet — first retrieve candidates via MetaboQuest or PubChemPy before
  InChIKey-based ranking.
- When working with spectral library entries that lack InChIKeys (e.g. some MoNA records);
  fall back to compound name or SMILES instead of forcing an InChIKey merge.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: MetFID
  role: Consumes InChIKey-annotated spectra and InChIKey candidate lists; merges spectra
    and computes fingerprint similarity per InChIKey block
  repo: https://github.com/aohongyu/MetFID
- name: OpenBabel
  role: Computes the 528-digit MACCS/FP3/FP4 fingerprint stored alongside each candidate's
    InChIKey
- name: PubChemPy
  role: Queries PubChem by molecular formula to obtain candidate compounds identified
    by InChIKey
- name: PyFingerprint
  role: Alternative fingerprint library referenced in the MetFID prerequisites for
    fingerprint generation
  repo: https://github.com/hcji/PyFingerprint
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:2ff343e3b798e2816e6c70b54c75b19058b68c0c3e3dbebf63cd7938af417b44
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - InChIKey
  - MetFID
  - MetaboQuest
  - OpenBabel
  - PubChemPy
  - PyFingerprint
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# inchikey-compound-identifier-handling

## Summary

Standardizes compound identity across the MetFID pipeline by using InChIKey as the compound identifier for merging spectra, listing candidates, and scoring fingerprint similarity.

## When to use

Reference for pipelines that must merge MS/MS spectra from the same compound across collision energies or align predicted fingerprints with candidate compound lists drawn from MetaboQuest/PubChem.

## When NOT to use

- When the compound identity is only a molecular formula or m/z value and no InChIKey has been resolved yet — first retrieve candidates via MetaboQuest or PubChemPy before InChIKey-based ranking.
- When working with spectral library entries that lack InChIKeys (e.g. some MoNA records); fall back to compound name or SMILES instead of forcing an InChIKey merge.

## Inputs

- MS/MS spectra annotated with the InChIKey of the originating compound (e.g. from MoNA or NIST 17 libraries)
- Candidate compound lists keyed by InChIKey (from MetaboQuest m/z search or PubChem formula queries via PubChemPy)
- inchikey_list.txt file containing InChIKey blocks separated by '#digit'
- testing_compound.txt file containing spectrum blocks separated by '#digit'

## Outputs

- Merged spectra keyed by InChIKey, one per compound across collision energies
- testing_compound_prediction.txt listing candidate InChIKeys and their fingerprint similarity scores for each spectrum block
- candidate_compounds.csv mapping each candidate InChIKey to its 528-digit binary fingerprint

## How to apply

Use the InChIKey as the canonical compound identifier wherever compounds are referenced: merge multiple MS/MS spectra with different collision energies acquired from the same compound by joining on InChIKey; store retrieved candidates with their InChIKeys so fingerprint similarity scores can be computed between the predicted 528-digit MACCS/FP3/FP4 fingerprint and each candidate's OpenBabel-derived fingerprint; and, in the MetFID runner, supply candidate InChIKeys in an inchikey_list.txt file whose blocks separated by '#digit' match the spectrum blocks in testing_compound.txt. Because InChIKey blocks are matched by position, an error is raised if the counts in the two files disagree, which serves as a built-in integrity check that every spectrum has a corresponding candidate set.

## Related tools

- **MetFID** (Consumes InChIKey-annotated spectra and InChIKey candidate lists; merges spectra and computes fingerprint similarity per InChIKey block) — https://github.com/aohongyu/MetFID
- **OpenBabel** (Computes the 528-digit MACCS/FP3/FP4 fingerprint stored alongside each candidate's InChIKey)
- **PubChemPy** (Queries PubChem by molecular formula to obtain candidate compounds identified by InChIKey)
- **PyFingerprint** (Alternative fingerprint library referenced in the MetFID prerequisites for fingerprint generation) — https://github.com/hcji/PyFingerprint

## Examples

```
python3 main.py  # expects _files/testing_compound.txt and _files/inchikey_list.txt with matching #digit block counts
```

## Evaluation signals

- The counts of '#digit' blocks in testing_compound.txt and inchikey_list.txt match; otherwise MetFID raises an error.
- Every merged spectrum is keyed by a valid InChIKey of the form XXXXXXXXXX-XXXXXXXXXX-N.
- Each candidate entry in candidate_compounds.csv pairs one InChIKey with a 528-entry binary fingerprint.
- testing_compound_prediction.txt lists exactly the InChIKeys supplied in the corresponding inchikey_list.txt block.
- Spectra from the same InChIKey but different collision energies are merged into a single record rather than duplicated.

## Limitations

- InChIKey-based merging requires that every spectrum and every candidate carry a valid InChIKey; missing identifiers break the join.
- Candidate lists are user-supplied and must be correctly ordered; mismatched blocks cause the program to error out.
- InChIKey does not capture stereochemical or adduct-specific ambiguity beyond the standard three-block form used here.

## Evidence
