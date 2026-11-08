---
name: fingerprint-similarity-scoring
description: Use when ranking candidate compounds for an analyte whose fingerprint
  has already been predicted from an MS/MS spectrum — score each candidate's 528-digit
  MACCS/FP3/FP4 fingerprint against the prediction and pick the top-ranked candidate.
when_to_use_negative:
- Input is not a molecular fingerprint pair but a raw MS/MS spectrum — spectrum-based
  spectral matching (HMDB, METLIN, GNPS libraries) is the wrong comparison method
  here
- Candidate structures are missing or unavailable, so OpenBabel fingerprints cannot
  be computed for candidates
- Only a small pre-defined candidate set is already known and no ranking is needed
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: OpenBabel
  role: Computes the 528-digit MACCS, FP3, and FP4 molecular fingerprints for candidate
    compounds
- name: PyFingerprint
  role: Python wrapper used to call OpenBabel fingerprints (fp3, fp4, maccs) for each
    candidate compound
  repo: https://github.com/hcji/PyFingerprint
- name: MetaboQuest
  role: Retrieves candidate compound lists by precursor m/z search with 10 PPM tolerance
    and H+ adduct assumption
  repo: https://omicscraft.com/MetaboQuest
- name: PubChemPy
  role: Queries the PubChem database by molecular formula to obtain candidate compounds
    when formula-based retrieval is needed
- name: MetFID
  role: Source of the predicted fingerprint that is scored against candidate fingerprints
  repo: https://github.com/aohongyu/MetFID
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_001
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:e174c519437e69877b5712fb63543add597d20ec161e5807dfa67c9cae0900e1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/PyFingerprint
  tools:
  - MetFID
  - MetaboQuest
  - MoNA
  - NIST 17
  - OpenBabel
  - PubChemPy
  - PyFingerprint
  - RDKit
  - TensorFlow
  - tabulate
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# fingerprint-similarity-scoring

## Summary

Computes a similarity score between a predicted molecular fingerprint and candidate fingerprints (assembled from OpenBabel MACCS, FP3, and FP4 binary entries) using δ(f′,f) = 1 − Σ|f′−f|/n, then ranks candidates to select the annotation. This is the decision step after MetFID's ANN fingerprint prediction.

## When to use

Use when ranking candidate compounds for an analyte whose fingerprint has already been predicted from an MS/MS spectrum — score each candidate's 528-digit MACCS/FP3/FP4 fingerprint against the prediction and pick the top-ranked candidate.

## When NOT to use

- Input is not a molecular fingerprint pair but a raw MS/MS spectrum — spectrum-based spectral matching (HMDB, METLIN, GNPS libraries) is the wrong comparison method here
- Candidate structures are missing or unavailable, so OpenBabel fingerprints cannot be computed for candidates
- Only a small pre-defined candidate set is already known and no ranking is needed

## Inputs

- Predicted 528-digit fingerprint (sigmoid output of MetFID ANN, MetFID_CNN_40088_5618.h5)
- Candidate compound InChIKey list (inchikey_list.txt blocks separated by #digit)
- Compound structures for candidates, needed to compute MACCS/FP3/FP4 fingerprints via OpenBabel
- Precursor m/z and ionization mode (e.g. 389.1626 positive) when candidates must be retrieved by m/z via MetaboQuest

## Outputs

- Ranked candidate list (InChIKey and similarity score, descending δ)
- testing_compound_prediction.txt with per-block tables of ranked InChIKeys and scores
- Top-ranked candidate chosen as the annotation (e.g. top-1 hit or 4-way tied top score in the HCC 12-candidate case)

## How to apply

After predicting the 528-bit fingerprint (MACCS + FP3 + FP4) with the pretrained MetFID_CNN_40088_5618.h5 model, compute the same fingerprint for every candidate compound using OpenBabel via PyFingerprint. Apply the similarity function δ(f′,f) = 1 − Σ|f′−f|/n, where f′ is the predicted fingerprint and f the candidate's fingerprint, so a candidate matching every bit scores 1.0 and a bit-flipped candidate scores lower. Sort candidates in descending δ order and assign the highest-ranked candidate as the annotation; record per-block ranked tables (InChIKey and score) in testing_compound_prediction.txt. When retrieving candidates for an unannotated m/z, query MetaboQuest with a 10 PPM tolerance and H+ adduct assumption, or query PubChem by molecular formula via PubChemPy.

## Related tools

- **OpenBabel** (Computes the 528-digit MACCS, FP3, and FP4 molecular fingerprints for candidate compounds)
- **PyFingerprint** (Python wrapper used to call OpenBabel fingerprints (fp3, fp4, maccs) for each candidate compound) — https://github.com/hcji/PyFingerprint
- **MetaboQuest** (Retrieves candidate compound lists by precursor m/z search with 10 PPM tolerance and H+ adduct assumption) — https://omicscraft.com/MetaboQuest
- **PubChemPy** (Queries the PubChem database by molecular formula to obtain candidate compounds when formula-based retrieval is needed)
- **MetFID** (Source of the predicted fingerprint that is scored against candidate fingerprints) — https://github.com/aohongyu/MetFID

## Examples

```
python3 main.py  # ranks candidates via δ = 1 − Σ|f′−f|/n and writes testing_compound_prediction.txt to _files/
```

## Evaluation signals

- Output file testing_compound_prediction.txt exists with per-block ranked tables of InChIKey and score columns in descending order
- Top-ranked candidate matches the reported correct annotation (e.g. the two HCC analytes with 12 and 6 candidates)
- For the m/z = 758.5697 analyte with 12 candidates, four candidates tie at the highest score, reflecting high structural similarity
- Candidate count per block matches the counterpart block in inchikey_list.txt (counter mismatch raises an error)
- Recomputed top-1 accuracy on structure-disjoint testing compounds reproduces the reported ~50% rate with MetaboQuest candidates

## Limitations

- Structurally similar candidates cannot be distinguished without additional ranking criteria — four of 12 candidates tied at the highest score for one HCC analyte
- MetFID correctly ranks the true metabolite first for only about 50% of testing cases; accuracy drops on unseen compound classes (structure-disjoint evaluation)
- Candidate retrieval depends on MetaboQuest coverage; candidates outside the database cannot be scored
- Ranking accuracy is sensitive to the quality of the predicted fingerprint; topology-fingerprint predictions show lower accuracy and F1 than the MACCS/FP3/FP4 baseline

## Evidence

- [other] Score each candidate with the similarity function delta(f',f)=1-sum|f'-f|/n and rank the candidate list in descending score order, assigning the top-ranked candidate as the annotation.: [evidence span withheld — anchor status: not_found]
