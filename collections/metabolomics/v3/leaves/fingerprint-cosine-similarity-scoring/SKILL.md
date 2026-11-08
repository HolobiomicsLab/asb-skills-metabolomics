---
name: fingerprint-cosine-similarity-scoring
description: Use when a predicted Morgan fingerprint bit string from an HSQC spectrum
  must be compared against reference fingerprints from chemical databases such as
  PubChem to retrieve, rank, and label (identical/similar) candidate structures.
when_to_use_negative:
- When authentic reference NMR spectra exist and direct spectral-library search (e.g.
  SMART 2.0-style shift matching) is preferable
- When the input is raw peak lists or spectra rather than a predicted fingerprint
  bit string — fingerprint prediction must occur first
- When ranking candidate structures requires exact structure elucidation rather than
  similarity-based annotation
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_0091
tools:
- name: DeepSAT
  role: Predicts the Morgan fingerprint bit string (plus molecular weight and NPClassifier
    class) from the input 1H-13C HSQC spectrum that is then cosine-scored against
    candidates
  repo: github.com/mwang87/DeepSAT
- name: PubChem
  role: Source of known molecular structures from which candidate Morgan fingerprints
    are generated for similarity comparison
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_002
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:e6fe8c02fd45b780cff2c3eae0f2f9589579c00bcabcee15a7d2392a406f6936
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  - NMRShiftDB
  - PubChem
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# fingerprint-cosine-similarity-scoring

## Summary

Ranks candidate molecular structures by computing cosine similarity between the Morgan fingerprint predicted from a query 1H-13C HSQC spectrum (via DeepSAT) and Morgan fingerprints generated from known small molecules, applying thresholds of 1.0 for identical (identification) and >0.8 for similar (annotation).

## When to use

Use when a predicted Morgan fingerprint bit string from an HSQC spectrum must be compared against reference fingerprints from chemical databases such as PubChem to retrieve, rank, and label (identical/similar) candidate structures.

## When NOT to use

- When authentic reference NMR spectra exist and direct spectral-library search (e.g. SMART 2.0-style shift matching) is preferable
- When the input is raw peak lists or spectra rather than a predicted fingerprint bit string — fingerprint prediction must occur first
- When ranking candidate structures requires exact structure elucidation rather than similarity-based annotation

## Inputs

- Predicted Morgan fingerprint bit string from DeepSAT inference on a query 1H-13C HSQC spectrum
- Known small-molecule candidate structures (e.g. from PubChem) for fingerprint generation
- Ground-truth Morgan fingerprints of query structures for benchmark scoring

## Outputs

- structure_annotations.csv with ranked candidates, cosine similarity scores, and identical/similar labels
- Annotation rate (fraction of queries with cosine score > 0.8) and Top-1 identification rates
- Precision, recall, and F1@k computed at the 0.8 cosine-similarity threshold

## How to apply

Load the predicted Morgan fingerprint bit string from the upstream DeepSAT prediction step, then generate Morgan fingerprints for candidate structures retrieved from databases such as PubChem using RDKit. Compute cosine similarity between predicted and each candidate fingerprint and rank candidates by descending score. Apply the paper's thresholds: cosine score 1.0 marks a candidate as identical (identification) and a cosine score over 0.8 marks it as similar; the 0.8 cutoff is also the threshold used for annotation-rate measurement and precision/recall/F1@k computation. Write the ranked candidate list with similarity scores and annotation labels to structure_annotations.csv. Correctness can be checked by confirming all scores lie in [0,1] and that the reported average predicted-vs-ground-truth cosine score (0.8450 normal HSQC, 0.8574 multiplicity-edited HSQC) is consistent with the threshold regime.

## Related tools

- **DeepSAT** (Predicts the Morgan fingerprint bit string (plus molecular weight and NPClassifier class) from the input 1H-13C HSQC spectrum that is then cosine-scored against candidates) — github.com/mwang87/DeepSAT
- **PubChem** (Source of known molecular structures from which candidate Morgan fingerprints are generated for similarity comparison)

## Examples

```
python score_fingerprints.py --predicted_fp predicted_bits.txt --candidates pubchem.smi --threshold 0.8 --identical 1.0 --out structure_annotations.csv
```

## Evaluation signals

- Every computed cosine score lies within [0, 1]
- 0.8 is applied as the similarity/annotation cutoff and 1.0 as the identical/identification cutoff
- Average predicted-vs-ground-truth fingerprint cosine score reproduces reported values (0.8450 normal HSQC; 0.8574 multiplicity-edited HSQC)
- Annotation rates on held-out spectra stay consistent with reported Top-1 annotation performance (e.g. 73.5% with multiplicity-edited HSQC vs 40.3% for SMART 2.0)
- Cross-solvent evaluation (methanol-d4 vs chloroform-d) retains over 65% correct identifications

## Limitations

- Interpretability is limited: occlusion-sensitivity analysis on a single compound (quercetin) 'do not fully explain the decision-making process of DeepSAT'
- Solvent transfer is imperfect — fewer than all results were correctly identified across methanol-d4 and chloroform-d (over 65% retention)
- Searches NMRshiftDB reference spectra by proton and carbon shifts achieve under 3% correct identifications, so shift-based retrieval cannot substitute for fingerprint cosine scoring

## Evidence

- [results] Structures considered similar if cosine score over 0.8 based on predicted and known Morgan fingerprints: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
- [results] Annotation threshold of cosine score 0.8 set for annotation rate measurement: "For the measurement of annotation rate, cosine score of 0.8 was set as the threshold"
- [discussion] Predicted fingerprints can be generated from all known small molecules for comparison: "these fingerprints can also be easily generated from all known small molecules to which DeepSAT results can be compared for similarity"
- [results] Average cosine score between predicted and ground-truth fingerprints reported per model branch: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [results] Baseline retrieval thresholds for shift matching in benchmark comparison: "The thresholds for the peak shift differences were set as 0.5 ppm for carbon signals and 0.05 ppm for proton signals"
