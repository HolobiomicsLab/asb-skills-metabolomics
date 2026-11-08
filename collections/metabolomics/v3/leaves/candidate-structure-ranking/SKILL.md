---
name: candidate-structure-ranking
description: Use when a predicted Morgan fingerprint bit string from an upstream DeepSAT
  1H-13C HSQC prediction must be scored against fingerprints of candidate structures
  (e.g., from PubChem) to produce a ranked, thresholded annotation list.
when_to_use_negative:
- When the query already has an authentic reference HSQC spectrum to match directly
  against a spectral library (the paper's baseline chemical-shift search with 0.5
  ppm carbon / 0.05 ppm proton thresholds) rather than fingerprint-based ranking
- When structures must be declared dissimilar below the paper's cutoff — cosine scores
  ≤ 0.8 fall outside the 'similar' label and this skill cannot annotate them
- When the input fingerprint was not produced by DeepSAT from an HSQC (or Multiplicity
  Edited HSQC) spectrum, since the 0.8/1.0 thresholds are calibrated to that model's
  reported performance
tools:
- name: DeepSAT
  role: Produces the predicted Morgan fingerprint bit string from the 1H-13C HSQC
    spectrum; its /api/smart3/search endpoint accepts 1H,13C peak lists as JSON dicts
  repo: github.com/mwang87/DeepSAT
- name: RDKit
  role: Generates Morgan fingerprints from candidate molecular structures for cosine-similarity
    comparison against the predicted fingerprint
- name: PubChem
  role: Source of candidate molecular structures whose fingerprints are generated
    and ranked against the predicted fingerprint
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:71c73207de86056a2e3414a2ad9919ea9a9327e63eb695bcbcd600df09b706e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  - PubChem
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# candidate-structure-ranking

## Summary

Rank and label candidate molecular structures against a DeepSAT-predicted Morgan fingerprint by computing cosine similarity scores, applying the paper's thresholds (cosine = 1.0 → identical/identified, cosine > 0.8 → similar/annotated).

## When to use

Use when a predicted Morgan fingerprint bit string from an upstream DeepSAT 1H-13C HSQC prediction must be scored against fingerprints of candidate structures (e.g., from PubChem) to produce a ranked, thresholded annotation list.

## When NOT to use

- When the query already has an authentic reference HSQC spectrum to match directly against a spectral library (the paper's baseline chemical-shift search with 0.5 ppm carbon / 0.05 ppm proton thresholds) rather than fingerprint-based ranking
- When structures must be declared dissimilar below the paper's cutoff — cosine scores ≤ 0.8 fall outside the 'similar' label and this skill cannot annotate them
- When the input fingerprint was not produced by DeepSAT from an HSQC (or Multiplicity Edited HSQC) spectrum, since the 0.8/1.0 thresholds are calibrated to that model's reported performance

## Inputs

- Predicted Morgan fingerprint bit string from the upstream DeepSAT HSQC prediction step
- Candidate molecular structures (e.g., SMILES/structures from PubChem or an in-house HSQC reference library)
- Optional: query 1H-13C peak list as JSON list of dicts with 1H,13C headers for the DeepSAT search API

## Outputs

- Ranked candidate list with per-candidate cosine similarity scores in [0, 1]
- Annotation labels per candidate (identical = cosine 1.0; similar = cosine > 0.8), e.g. structure_annotations.csv

## How to apply

Load the predicted Morgan fingerprint bit string produced by the DeepSAT HSQC prediction step, and generate Morgan fingerprints for each candidate structure (retrieved from chemical databases such as PubChem) using RDKit. Compute the cosine similarity between the predicted fingerprint and every candidate fingerprint, then sort candidates by descending score. Apply the documented thresholds: a cosine score of 1.0 marks a candidate as identical (identification), and a cosine score over 0.8 marks it as similar (annotation), matching the paper's annotation-rate cutoff and consistent with the reported average predicted-vs-ground-truth cosine score of 0.8450 (normal HSQC) / 0.8574 (Multiplicity Edited HSQC). Emit the ranked candidate list with scores and identical/similar labels (e.g., to structure_annotations.csv) and validate that every score lies in [0, 1].

## Related tools

- **DeepSAT** (Produces the predicted Morgan fingerprint bit string from the 1H-13C HSQC spectrum; its /api/smart3/search endpoint accepts 1H,13C peak lists as JSON dicts) — github.com/mwang87/DeepSAT
- **RDKit** (Generates Morgan fingerprints from candidate molecular structures for cosine-similarity comparison against the predicted fingerprint)
- **PubChem** (Source of candidate molecular structures whose fingerprints are generated and ranked against the predicted fingerprint)

## Examples

```
curl -d '[{"1H":1.25,"13C":45.0}]' https://deepsat.ucsd.edu/api/smart3/search
```

## Evaluation signals

- Every reported cosine score lies within [0, 1]
- The 0.8 cosine threshold is applied as the annotation cutoff, and 1.0 as the identification cutoff, as documented in the paper
- Predicted-vs-ground-truth fingerprint cosine scores fall near the reported baselines (0.8450 normal HSQC, 0.8574 Multiplicity Edited HSQC)
- Ranked output is sorted by descending cosine score and each candidate carries an identical/similar label
- Candidate fingerprints are Morgan fingerprints generated from known small-molecule structures (not HSQC-shift matches)

## Limitations

- Interpretability/decision rationale is limited: 'these results do not fully explain the decision-making process of DeepSAT' (occlusion analysis shown for a single compound, quercetin)
- Solvent-transfer robustness is incomplete — over 65% of results were still correctly identified across different solvents, but not all
- Model behavior can change if TensorFlow Serving model input names change ('If the model input names change, then we need to change it in the code')
- Absolute precision/recall/F1 values for comparison against SMART 2.0 are deferred to supplementary tables/figures, not in the main text

## Evidence

- [results] threshold-similarity definition: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
- [results] annotation cutoff: "For the measurement of annotation rate, cosine score of 0.8 was set as the threshold"
- [results] expected cosine baselines: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [discussion] candidate fingerprint generation and comparison: "these fingerprints can also be easily generated from all known small molecules to which DeepSAT results can be compared for similarity"
- [readme] API input format for query peaks: "You can put in your peaks as a json list of dicts, with 1H,13C as headers."
