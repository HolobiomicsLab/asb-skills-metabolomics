---
name: tanimoto-similarity-structure-evaluation
description: Use when evaluating de novo molecular structure elucidation from mass
  spectra (e.g., MassSpecGym/NPLIB1 predictions) by comparing predicted vs. reference
  structures via fingerprint overlap, or when supplying similarity-based chemical
  feedback to guide generation.
when_to_use_negative:
- The input is an experimental mass spectrum rather than a decoded molecular structure
  — Tanimoto is computed structure-to-structure; spectra must first be converted to
  predicted structures (e.g., via MS-BART or MIST-predicted fingerprints).
- You need exact-match identity assessment — Tanimoto measures substructure overlap,
  not identity; use full InChIKey exact-match accuracy or MCES distance for identity/structural-distance
  claims.
- Molecules are outside the chemical space RDKit can parse (invalid SELFIES/SMILES
  decoding) — scores cannot be computed.
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3473
tools:
- name: RDKit
  role: Computes Morgan fingerprints (radius 2, 2048 bits) from molecular structures
    and supplies the Tanimoto similarity used both for evaluation and as the chemical-feedback
    rank-loss signal
- name: MS-BART
  role: Generates candidate molecular structures whose Top-1/Top-10 Tanimoto similarity
    to references is evaluated; the similarity also drives its contrastive alignment
    step
  repo: https://github.com/OpenDFM/MS-BART
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:de0b6a8f58f24271c9c7c955e47b7ca90121ac4a5d1019d1a73a4fd57d0ddabb
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenDFM/MS-BART
  tools:
  - BART-Base
  - MS-BART
  - RDKit
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# tanimoto-similarity-structure-evaluation

## Summary

Computes Tanimoto similarity between predicted and ground-truth molecular structures using Morgan fingerprints (radius 2, 2048 bits via RDKit) to quantify structural closeness of generated candidates, used both as a headline evaluation metric and as chemical-feedback reward signal in structure elucidation pipelines.

## When to use

Use when evaluating de novo molecular structure elucidation from mass spectra (e.g., MassSpecGym/NPLIB1 predictions) by comparing predicted vs. reference structures via fingerprint overlap, or when supplying similarity-based chemical feedback to guide generation.

## When NOT to use

- The input is an experimental mass spectrum rather than a decoded molecular structure — Tanimoto is computed structure-to-structure; spectra must first be converted to predicted structures (e.g., via MS-BART or MIST-predicted fingerprints).
- You need exact-match identity assessment — Tanimoto measures substructure overlap, not identity; use full InChIKey exact-match accuracy or MCES distance for identity/structural-distance claims.
- Molecules are outside the chemical space RDKit can parse (invalid SELFIES/SMILES decoding) — scores cannot be computed.

## Inputs

- Predicted molecular structures (SMILES/SELFIES) from a structure elucidation model
- Ground-truth reference molecular structures for the test fold
- Ranked candidate lists for Top-1 and Top-10 evaluation

## Outputs

- Per-candidate Tanimoto similarity scores between predicted and reference Morgan fingerprints (radius 2, 2048 bits)
- Aggregate Top-1 and Top-10 maximum Tanimoto similarity metrics for benchmark comparison tables

## How to apply

For each predicted structure and its ground truth, compute Morgan circular fingerprints in RDKit with radius 2 and 2048 bits (the paper's evaluation setting; note that spectra-side fingerprint tokenization uses 4096 bits), then calculate the Tanimoto coefficient between the binary bit vectors. Report Top-1 and Top-10 maximum Tanimoto similarity across ranked candidates alongside exact-match InChIKey accuracy and minimum MCES distance. The same similarity also serves as chemical feedback during alignment: a Tanimoto-similarity rank loss assigns higher generation probabilities to structures closer to the reference. Tanimoto similarity is chosen because fingerprints are invariant to experimental conditions and can be computed reliably from structures with cheminformatics toolkits such as RDKit.

## Related tools

- **RDKit** (Computes Morgan fingerprints (radius 2, 2048 bits) from molecular structures and supplies the Tanimoto similarity used both for evaluation and as the chemical-feedback rank-loss signal)
- **MS-BART** (Generates candidate molecular structures whose Top-1/Top-10 Tanimoto similarity to references is evaluated; the similarity also drives its contrastive alignment step) — https://github.com/OpenDFM/MS-BART

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; fp1=AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(smi_pred),2,nBits=2048); tanimoto=DataStructs.TanimotoSimilarity(fp1,fp_ref)
```

## Evaluation signals

- Tanimoto values fall in [0, 1]; on MassSpecGym MS-Bart's reported Top-1 Tanimoto is approximately 0.23–0.44 range depending on benchmark and epsilon setting (e.g., ~0.23 across tested epsilon values with epsilon=0.11 giving Top-1 accuracy 1.07%).
- Identical structures must yield Tanimoto = 1.0; a self-comparison below 1.0 indicates a fingerprinting bug.
- RDKit fingerprint parameters must match the evaluation protocol: radius 2, 2048 bits — mismatched radius/bit count invalidates comparability with published tables.
- Top-10 maximum Tanimoto should be ≥ Top-1 Tanimoto for every example, since the maximum over a larger candidate set cannot decrease.
- When used as chemical-feedback reward, the rank loss should monotonically favor candidates with higher Tanimoto to the ground truth.

## Limitations

- Tanimoto on Morgan fingerprints rewards substructure overlap but not exact identity; the paper therefore reports it alongside full InChIKey exact-match accuracy and minimum MCES distance.
- Tanimoto similarity as chemical feedback mitigates but does not eliminate molecular hallucinations; alignment assigns higher probabilities to more-similar structures without guaranteeing identity.
- Accuracy of MIST-predicted fingerprints (the spectra-side representation compared during training) significantly impacts overall pipeline performance, indirectly affecting what similarity scores are achievable.

## Evidence

- [methods] Morgan fingerprints radius 2, 2048 bits, RDKit: [evidence span withheld — anchor status: not_found]
- [methods] 4096-bit Morgan fingerprint representation of spectra: [evidence span withheld — anchor status: not_found]
