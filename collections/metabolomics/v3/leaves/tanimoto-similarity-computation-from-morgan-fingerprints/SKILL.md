---
name: tanimoto-similarity-computation-from-morgan-fingerprints
description: Use when evaluating de novo molecular structure predictions from mass
  spectra (e.g., SELFIES outputs from MS-Bart) and you need a graded structural-similarity
  metric alongside exact-match accuracy and MCES distance.
when_to_use_negative:
- When exact structural identity is required — use full InChIKey comparison for Top-k
  exact-match accuracy instead, since Tanimoto similarity rewards near-misses
- When comparing spectra or fragments rather than whole molecules — the metric is
  defined over molecular fingerprints, not spectral similarity (use cosine/Greedy
  cosine for spectra)
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: RDKit
  role: Computes Morgan fingerprints (radius 2, 2048 bits) from molecular structures
    and converts predicted SELFIES to structures for Tanimoto similarity scoring
- name: MS-Bart
  role: Generates the candidate SELFIES molecules whose top-k predictions are scored
    by Top-k maximum Tanimoto similarity on MassSpecGym/NPLIB1
  repo: https://github.com/OpenDFM/MS-BART
- name: MIST
  role: Predicts 4096-bit circular Morgan fingerprints from mass spectra as an intermediate
    representation, whose binarized bits condition candidate generation
  repo: https://github.com/coleygroup/FRIGID
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:6e8d6295d79f2b12b6e3783ee3f807f64353bc552c0cc9bd7e04f62c30b7230d
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenDFM/MS-BART
  tools:
  - MIST
  - MS-Bart
  - RDKit
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# tanimoto-similarity-computation-from-morgan-fingerprints

## Summary

Computes Top-k maximum Tanimoto similarity between predicted and reference molecular structures by deriving radius-2, 2048-bit Morgan fingerprints with RDKit; used in MS-Bart evaluation to quantify structural similarity when exact InChIKey matches are too strict.

## When to use

Use when evaluating de novo molecular structure predictions from mass spectra (e.g., SELFIES outputs from MS-Bart) and you need a graded structural-similarity metric alongside exact-match accuracy and MCES distance.

## When NOT to use

- When exact structural identity is required — use full InChIKey comparison for Top-k exact-match accuracy instead, since Tanimoto similarity rewards near-misses
- When comparing spectra or fragments rather than whole molecules — the metric is defined over molecular fingerprints, not spectral similarity (use cosine/Greedy cosine for spectra)

## Inputs

- Predicted SELFIES strings (top-k candidates per spectrum, e.g., 100 sampled candidates re-ranked by formula distance)
- Ground-truth molecular structures (SMILES/SELFIES/InChIKey) from the MassSpecGym or NPLIB1 test split
- RDKit Morgan fingerprint configuration: radius 2, 2048 bits

## Outputs

- Top-1 and Top-10 maximum Tanimoto similarity scores per model/benchmark (e.g., MS-Bart values reported in the MassSpecGym and NPLIB1 comparison tables)
- Per-spectrum similarity values aggregateable into result tables such as table8_temperature_sensitivity.csv

## How to apply

Convert each predicted structure (decoded from SELFIES via RDKit) and each ground-truth structure into a Morgan fingerprint with radius 2 folded to 2048 bits. Compute pairwise Tanimoto similarity between the top-k candidate predictions per spectrum and the reference molecule, and report the maximum Tanimoto similarity within the top-k set (Top-1 and Top-10 variants). Rank candidates first by formula distance (sum of absolute atom-count differences, ties broken by estimated log-probability) before selecting the top-k. When fingerprints must be predicted from spectra rather than computed from structures, MS-Bart's pipeline tokenizes each spectrum as a 4096-bit circular Morgan fingerprint via MIST and applies threshold epsilon (e.g., 0.11 on the MassSpecGym fingerprint-SELFIES table) to binarize bits before decoding.

## Related tools

- **RDKit** (Computes Morgan fingerprints (radius 2, 2048 bits) from molecular structures and converts predicted SELFIES to structures for Tanimoto similarity scoring)
- **MS-Bart** (Generates the candidate SELFIES molecules whose top-k predictions are scored by Top-k maximum Tanimoto similarity on MassSpecGym/NPLIB1) — https://github.com/OpenDFM/MS-BART
- **MIST** (Predicts 4096-bit circular Morgan fingerprints from mass spectra as an intermediate representation, whose binarized bits condition candidate generation) — https://github.com/coleygroup/FRIGID

## Examples

```
from rdkit import Chem; from rdkit.Chem import AllChem, DataStructs; fp_pred = AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(smiles_pred), radius=2, nBits=2048); sim = DataStructs.TanimotoSimilarity(fp_pred, fp_ref)
```

## Evaluation signals

- Top-k maximum Tanimoto similarity values fall in the plausible range reported for comparable models (e.g., MS-Bart Top-1 Tanimoto 0.44 on NPLIB1; random/near-baseline models score ~0.12-0.27)
- Tanimoto column remains essentially constant across decoding temperatures (0.2, 0.4, 0.8) when reproducing Table 8, matching the paper's temperature-robustness finding
- Tanimoto Top-k values should increase monotonically from Top-1 to Top-10 since the maximum is taken over a larger candidate set
- Fingerprint parameters are consistent (radius 2, 2048 bits for structure comparison vs. 4096-bit MIST spectral fingerprints) — mismatched bit widths indicate a pipeline error

## Limitations

- Tanimoto similarity to a fingerprint does not guarantee structural correctness; near-miss structures score high, which is why the paper pairs it with exact InChIKey match and MCES graph distance
- When fingerprints are predicted from spectra by MIST rather than computed from structures, 'The accuracy of these fingerprint predictions significantly impacts the overall performance.'
- The paper reports no software/tool versions for RDKit in its discussion/limitations text, so exact bit-order reproducibility across RDKit versions is not guaranteed

## Evidence

- [readme] Top-k maximum Tanimoto similarity using Morgan fingerprints (radius 2, 2048 bits, RDKit): [evidence span withheld — anchor status: not_found]
- [methods] spectra as Morgan fingerprints via MIST: [evidence span withheld — anchor status: not_found]
- [other] temperature robustness of MS-Bart metrics: [evidence span withheld — anchor status: not_found]
- [readme] benchmark comparison values from FRIGID README: [evidence span withheld — anchor status: external_doc]
