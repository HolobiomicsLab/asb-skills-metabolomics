---
name: molecular-fingerprint-tanimoto-similarity-scoring
description: Use when comparing a predicted molecular structure (e.g. from de novo
  MS2 spectrum prediction) against a known reference structure and you need a graded
  similarity score or thresholded match counts.
when_to_use_negative:
- When spectra were recorded in negative ion mode or outside positive ion mode (H+)
  conditions the model was trained on — similarity scoring will not compensate for
  out-of-scope predictions.
- When the goal is spectral library matching (cosine scoring on raw spectra) rather
  than structure-level comparison — fingerprint Tanimoto operates on structures, not
  peak lists.
- When comparing high-resolution-spectra-optimized models (e.g. MassGenie, MSNovelist
  outputs) where fingerprint similarity thresholds were not the validated evaluation
  basis.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: Transformer enabled Fragment Tree (TeFT)
  role: Produces the predicted candidate molecular structures whose SMILES are fingerprint-compared
    against reference structures
  repo: https://github.com/thumingo/TeFT
- name: SIRIUS4
  role: Baseline structure-prediction method whose outputs are scored with the same
    Tanimoto thresholds for comparison (27.6% exact, 39% >0.9)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:24ea84fb39a7daf617a44c3eab02bb4a735baf1da3b43fc5747f5795a1a32088
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - SIRIUS
  - SIRIUS4
  - Transformer
  - Transformer enabled Fragment Tree (TeFT)
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# molecular-fingerprint-tanimoto-similarity-scoring

## Summary

Evaluate predicted-vs-reference molecular structure agreement by converting both structures to SMILES, computing MACCS-key and Morgan fingerprint Tanimoto similarities, and counting exact (1.0) and thresholded (>0.9, >0.8) matches. This is the metric used in the TeFT paper to benchmark structure generation against SIRIUS4 on 660 MS2 test spectra.

## When to use

Use when comparing a predicted molecular structure (e.g. from de novo MS2 spectrum prediction) against a known reference structure and you need a graded similarity score or thresholded match counts.

## When NOT to use

- When spectra were recorded in negative ion mode or outside positive ion mode (H+) conditions the model was trained on — similarity scoring will not compensate for out-of-scope predictions.
- When the goal is spectral library matching (cosine scoring on raw spectra) rather than structure-level comparison — fingerprint Tanimoto operates on structures, not peak lists.
- When comparing high-resolution-spectra-optimized models (e.g. MassGenie, MSNovelist outputs) where fingerprint similarity thresholds were not the validated evaluation basis.

## Inputs

- Predicted molecular structure (SMILES or convertible structure object) from a structure-generation model
- Reference/actual molecular structure (SMILES or convertible structure object)
- Set of MS2 spectra paired with known reference structures (e.g. 660-spectrum held-out test set from GNPS, HMDB 5.0, MoNA)

## Outputs

- Per-pair MACCS-key and Morgan fingerprint Tanimoto similarity scores
- Counts and percentages of exact matches (Tanimoto = 1) and matches above 0.9 and 0.8 thresholds
- Aggregate metrics table (e.g. results_table.csv)

## How to apply

Convert both the predicted candidate structure and the real (reference) structure to canonical SMILES, then compute MACCS-key and Morgan fingerprint Tanimoto similarity scores between them. Count spectra/structures whose best candidate equals the real structure (similarity of 1) and those exceeding 0.9 and 0.8 similarity thresholds, then compute percentages over the total number of spectra (in the TeFT benchmark: 195/660 = 30% exact, 311/660 = 47% >0.9, 439/660 = 67% >0.8). Aggregate the counts into a results table (results_table.csv) and, where a benchmark figure exists, validate reproduction against the reported counts. This fingerprint-based comparison is the standard way to assign partial credit when exact structure identification fails, since de novo generators may return near-isomers rather than exact matches.

## Related tools

- **Transformer enabled Fragment Tree (TeFT)** (Produces the predicted candidate molecular structures whose SMILES are fingerprint-compared against reference structures) — https://github.com/thumingo/TeFT
- **SIRIUS4** (Baseline structure-prediction method whose outputs are scored with the same Tanimoto thresholds for comparison (27.6% exact, 39% >0.9))

## Examples

```
python evaluate_tanimoto.py --pred smiles_pred.csv --ref smiles_ref.csv --fingerprints maccs,morgan --thresholds 0.8,0.9 --out results_table.csv
```

## Evaluation signals

- Reproduced counts on the 660-spectrum test set match the reported figures: 195 exact matches, 311 above 0.9 similarity, 439 above 0.8 similarity.
- Computed percentages equal 30% (195/660), 47% (311/660), and 67% (439/660) respectively.
- Both predicted and reference structures convert to valid canonical SMILES before fingerprint computation (no conversion failures silently dropped from the count).
- Fingerprints computed with both MACCS-key and Morgan types, as in the paper, not a single fingerprint family.
- Aggregate metrics written to results_table.csv with counts traceable to individual spectrum-level scores.

## Limitations

- Tanimoto similarity is a partial-credit proxy — a >0.9 score does not guarantee correct stereochemistry or exact isomer identity.
- Reference benchmark performance (30% exact on 660 spectra) applies to positive-ion-mode, low-resolution MS2 spectra compiled from GNPS, HMDB 5.0, and MoNA; results may not transfer to other acquisition modes.
- Fingerprint choice (MACCS vs Morgan) affects scores; using only one may shift threshold crossing counts.

## Evidence

- [other] Convert both the predicted and the real structures to SMILES and compute their MACCS-key and Morgan fingerprint Tanimoto similarity scores.: [evidence span withheld — anchor status: not_found]
- [results] On the 660-spectrum test set, the TeFT Transformer model correctly identified 30% (195/660) of actual structures with Tanimoto similarity of 1: [evidence span withheld — anchor status: not_found]
- [other] Load the 660 non-repetitive MS2 test spectra compiled from the open-source databases GNPS, HMDB (5.0), and MoNA.: [evidence span withheld — anchor status: not_found]
- [discussion] we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability: "we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability"
