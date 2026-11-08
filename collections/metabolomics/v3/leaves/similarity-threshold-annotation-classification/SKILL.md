---
name: similarity-threshold-annotation-classification
description: Use when a predicted Morgan fingerprint bit string from an HSQC-based
  predictor must be compared against fingerprints of candidate structures from chemical
  databases such as PubChem to produce ranked, thresholded annotation results.
when_to_use_negative:
- The query is a raw 1H/13C chemical shift list to be searched against a spectral
  reference library by ppm matching (use the baseline shift-search workflow with 0.5
  ppm carbon / 0.05 ppm proton thresholds instead)
- Ground-truth fingerprints are unavailable or candidate structures cannot be converted
  to Morgan fingerprints (cosine similarity is undefined in that case)
- The goal is compound class (scaffold) prediction accuracy rather than per-structure
  annotation — DeepSAT's multi-task model already outputs class predictions separately
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3314
tools:
- name: DeepSAT
  role: Produces the predicted Morgan fingerprint bit string from the query HSQC spectrum
    that is compared against candidate fingerprints
  repo: github.com/mwang87/DeepSAT
- name: PubChem
  role: Source of candidate molecular structures whose fingerprints are generated
    for cosine-score comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:99a6c2323743aa69421e0c06835e03190a55248e2fd8cd6f034bdb55dff953d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  - PubChem
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# similarity-threshold-annotation-classification

## Summary

Ranks candidate molecular structures by cosine similarity between DeepSAT-predicted Morgan fingerprints and fingerprints generated from known small molecules, then labels candidates as identical (cosine score 1.0) or similar (cosine score over 0.8). This thresholding step converts fingerprint predictions into concrete structure annotation results.

## When to use

Use when a predicted Morgan fingerprint bit string from an HSQC-based predictor must be compared against fingerprints of candidate structures from chemical databases such as PubChem to produce ranked, thresholded annotation results.

## When NOT to use

- The query is a raw 1H/13C chemical shift list to be searched against a spectral reference library by ppm matching (use the baseline shift-search workflow with 0.5 ppm carbon / 0.05 ppm proton thresholds instead)
- Ground-truth fingerprints are unavailable or candidate structures cannot be converted to Morgan fingerprints (cosine similarity is undefined in that case)
- The goal is compound class (scaffold) prediction accuracy rather than per-structure annotation — DeepSAT's multi-task model already outputs class predictions separately

## Inputs

- Predicted Morgan fingerprint bit string from the upstream DeepSAT HSQC prediction step
- Candidate molecular structures from chemical databases (e.g., PubChem), as SMILES or structure records for RDKit fingerprint generation

## Outputs

- structure_annotations.csv containing ranked candidate structures, cosine similarity scores, and annotation labels (identical/similar)

## How to apply

Load the predicted Morgan fingerprint bit string produced by the upstream DeepSAT prediction step from the input 1H-13C HSQC spectrum. Generate Morgan fingerprints for candidate molecular structures retrieved from chemical databases such as PubChem using RDKit. Compute the cosine similarity between the predicted fingerprint and each candidate fingerprint, rank candidates by descending score, and apply the paper's thresholds: a cosine score of 1.0 marks a candidate as identical (identification), while a score over 0.8 marks it as similar (annotation). Write the ranked candidate list with similarity scores and annotation labels to structure_annotations.csv. The rationale is that fingerprint cosine similarity bypasses the need for authentic reference NMR spectra, enabling direct structure search in large databases; the 0.8 cutoff is the paper's documented annotation-rate threshold and is consistent with its reported average predicted-vs-ground-truth cosine score of 0.8450.

## Related tools

- **DeepSAT** (Produces the predicted Morgan fingerprint bit string from the query HSQC spectrum that is compared against candidate fingerprints) — github.com/mwang87/DeepSAT
- **PubChem** (Source of candidate molecular structures whose fingerprints are generated for cosine-score comparison)

## Examples

```
from scipy.spatial.distance import cosine; score = 1 - cosine(pred_fp, cand_fp); label = 'identical' if score == 1.0 else ('similar' if score > 0.8 else 'none')
```

## Evaluation signals

- Every reported cosine score lies within [0, 1]
- The 0.8 similarity threshold is applied as the annotation cutoff, consistent with the paper's reported average predicted-vs-ground-truth fingerprint cosine score of 0.8450 (normal HSQC model)
- A cosine score of exactly 1.0 is labeled identical (identification); scores over 0.8 but below 1.0 are labeled similar (annotation)
- Candidates are ranked by descending cosine score with no missing scores in the output table
- Boundary check: a candidate with cosine score exactly 0.8 should NOT be labeled similar (threshold is 'over 0.8')

## Limitations

- The 0.8 cutoff treats scores over 0.8 as 'similar', which captures structural resemblance but not identity; only a score of 1.0 counts as identical
- The average predicted-vs-ground-truth cosine score is 0.8450, meaning predicted fingerprints imperfectly reproduce true fingerprints, so borderline candidates near the 0.8 cutoff may be mislabeled
- Multiplicity Edited HSQC input performs slightly better (average cosine 0.8574 vs 0.8450), so results depend on the HSQC variant used upstream

## Evidence

- [results] Structures considered similar if cosine score over 0.8 based on predicted and known Morgan fingerprints: [evidence span withheld — anchor status: not_found]
- [results] Annotation threshold set at cosine 0.8: "For the measurement of annotation rate, cosine score of 0.8 was set as the threshold"
- [results] Average predicted-vs-ground-truth cosine score: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [discussion] Candidate fingerprints generated from known small molecules for comparison: "these fingerprints can also be easily generated from all known small molecules to which DeepSAT results can be compared for similarity"
- [intro] Molecular structure databases such as PubChem contain millions of compounds: "molecular structure databases such as Pubchem [36] contain millions of compounds"
