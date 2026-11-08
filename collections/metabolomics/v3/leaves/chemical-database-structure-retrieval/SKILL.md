---
name: chemical-database-structure-retrieval
description: Use when a predicted Morgan fingerprint bit string from an HSQC spectrum
  (DeepSAT output) is available and candidates must be retrieved, ranked, and annotated
  against known-molecule fingerprint databases.
when_to_use_negative:
- The input is a raw HSQC spectrum rather than an already-predicted Morgan fingerprint
  — the upstream DeepSAT prediction step must run first.
- Authentic reference NMR spectra are available and a direct spectral-library search
  (e.g. peak-shift matching at 0.5 ppm carbon / 0.05 ppm proton tolerance) is the
  appropriate baseline retrieval.
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3314
tools:
- name: DeepSAT
  role: Produces the predicted Morgan fingerprint bit strings from 1H-13C HSQC spectra
    that are compared against candidate-structure fingerprints
  repo: https://github.com/mwang87/DeepSAT
- name: PubChem
  role: Source of candidate molecular structures whose Morgan fingerprints are generated
    for cosine-similarity comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:e24ae993ce311ba687ea8d19eefcf3f856996b43ed0a95001784ab04447e2f6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mwang87/DeepSAT
  tools:
  - DeepSAT
  - PubChem
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# chemical-database-structure-retrieval

## Summary

Retrieve candidate molecular structures for an HSQC query by comparing the DeepSAT-predicted Morgan fingerprint against Morgan fingerprints generated for known small molecules from databases like PubChem, ranking candidates by cosine similarity, and labeling them with cosine-score thresholds (1.0 = identical, >0.8 = similar/annotation).

## When to use

Use when a predicted Morgan fingerprint bit string from an HSQC spectrum (DeepSAT output) is available and candidates must be retrieved, ranked, and annotated against known-molecule fingerprint databases.

## When NOT to use

- The input is a raw HSQC spectrum rather than an already-predicted Morgan fingerprint — the upstream DeepSAT prediction step must run first.
- Authentic reference NMR spectra are available and a direct spectral-library search (e.g. peak-shift matching at 0.5 ppm carbon / 0.05 ppm proton tolerance) is the appropriate baseline retrieval.

## Inputs

- Predicted Morgan fingerprint bit string from the DeepSAT HSQC prediction step
- Candidate molecular structures from a chemical database such as PubChem

## Outputs

- Ranked candidate list with cosine similarity scores and annotation labels (identical/similar), e.g. structure_annotations.csv

## How to apply

Load the predicted Morgan fingerprint bit string produced by the upstream DeepSAT HSQC prediction step. Generate Morgan fingerprints for candidate molecular structures retrieved from chemical databases such as PubChem using RDKit. Compute the cosine similarity between the predicted fingerprint and each candidate fingerprint and rank candidates by descending score. Apply the documented thresholds — a cosine score of 1.0 marks a candidate as identical (identification) and a cosine score over 0.8 marks it as similar (annotation), matching the annotation-rate cutoff and the reported average predicted-vs-ground-truth cosine score of 0.8450. Write the ranked candidate list with similarity scores and annotation labels (identical/similar) to the output table.

## Related tools

- **DeepSAT** (Produces the predicted Morgan fingerprint bit strings from 1H-13C HSQC spectra that are compared against candidate-structure fingerprints) — https://github.com/mwang87/DeepSAT
- **PubChem** (Source of candidate molecular structures whose Morgan fingerprints are generated for cosine-similarity comparison)

## Examples

```
python deepsat_search.py --fingerprint predicted_bits.txt --library pubchem_smiles.smi --threshold 0.8 --out structure_annotations.csv
```

## Evaluation signals

- Every reported cosine score lies within [0, 1].
- The 0.8 similarity threshold is applied as the annotation cutoff, consistent with the reported average predicted-vs-ground-truth fingerprint cosine score of 0.8450 (normal HSQC) / 0.8574 (Multiplicity Edited HSQC).
- Cosine score of exactly 1.0 is labeled 'identical'; scores over 0.8 are labeled 'similar'.
- Candidates are ranked by descending cosine score before thresholding.
- Output table contains per-candidate similarity scores plus annotation labels.

## Limitations

- Interpretability of retrieval decisions is limited: occlusion-sensitivity analysis covers a single compound (quercetin) and 'these results do not fully explain the decision-making process of DeepSAT'.
- Solvent-transfer robustness is incomplete: fewer than all results were correctly identified across solvents, though over 65% were still correctly identified.
- The open NMR reference databases underlying candidate coverage cover limited chemical diversity compared to the enormous chemical diversity of nature.

## Evidence

- [results] structures considered similar: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
- [results] annotation threshold: "For the measurement of annotation rate, cosine score of 0.8 was set as the threshold"
- [results] average cosine score: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [discussion] fingerprint comparison: "these fingerprints can also be easily generated from all known small molecules to which DeepSAT results can be compared for similarity"
- [discussion] fingerprint prediction: "learned to recognize features of a molecule from HSQC spectra that could be used to populate fingerprint bit strings"
