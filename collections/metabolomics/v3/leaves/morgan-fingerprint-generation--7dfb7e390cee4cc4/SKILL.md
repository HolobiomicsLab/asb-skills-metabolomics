---
name: morgan-fingerprint-generation
description: Use when you have a predicted Morgan fingerprint bit string from a DeepSAT
  HSQC prediction and need to retrieve, rank, and annotate candidate molecular structures
  from databases like PubChem by fingerprint cosine similarity.
when_to_use_negative:
- You have raw 1H-13C HSQC peak lists or spectra but no predicted fingerprint — the
  upstream DeepSAT fingerprint prediction step must run first.
- Your reference data are proton/carbon chemical-shift libraries searched by shift
  matching (0.5 ppm carbon / 0.05 ppm proton thresholds) rather than fingerprint-based
  candidate retrieval.
- You need full structure elucidation of a single compound with expert rules (e.g.,
  ACD/structure elucidator, CMC-se) rather than similarity-ranked candidate retrieval.
tools:
- name: DeepSAT
  role: Produces the predicted fingerprint bit string from the query HSQC spectrum
    that is compared against candidate fingerprints
  repo: github.com/mwang87/DeepSAT
- name: PubChem
  role: Source of candidate molecular structures whose Morgan fingerprints are generated
    for cosine-score comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:54fd8f72b161d5f295b2b89dce8640564aef53d0e207d5e9816dee3e099b97ab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  - PubChem
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# morgan-fingerprint-generation

## Summary

Generates Morgan fingerprints from known small-molecule structures and compares them to DeepSAT's predicted fingerprint bit strings via cosine similarity, ranking candidate structures and labeling them identical (score 1.0) or similar (> 0.8) for HSQC-based structure annotation.

## When to use

Use when you have a predicted Morgan fingerprint bit string from a DeepSAT HSQC prediction and need to retrieve, rank, and annotate candidate molecular structures from databases like PubChem by fingerprint cosine similarity.

## When NOT to use

- You have raw 1H-13C HSQC peak lists or spectra but no predicted fingerprint — the upstream DeepSAT fingerprint prediction step must run first.
- Your reference data are proton/carbon chemical-shift libraries searched by shift matching (0.5 ppm carbon / 0.05 ppm proton thresholds) rather than fingerprint-based candidate retrieval.
- You need full structure elucidation of a single compound with expert rules (e.g., ACD/structure elucidator, CMC-se) rather than similarity-ranked candidate retrieval.

## Inputs

- Predicted Morgan fingerprint bit string from the DeepSAT HSQC prediction step
- Candidate molecular structures from a chemical database (e.g., PubChem) as parseable structure records (SMILES/MOL)
- Reference in-house HSQC library context (n=143,467) from which candidates may be retrieved

## Outputs

- structure_annotations.csv with ranked candidate structures, cosine similarity scores, and annotation labels (identical/similar)

## How to apply

Take the predicted Morgan fingerprint bit string emitted by the upstream DeepSAT prediction step and generate Morgan fingerprints (e.g., with RDKit) for candidate structures pulled from chemical databases such as PubChem. Compute the cosine similarity between the predicted fingerprint and each candidate fingerprint, then rank candidates by descending score. Apply DeepSAT's documented thresholds: a cosine score of 1.0 marks a candidate as identical (identification), and a score over 0.8 marks it as similar (annotation cutoff, consistent with the reported mean predicted-vs-ground-truth cosine score of 0.8450). Write the ranked candidate list with similarity scores and identical/similar labels to structure_annotations.csv.

## Related tools

- **DeepSAT** (Produces the predicted fingerprint bit string from the query HSQC spectrum that is compared against candidate fingerprints) — github.com/mwang87/DeepSAT
- **PubChem** (Source of candidate molecular structures whose Morgan fingerprints are generated for cosine-score comparison)

## Examples

```
from rdkit.Chem import rdFingerprintGenerator as rfg; fp = rfg.GetMorganGenerator().GetFingerprint(mol)
```

## Evaluation signals

- Every reported cosine score lies within [0, 1].
- The 0.8 threshold is applied as the annotation cutoff: only candidates scoring over 0.8 are labeled 'similar', and 1.0 labels 'identical'.
- Predicted-vs-ground-truth fingerprint cosine scores are consistent with the paper's reported averages (0.8450 normal HSQC; 0.8574 multiplicity-edited HSQC).
- The candidate ranking in structure_annotations.csv is monotonically ordered by descending cosine score, and each row carries an annotation label.

## Limitations

- Candidate coverage depends on the underlying database (e.g., PubChem) and the in-house HSQC library (n=143,467); known open NMR libraries like NMRShiftDB underperformed (< 3% correct identifications).
- Solvent-transfer robustness is incomplete: over 65% of results were correctly identified when spectra were recorded in different solvents.
- A cosine score over 0.8 indicates similarity, not structural identity; the annotation threshold is a proxy validated by the paper's average score of ~0.85.

## Evidence

- [results] we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
- [discussion] fingerprints generated from known small molecules: "these fingerprints can also be easily generated from all known small molecules to which DeepSAT results can be compared for similarity"
- [results] average cosine score was 0.8450 for the normal HSQC model: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [intro] PubChem as candidate structure source: "molecular structure databases such as Pubchem [36] contain millions of compounds"
- [results] Morgan fingerprints for annotation rate threshold: "For the measurement of annotation rate, cosine score of 0.8 was set as the threshold"
