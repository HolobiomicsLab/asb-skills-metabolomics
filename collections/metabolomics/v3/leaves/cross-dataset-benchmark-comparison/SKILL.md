---
name: cross-dataset-benchmark-comparison
description: Use when a spectral/annotation method's accuracy claim must be validated
  across two or more independent test sets (e.g. NPLIB1 vs MassSpecGym) rather than
  a single benchmark.
when_to_use_negative:
- Spectra lack the high-resolution MS1 peak lists needed for formula annotation, so
  ranked formula candidates cannot be produced.
- The datasets being compared have different annotation conventions or split definitions,
  so accuracy figures would not be comparable.
- You only need structure generation metrics and already have guaranteed ground-truth
  formulae — the annotation-tool comparison step adds no value.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: MIST-CF
  role: Formula annotation tool run on each test spectrum to output top-5 ranked molecular
    formula candidates
  repo: github.com/coleygroup/MIST
- name: BUDDY
  role: Second formula annotation tool run on the same test spectra for its top-5
    ranked formula candidates
- name: DiffMS
  role: Downstream de novo structure generator evaluated with predicted top-5 formulae
    in place of ground truth
  repo: github.com/coleygroup/DiffMS
- name: PyTorch
  role: Runtime for DiffMS and MIST models
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:883e1ca6f6d0d4f7d5298f688879767cc115facb5794cf13036bc171102176c0
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BUDDY
  - DiffMS
  - MIST-CF
  - PyTorch
  - coleygroup/MIST
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# cross-dataset-benchmark-comparison

## Summary

Evaluates formula-annotation tools (MIST-CF, BUDDY) and a downstream de novo structure generator (DiffMS) side-by-side on two public benchmarks, NPLIB1 (CANOPUS/GNPS subset) and MassSpecGym, so that performance conclusions are not tied to a single dataset.

## When to use

Use when a spectral/annotation method's accuracy claim must be validated across two or more independent test sets (e.g. NPLIB1 vs MassSpecGym) rather than a single benchmark.

## When NOT to use

- Spectra lack the high-resolution MS1 peak lists needed for formula annotation, so ranked formula candidates cannot be produced.
- The datasets being compared have different annotation conventions or split definitions, so accuracy figures would not be comparable.
- You only need structure generation metrics and already have guaranteed ground-truth formulae — the annotation-tool comparison step adds no value.

## Inputs

- NPLIB1 test spectra (CANOPUS subset of GNPS data) with peak lists
- MassSpecGym test spectra with peak lists
- Ground-truth molecular formulae for each test spectrum
- Pretrained DiffMS end-to-end checkpoints (Zenodo 23542112) and MIST encoder checkpoints (Zenodo 23303289)
- MIST-CF (coleygroup/MIST) and BUDDY formula annotation tools

## Outputs

- formula_predictions.csv with per-spectrum ranked top-5 formula candidates from MIST-CF and BUDDY
- Per-dataset top-1 and top-5 formula annotation accuracies
- DiffMS de novo generation metrics (MCES, Tanimoto) computed with predicted vs ground-truth formulae on both benchmarks

## How to apply

Process both benchmark spectra with the DiffMS data_processing scripts (01 for CANOPUS/NPLIB1, 02 for MassSpecGym) so peak lists are in a consistent format. Run MIST-CF and BUDDY on each test spectrum to obtain top-5 ranked molecular formula candidates, and compare each ranked list against the known ground-truth formula to compute top-1 and top-5 annotation accuracies per dataset. Then re-run DiffMS (e.g. with pretrained end-to-end checkpoints from Zenodo record 23542112) supplying the predicted top-5 formulae instead of ground truth, and evaluate structure generation with MCES and Tanimoto metrics on both datasets. The rationale is that formula annotation tools can be unreliable in real elucidation settings, so DiffMS's robustness when the formula is supplied by annotation tools is itself a cross-dataset benchmark result. Report metrics per dataset rather than pooled, since accuracy and diversity effects (e.g. higher MCES/Tanimoto due to candidate diversity) can diverge.

## Related tools

- **MIST-CF** (Formula annotation tool run on each test spectrum to output top-5 ranked molecular formula candidates) — github.com/coleygroup/MIST
- **BUDDY** (Second formula annotation tool run on the same test spectra for its top-5 ranked formula candidates)
- **DiffMS** (Downstream de novo structure generator evaluated with predicted top-5 formulae in place of ground truth) — github.com/coleygroup/DiffMS
- **PyTorch** (Runtime for DiffMS and MIST models)

## Examples

```
bash data_processing/01_download_canopus_data.sh && bash data_processing/02_download_msg_data.sh
```

## Evaluation signals

- All test spectra in both NPLIB1 and MassSpecGym receive a non-empty ranked formula list from each annotation tool.
- Computed top-1/top-5 annotation accuracies fall in the expected range reported for the benchmark.
- DiffMS metrics with predicted formulae are only slightly lower than with ground-truth formulae, while MCES/Tanimoto may be equal or better due to candidate diversity.
- Per-dataset metrics are reported separately for NPLIB1 and MassSpecGym (dataset config 'canopus' vs 'msg' in DiffMS).
- formula_predictions.csv covers every test spectrum with exactly 5 ranked candidates per tool.

## Limitations

- MIST-CF does not consider isotope distributions, unlike tools such as SIRIUS that exploit isotopic traces.
- Formulae should be inferred from high-resolution MS1 data; low-resolution spectra may not support reliable annotation.
- Accuracy with predicted formulae is slightly lower than with true formulae; higher MCES/Tanimoto can arise from prediction diversity rather than better structure elucidation.
- The DiffMS codebase is adapted from DiGress, and a reported bug (dim_ffy ignored in GraphTransformer, default 2048) means described paper values may differ from code defaults.

## Evidence

- [methods] Formulae can be inferred from high-resolution MS1 data and isotopic traces with sufficient accuracy using tools like SIRIUS: "Formulae can be inferred from high-resolution MS1 data and isotopic traces with sufficient accuracy using tools like SIRIUS"
- [methods] benchmark datasets: [evidence span withheld — anchor status: not_found]
- [intro] formula as given: "One insight of our work is to use those available tools by taking the chemical formula as given and developing a formula-constrained (i.e., heavy atom-constrained) generation pipeline"
- [methods] MIST-CF limitation: [evidence span withheld — anchor status: not_found]
- [readme] data processing scripts: "To download/setup the datasets, run the scripts in the data_processing/ folder in order"
