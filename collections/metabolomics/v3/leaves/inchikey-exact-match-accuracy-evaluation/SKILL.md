---
name: inchikey-exact-match-accuracy-evaluation
description: Use when benchmarking de novo structure elucidation from mass spectra
  on MassSpecGym/NPLIB1 test folds and you need Top-1/Top-10 exact-identity accuracy
  over full InChIKey strings for generated molecule sets.
when_to_use_negative:
- The analysis goal is structural similarity or substructure recovery rather than
  exact identity — use Top-k minimum MCES distance and maximum Tanimoto similarity
  (Morgan radius 2, 2048 bits) instead
- Candidates are intermediate representations (MIST-predicted 4096-bit Morgan fingerprints)
  rather than decodable molecular structures, so no InChIKey can be computed
- The evaluation dataset is restricted-access (e.g., NIST23, unavailable due to commercial
  licensing) and has no comparable ground-truth structure annotations
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3474
tools:
- name: RDKit
  role: Decodes predicted SELFIES into structures, canonicalizes them, and computes
    full InChIKey strings plus Morgan fingerprints (radius 2, 2048 bits) for the companion
    Tanimoto metric
- name: SELFIES
  role: String-based molecular representation in which MS-Bart generates candidate
    molecules before conversion to structures for InChIKey comparison
- name: MS-BART
  role: Generator whose beam-search multinomial sampled SELFIES candidates are scored
    by Top-1/Top-10 InChIKey accuracy on MassSpecGym and NPLIB1
  repo: github.com/OpenDFM/MS-BART
- name: MIST
  role: Predicts the fingerprints from spectra that condition candidate generation,
    indirectly bounding the achievable exact-match accuracy (gold-fingerprint upper
    bounds 73.50% and 47.56%)
  repo: github.com/coleygroup/FRIGID
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_content_hash: sha256:60c3c27fca0ffff4f0f4c97df8e48420166158e619c97d8aa25f277f366a0211
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BART-Base
  - MIST
  - MS-BART
  - MassSpecGym
  - NPLIB1
  - RDKit
  - SELFIES
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# InChIKey exact-match accuracy evaluation

## Summary

Computes Top-1/Top-10 identification accuracy by comparing generated candidate molecular structures (SELFIES decoded via RDKit) against ground-truth structures through full InChIKey exact match, the strictest identity metric used to benchmark mass-spectra-to-molecule models on MassSpecGym and NPLIB1.

## When to use

Use when benchmarking de novo structure elucidation from mass spectra on MassSpecGym/NPLIB1 test folds and you need Top-1/Top-10 exact-identity accuracy over full InChIKey strings for generated molecule sets.

## When NOT to use

- The analysis goal is structural similarity or substructure recovery rather than exact identity — use Top-k minimum MCES distance and maximum Tanimoto similarity (Morgan radius 2, 2048 bits) instead
- Candidates are intermediate representations (MIST-predicted 4096-bit Morgan fingerprints) rather than decodable molecular structures, so no InChIKey can be computed
- The evaluation dataset is restricted-access (e.g., NIST23, unavailable due to commercial licensing) and has no comparable ground-truth structure annotations

## Inputs

- Predicted candidate SELFIES/SMILES molecules per spectrum (e.g., 100 beam-search multinomial samples from MS-Bart or DiffMS, re-ranked by formula distance with log-probability tie-breaking)
- Ground-truth molecular structures/InChIKeys for the complete MassSpecGym and NPLIB1 test folds
- Re-ranked top-k prediction lists per spectrum (top-1 and top-10)

## Outputs

- Top-1 and Top-10 exact-match accuracy percentages per model per benchmark
- Table-1-style metric rows combining accuracy with MCES and Tanimoto columns (results_table.csv; table8_temperature_sensitivity.csv for the temperature sweep)

## How to apply

For each spectrum in the MassSpecGym or NPLIB1 test fold, take the beam-search multinomial sampled candidate SELFIES molecules (e.g., 100 candidates per spectrum from the fully optimized pretrain-FT-rank model), re-rank them by formula distance (sum of absolute atom-count differences, Eq. 6) with estimated log-probability tie-breaking, and select the top-k predictions. Decode each predicted SELFIES to a structure with RDKit, canonicalize to a full InChIKey, and mark a hit when the candidate InChIKey exactly equals the ground-truth InChIKey; Top-1 uses the rank-1 candidate and Top-10 scans the top ten. Report accuracy as a percentage over the complete test fold, always alongside the looser Top-k minimum MCES distance and Top-k maximum Tanimoto similarity (Morgan fingerprints, radius 2, 2048 bits) since exact-match misses near-correct structures. Validate against published anchors: MS-Bart Top-1 accuracy 7.45% on NPLIB1 and 1.07% on MassSpecGym (gold-fingerprint upper bounds 73.50% and 47.56%), and FRIGID 14.16% on NPLIB1 / 9.05% on MassSpecGym with the revised MIST encoders.

## Related tools

- **RDKit** (Decodes predicted SELFIES into structures, canonicalizes them, and computes full InChIKey strings plus Morgan fingerprints (radius 2, 2048 bits) for the companion Tanimoto metric)
- **SELFIES** (String-based molecular representation in which MS-Bart generates candidate molecules before conversion to structures for InChIKey comparison)
- **MS-BART** (Generator whose beam-search multinomial sampled SELFIES candidates are scored by Top-1/Top-10 InChIKey accuracy on MassSpecGym and NPLIB1) — github.com/OpenDFM/MS-BART
- **MIST** (Predicts the fingerprints from spectra that condition candidate generation, indirectly bounding the achievable exact-match accuracy (gold-fingerprint upper bounds 73.50% and 47.56%)) — github.com/coleygroup/FRIGID

## Examples

```
import selfies as sf; from rdkit import Chem; ik = Chem.MolToInChIKey(Chem.MolFromSmiles(sf.decoder(pred_selfies)))
```

## Evaluation signals

- Reproduced Top-1 accuracy matches published anchors: 7.45% (NPLIB1) and 1.07% (MassSpecGym) for MS-Bart; 14.16% / 9.05% for FRIGID with revised MIST encoders
- Top-10 accuracy is strictly >= Top-1 accuracy for every model and fold (e.g., MS-Bart NPLIB1: 7.45% vs 10.99%)
- Accuracy stays well below the gold-fingerprint upper bounds (73.50% NPLIB1, 47.56% MassSpecGym) since predicted fingerprints limit performance
- Metrics remain essentially constant across decoding temperatures 0.2/0.4/0.8 (Table 8 stability criterion)
- Computed over the complete test fold, with known MSG eval script issues (test-path filename, NUM_BEAMS/NUM_BEAM variable mismatch) corrected

## Limitations

- Full InChIKey exact match is an all-or-nothing identity test: structurally close candidates score zero, which is why MCES and Tanimoto are always reported alongside
- Accuracy ceiling depends on MIST fingerprint-prediction accuracy; 'The accuracy of these fingerprint predictions significantly impacts the overall performance.'
- Formula information is used only for re-ranking, not for training, so exact-match accuracy cannot be improved by conditioning on formula during generation
- Comparisons are limited to MassSpecGym and NPLIB1; MADGEN's NIST23 results cannot be reproduced due to commercial licensing restrictions

## Evidence

- [methods] Benchmarks evaluated with these metrics: [evidence span withheld — anchor status: not_found]
- [readme] FRIGID Top-1 accuracy anchors from README results: [evidence span withheld — anchor status: external_doc]
- [readme] MS-BART baseline row in the FRIGID README comparison table: [evidence span withheld — anchor status: external_doc]
