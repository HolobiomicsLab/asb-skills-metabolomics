---
name: lc-msms-ground-truth-evaluation
description: Use when you have MS/MS spectra of a compound set whose molecular formulas/structures
  were validated by spectral library search and manual inspection, and you need to
  quantify annotation accuracy of a formula- or structure-annotation tool.
when_to_use_negative:
- When the compound set has no independently validated annotations — no ground truth
  exists, so no accuracy can be computed
- When you need structural rather than molecular-formula accuracy and the tool only
  outputs formulas (or vice versa) — the comparison must match annotation level
- When the goal is discovery of novel formulas rather than benchmarking — novel formulas
  by definition lack library-validated ground truth
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SIRIUS
  role: Produces top-ranked molecular formula candidates that are scored against the
    ground truth; also used to establish ground truth via fragmentation tree and isotope
    analysis
  repo: https://github.com/sirius-ms/sirius
- name: ZODIAC
  role: Reranks SIRIUS candidates; its top-1 output is the second prediction being
    scored against the same ground truth to measure improvement
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Structure-level annotation whose top-1 hits can serve as ground-truth anchors
    during evaluation
- name: OpenMS
  role: Preprocesses input mzML/mzXML and discards low-quality MS/MS spectra before
    ground-truth evaluation
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:e7fe5219cbd139b6e1c3e231947ded4c3f2291a5280aa9d718e444e76b4a17a0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSI:FingerID
  - OpenMS
  - SIRIUS
  - ZODIAC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# lc-msms-ground-truth-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates computational annotation tools (SIRIUS, ZODIAC, CSI:FingerID) against a hand-curated ground truth built from spectral library search plus manual validation, computing exact top-1 accuracy per tool on compounds like those of the dendroides dataset (MassIVE MSV000080502).

## When to use

Use when you have MS/MS spectra of a compound set whose molecular formulas/structures were validated by spectral library search and manual inspection, and you need to quantify annotation accuracy of a formula- or structure-annotation tool.

## When NOT to use

- When the compound set has no independently validated annotations — no ground truth exists, so no accuracy can be computed
- When you need structural rather than molecular-formula accuracy and the tool only outputs formulas (or vice versa) — the comparison must match annotation level
- When the goal is discovery of novel formulas rather than benchmarking — novel formulas by definition lack library-validated ground truth

## Inputs

- MS/MS spectra (mzML/mzXML) of compounds with validated formulas/structures
- Ground-truth table mapping each compound to its validated molecular formula and/or structure (e.g. groundtruth_dendroides.csv)
- Tool output with top-ranked candidate per compound (SIRIUS formula list, ZODIAC reranked formula list, or CSI:FingerID structure list)

## Outputs

- Per-compound exact-match table (compound id, predicted top-1 annotation, ground truth, correct/incorrect)
- Top-1 annotation accuracy per tool (fraction of compounds with exact top-1 agreement)

## How to apply

First establish the ground truth independently of the tool under test: search each MS/MS spectrum against a spectral library (e.g. NIST17 for the ZODIAC evaluation), then manually validate library hits, precursor mass and isotope pattern before accepting a compound's formula/structure as known. Extract each compound's top-ranked candidate from the tool output (SIRUS formula, ZODIAC formula, or CSI:FingerID structure), and compare it to the ground-truth value compound by compound, counting a hit only on exact top-1 agreement. Compute accuracy as the fraction of compounds with the correct top-ranked annotation, and optionally condition on molecular mass (e.g. accuracy loss above 500 Da is the key regime highlighted by the paper). Report per-tool accuracy so the effect of a reranking or constraint (e.g. ZODIAC under known-adducts: 67.17% SIRIUS vs 99.50% ZODIAC on 135 dendroides compounds) is directly visible. The ground truth must remain untouched by the tool being evaluated — no parameter tuning on the evaluation set.

## Related tools

- **SIRIUS** (Produces top-ranked molecular formula candidates that are scored against the ground truth; also used to establish ground truth via fragmentation tree and isotope analysis) — https://github.com/sirius-ms/sirius
- **ZODIAC** (Reranks SIRIUS candidates; its top-1 output is the second prediction being scored against the same ground truth to measure improvement) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Structure-level annotation whose top-1 hits can serve as ground-truth anchors during evaluation)
- **OpenMS** (Preprocesses input mzML/mzXML and discards low-quality MS/MS spectra before ground-truth evaluation) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Examples

```
sirius --input dendroides.mzML --groundtruth groundtruth_dendroides.csv --output accuracy_known_adducts.csv
```

## Evaluation signals

- Every compound in the accuracy table has both a predicted and a ground-truth value (no orphan rows)
- Top-1 accuracy is computed on the full compound list, not on a filtered subset, unless the filter is stated (e.g. known-adducts condition)
- Accuracy numbers are reproducible from the raw tool outputs — re-deriving the fraction from SIRIUS/ZODIAC top-1 hits matches the reported value (e.g. 135/200 ≈ 67.17% for dendroides SIRIUS)
- Accuracy drops markedly for compounds above 500 Da, consistent with the paper's reported regime
- Ground-truth assignments are traceable to a spectral library search plus manual validation step, not to the tool under test

## Limitations

- Ground truth covers only compounds with reliable library hits or manual validation; compounds with novel formulas (e.g. C24H47BrNO8P) cannot be included, biasing accuracy estimates toward known chemistry
- Complete evaluation of novel molecular formula annotations would require full structural elucidation, which is experimentally infeasible
- Adduct information cannot be reliably deduced from isotope pattern and MS/MS data, so ground truth must fix adducts during preprocessing (high-quality adduct annotations should be established during preprocessing)
- Library-hit anchors contribute only a small effect to molecular formula annotations, so evaluation with/without anchors shows limited contrast

## Evidence

- [results] Known-adducts accuracy on dendroides: [evidence span withheld — anchor status: not_found]
- [discussion] Adducts must be fixed during preprocessing: [evidence span withheld — anchor status: not_found]
