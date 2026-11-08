---
name: spectral-formula-prediction-inference
description: Use when you have processed tandem MS test spectra (peak lists, precursor
  m/z) whose true formula may be unknown and you need ranked formula candidates to
  feed downstream de novo structure generation.
when_to_use_negative:
- Ground-truth chemical formulae are already available — use them directly, since
  predicted-formula conditioning yields slightly lower DiffMS accuracy than true formulae
- Spectra lack the high-resolution MS1 precursor information required for formula
  annotation
- The goal is molecular fingerprint prediction or structure generation rather than
  formula-level annotation
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MIST-CF
  role: Chemical formula transformer run on each test spectrum to output top-5 ranked
    formula candidates
  repo: https://github.com/coleygroup/MIST
- name: BUDDY
  role: Second formula annotation tool run on the same test spectra to obtain top-5
    ranked candidates for comparison
- name: DiffMS
  role: Downstream consumer of the predicted top-5 formulae as conditioning input
    to formula-restricted structure generation
  repo: https://github.com/coleygroup/DiffMS
- name: PyTorch
  role: Runtime for MIST-CF inference (tested with PyTorch 2.3.1 in the DiffMS codebase;
    MIST requires PyTorch 2.7+)
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:d4f5d63bbf5625e0e9e54e6aa19919300d3ef7580d6909056f95827701563a1a
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/MIST
  tools:
  - BUDDY
  - DiffMS
  - MIST-CF
  - PyTorch
  - coleygroup/MIST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# spectral-formula-prediction-inference

## Summary

Annotate tandem mass spectra with ranked molecular formula candidates using out-of-the-box tools (MIST-CF, BUDDY), then quantify top-1/top-5 annotation accuracy against known ground-truth formulae on NPLIB1 and MassSpecGym test sets.

## When to use

Use when you have processed tandem MS test spectra (peak lists, precursor m/z) whose true formula may be unknown and you need ranked formula candidates to feed downstream de novo structure generation.

## When NOT to use

- Ground-truth chemical formulae are already available — use them directly, since predicted-formula conditioning yields slightly lower DiffMS accuracy than true formulae
- Spectra lack the high-resolution MS1 precursor information required for formula annotation
- The goal is molecular fingerprint prediction or structure generation rather than formula-level annotation

## Inputs

- Processed NPLIB1/CANOPUS test spectra (from data_processing/01_download_canopus_data.sh)
- Processed MassSpecGym test spectra (from data_processing/02_download_msg_data.sh)
- Peak lists with precursor m/z used for formula annotation
- Ground-truth molecular formulae for each test spectrum

## Outputs

- formula_predictions.csv with per-spectrum top-5 ranked formula candidates from MIST-CF and BUDDY
- Top-1 and top-5 formula annotation accuracies per tool per benchmark

## How to apply

First prepare the NPLIB1 (CANOPUS) and MassSpecGym test spectra by running the DiffMS data_processing scripts 01 and 02, which yield peak lists suitable for formula annotation. Run MIST-CF (coleygroup/MIST) on each test spectrum to produce the top-5 ranked molecular formula candidates per spectrum, and run BUDDY on the same spectra to obtain its top-5 ranked candidates. Note that MIST-CF does not consider isotope distributions, whereas formulae can also be inferred from high-resolution MS1 data and isotopic traces with tools like SIRIUS. Compare each tool's ranked list against the known ground-truth formula of each spectrum to compute top-1 and top-5 annotation accuracies. Finally write per-spectrum formula predictions and computed accuracies to formula_predictions.csv; these predictions can then substitute ground-truth formulae when conditioning DiffMS's formula-constrained (heavy atom-constrained) generation pipeline.

## Related tools

- **MIST-CF** (Chemical formula transformer run on each test spectrum to output top-5 ranked formula candidates) — https://github.com/coleygroup/MIST
- **BUDDY** (Second formula annotation tool run on the same test spectra to obtain top-5 ranked candidates for comparison)
- **DiffMS** (Downstream consumer of the predicted top-5 formulae as conditioning input to formula-restricted structure generation) — https://github.com/coleygroup/DiffMS
- **PyTorch** (Runtime for MIST-CF inference (tested with PyTorch 2.3.1 in the DiffMS codebase; MIST requires PyTorch 2.7+))

## Examples

```
bash data_processing/01_download_canopus_data.sh && bash data_processing/02_download_msg_data.sh && mist-download canopus-compact && mist-download canopus-noaug --output-dir checkpoints
```

## Evaluation signals

- All test spectra in both NPLIB1 and MassSpecGym receive a complete ranked formula list (no missing spectra)
- Computed top-1 and top-5 annotation accuracies fall within the expected range reported for MIST-CF and BUDDY on these benchmarks
- formula_predictions.csv contains a ranked top-5 formula list per spectrum with correct schema
- Ranked candidate lists agree with known ground-truth formulae at the reported top-1/top-5 rates
- DiffMS conditioned on the predicted top-5 formulae retains strong de novo performance close to (or exceeding on MCES/Tanimoto) its ground-truth-formula runs

## Limitations

- MIST-CF does not consider isotope distributions, unlike tools such as SIRIUS that exploit isotopic traces
- DiffMS accuracy is slightly lower when relying on predicted (MIST-CF top-5) formulae than on ground-truth formulae, though MCES and Tanimoto are sometimes better due to higher candidate diversity
- BUDDY is referenced as a formula inference tool but no run configuration or checkpoint is provided in this context

## Evidence

- [intro] utilizing tools such as SIRIUS, BUDDY, or MIST-CF: [evidence span withheld — anchor status: not_found]
- [methods] the latter does not consider isotope distributions: [evidence span withheld — anchor status: not_found]
- [readme] MIST predicts molecular fingerprints from tandem mass spectra using a chemical formula transformer: [evidence span withheld — anchor status: external_doc]
