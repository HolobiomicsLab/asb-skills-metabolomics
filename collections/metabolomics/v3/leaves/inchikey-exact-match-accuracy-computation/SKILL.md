---
name: inchikey-exact-match-accuracy-computation
description: Use when evaluating de novo structure elucidation output (SELFIES/SMILES
  candidates from a spectrum-conditioned model) and you need the fraction of predictions
  whose full InChIKey string exactly equals the reference molecule's InChIKey.
when_to_use_negative:
- The evaluation target is structural similarity rather than identity — use Tanimoto
  similarity (Morgan, radius 2, 2048 bits) or MCES distance instead, since exact InChIKey
  match gives 0 credit for near-miss analogues.
- Only the carbon skeleton (first InChIKey block) matters — the paper's metric explicitly
  uses the full InChIKey, so partial-block matching is a different, more lenient evaluation.
- The benchmark is NIST23 — access is restricted due to commercial licensing, so this
  metric as reproduced here applies to the open MassSpecGym and NPLIB1 benchmarks.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: RDKit
  role: Converts predicted and reference molecular structures to InChIKey strings
    and computes Morgan fingerprints (radius 2, 2048 bits) for the companion Tanimoto
    metric
- name: MS-BART
  role: Source of the evaluation pipeline (scripts/msg/eval.sh) whose beam-search
    multinomial sampling produces the top-10 candidate lists scored by exact-match
    accuracy
  repo: github.com/OpenDFM/MS-BART
- name: FRIGID
  role: Baseline framework benchmarked with the same Top-1/Top-10 InChIKey accuracy
    metric on NPLIB1 (14.16%) and MassSpecGym (9.05%)
  repo: github.com/coleygroup/FRIGID
- name: MolGen
  role: Lineage of the chemical-feedback alignment used in MS-Bart whose downstream
    generations are scored with this metric
  repo: github.com/zjunlp/MolGen
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:76073937da3932b6d552013d2ae93d150082e586aecc6ce94e8c48d03d9e979c
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BART-Base
  - FRIGID
  - MS-BART
  - MolGen
  - RDKit
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# InChIKey exact-match accuracy computation

## Summary

Computes Top-1/Top-10 exact-match accuracy on the full InChIKey string, comparing generated candidate structures against ground-truth reference molecules as used to benchmark MS-Bart and competing methods on MassSpecGym and NPLIB1.

## When to use

Use when evaluating de novo structure elucidation output (SELFIES/SMILES candidates from a spectrum-conditioned model) and you need the fraction of predictions whose full InChIKey string exactly equals the reference molecule's InChIKey.

## When NOT to use

- The evaluation target is structural similarity rather than identity — use Tanimoto similarity (Morgan, radius 2, 2048 bits) or MCES distance instead, since exact InChIKey match gives 0 credit for near-miss analogues.
- Only the carbon skeleton (first InChIKey block) matters — the paper's metric explicitly uses the full InChIKey, so partial-block matching is a different, more lenient evaluation.
- The benchmark is NIST23 — access is restricted due to commercial licensing, so this metric as reproduced here applies to the open MassSpecGym and NPLIB1 benchmarks.

## Inputs

- Ranked list of predicted molecular structures per spectrum (SELFIES/SMILES strings, top-10 from beam-search multinomial sampling)
- Ground-truth reference structure per test spectrum (MassSpecGym or NPLIB1 test fold, with reference InChIKey or SMILES)

## Outputs

- Top-1 exact-match accuracy (%) on full InChIKey
- Top-10 exact-match accuracy (%) on full InChIKey
- Companion metrics: Top-1/Top-10 minimum MCES distance and maximum Tanimoto similarity

## How to apply

Convert each generated candidate (SELFIES or SMILES, as produced by MS-Bart's decoder) to a molecule and derive its InChIKey using RDKit; do the same for the ground-truth reference structure. An exact match is counted only when the full 27-character InChIKey (not just the first skeleton block) matches. Report Top-1 accuracy over the ranked candidate list and Top-10 accuracy over the top-10 beam-search candidates, on the full MassSpecGym test fold (all adducts retained) and on NPLIB1. Always report the metric alongside minimum MCES distance and maximum Tanimoto similarity (Morgan fingerprints, radius 2, 2048 bits, RDKit), since exact-match accuracy is a strict identity metric and near-misses must be characterized by the softer measures. Compare against published baseline rows (e.g., MADGEN, DiffMS, MIST+MolForge, MS-BART) using the same known-formula setting.

## Related tools

- **RDKit** (Converts predicted and reference molecular structures to InChIKey strings and computes Morgan fingerprints (radius 2, 2048 bits) for the companion Tanimoto metric)
- **MS-BART** (Source of the evaluation pipeline (scripts/msg/eval.sh) whose beam-search multinomial sampling produces the top-10 candidate lists scored by exact-match accuracy) — github.com/OpenDFM/MS-BART
- **FRIGID** (Baseline framework benchmarked with the same Top-1/Top-10 InChIKey accuracy metric on NPLIB1 (14.16%) and MassSpecGym (9.05%)) — github.com/coleygroup/FRIGID
- **MolGen** (Lineage of the chemical-feedback alignment used in MS-Bart whose downstream generations are scored with this metric) — github.com/zjunlp/MolGen

## Examples

```
from rdkit import Chem; ik = Chem.MolToInchiKey(Chem.MolFromSmiles(pred_smiles)); exact_top1 = sum(ik == ref_ik for ik, ref_ik in zip(top1_iks, ref_iks)) / len(ref_iks)
```

## Evaluation signals

- Match decision uses the full InChIKey string identity — verify no truncation to the skeleton block before comparison.
- Top-10 accuracy must be >= Top-1 accuracy for the same model on the same fold.
- Top-1 values on these benchmarks should fall in the published range (e.g., 9.05% MassSpecGym / 14.16% NPLIB1 for FRIGID with revised MIST encoders; ~1.07% for the epsilon=0.11 MS-Bart fingerprint-threshold variant).
- Evaluation must run on the full MassSpecGym test fold with all adducts retained, even though fine-tuning/alignment used only [M+H]+ data.
- Exact-match accuracy should be reported together with Top-1 MCES (~16.47 for the MS-Bart threshold variant) and Tanimoto (~0.23) to contextualize near-misses.

## Limitations

- The accuracy of MIST fingerprint predictions significantly impacts downstream generation quality, and thus the exact-match accuracy achievable from spectra.
- Formula information is used only for re-ranking and not for training the model, which caps identification performance in this evaluation setting.
- Exact-match accuracy is extremely strict: methods with strong similarity scores (e.g., Top-1 Tanimoto ~0.23) can still yield near-zero exact-match rates, so the metric alone understates partial correctness.

## Evidence

- [methods] Metrics computed in the evaluation: [evidence span withheld — anchor status: not_found]
- [methods] Benchmarks on which the metric is reported: [evidence span withheld — anchor status: not_found]
- [readme] Published baseline magnitudes for calibration: [evidence span withheld — anchor status: external_doc]
