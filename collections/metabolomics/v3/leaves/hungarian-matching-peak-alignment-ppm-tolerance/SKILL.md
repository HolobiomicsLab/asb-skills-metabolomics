---
name: hungarian-matching-peak-alignment-ppm-tolerance
description: Use when comparing a predicted high-resolution MS/MS spectrum to a reference
  spectrum where peak mass accuracy matters — e.g. evaluating fragment-annotated models
  on Orbitrap [M+H]+ HCD data where 0.01 Da binning would blur small mass offsets.
when_to_use_negative:
- When the metric of interest is deliberately mass-accuracy-insensitive — use binned
  cosine C_BIN (0.01 Da bins) instead, as the paper reports both
- When spectra come from low-resolution instruments or other adducts/instrument platforms,
  since the 10 ppm threshold was calibrated on Orbitrap [M+H]+ HCD data
- When peaks must be interpreted as formula annotations rather than just scored —
  the paper excludes formula annotations more than 10 ppm from peak mass, a related
  but distinct filter
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FraGNNet
  role: Source of high-mass-accuracy predicted spectra whose peaks are matched to
    reference peaks under the 10 ppm tolerance
  repo: github.com/FraGNNet/fragnnet
- name: Python
  role: Implementation language for the evaluation scripts computing C_HUN over the
    NIST20 test set
- name: PyTorch
  role: Framework in which the models producing the predicted spectra were implemented
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:112078bc25dc345fd120cdcde04e7b83dcc88ae9029865d469d6d814cc54e1e9
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cython
  - FraGNNet
  - GrAFF-MS
  - PyTorch
  - PyTorch Geometric
  - PyTorch Lightning
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 22
---

# Hungarian-matching peak alignment with ppm tolerance

## Summary

Scores predicted vs. real MS/MS spectra by optimally matching peaks with the Hungarian algorithm under a 10 ppm mass tolerance, yielding the C_HUN similarity metric used alongside binned cosine (C_BIN) to benchmark C2MS models like FraGNNet against ICEBERG, GrAFF-MS, MassFormer, and NEIMS.

## When to use

Use when comparing a predicted high-resolution MS/MS spectrum to a reference spectrum where peak mass accuracy matters — e.g. evaluating fragment-annotated models on Orbitrap [M+H]+ HCD data where 0.01 Da binning would blur small mass offsets.

## When NOT to use

- When the metric of interest is deliberately mass-accuracy-insensitive — use binned cosine C_BIN (0.01 Da bins) instead, as the paper reports both
- When spectra come from low-resolution instruments or other adducts/instrument platforms, since the 10 ppm threshold was calibrated on Orbitrap [M+H]+ HCD data
- When peaks must be interpreted as formula annotations rather than just scored — the paper excludes formula annotations more than 10 ppm from peak mass, a related but distinct filter

## Inputs

- Predicted MS/MS spectra (peak m/z + intensity lists) from FraGNNet-D3/D4 or baseline C2MS models
- Real held-out NIST20 MS/MS test spectra ([M+H]+ Orbitrap HCD)
- Scaffold and InChIKey split assignments for the test molecules

## Outputs

- Per-molecule Hungarian cosine similarity scores (C_HUN, 10 ppm tolerance)
- Aggregated mean ± std C_HUN per model per split (Table 1-equivalent, spectrum_prediction_benchmark.csv)

## How to apply

Take the predicted spectrum (from FraGNNet's fragment distribution converted to peaks via formula masses, or from a baseline model) and the real NIST20 test spectrum, and compute an optimal one-to-one peak assignment with the Hungarian algorithm, counting a match only when the predicted and observed peak masses agree within 10 ppm. Average the resulting Hungarian cosine similarity (C_HUN) over the held-out test set, separately for the InChIKey split and the harder scaffold split. Report C_HUN alongside the binned cosine C_BIN (0.01 Da bins), because C_HUN is sensitive to mass accuracy while C_BIN is not — this distinction is why the fragment-based FraGNNet approach is evaluated with both. Run the evaluation over 5 random seeds and report mean ± standard deviation per model per split.

## Related tools

- **FraGNNet** (Source of high-mass-accuracy predicted spectra whose peaks are matched to reference peaks under the 10 ppm tolerance) — github.com/FraGNNet/fragnnet
- **Python** (Implementation language for the evaluation scripts computing C_HUN over the NIST20 test set)
- **PyTorch** (Framework in which the models producing the predicted spectra were implemented)

## Examples

```
python scripts/evaluate.py --model fragnnet-d4 --split inchikey --metrics c_bin,c_hun --c_hun_tol_ppm 10 --seeds 5 --out spectrum_prediction_benchmark.csv
```

## Evaluation signals

- FraGNNet-D4 should reach C_HUN ≈ 0.736 ± 0.002 on the InChIKey split and ≈ 0.709 ± 0.002 on the scaffold split (with the analogous C_BIN values), outperforming all baselines
- Standard deviation across 5 random seeds should stay small (order 0.002) for the top-performing model
- Scaffold-split scores should be lower than InChIKey-split scores for the same model, reflecting the harder split
- C_HUN should discriminate mass-accuracy differences between models that C_BIN (0.01 Da bins) cannot

## Limitations

- The 10 ppm matching tolerance was established on merged Orbitrap [M+H]+ HCD spectra; performance on unmerged spectra or other instrument platforms/adducts is not established
- Unexplained 'Outside of the Support' (OS) peaks arise from fragments the recursive bond-breaking algorithm does not account for (e.g. cyclizations), which depress matching scores in ways unrelated to the metric itself

## Evidence

- [methods] C_HUN definition (10 ppm Hungarian matching): [evidence span withheld — anchor status: not_found]
- [methods] Evaluation dataset scope: [evidence span withheld — anchor status: not_found]
