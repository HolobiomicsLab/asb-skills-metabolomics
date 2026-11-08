---
name: metabolite-concentration-simulation
description: Use when labeled pathway data is sparse (e.g. few known tomato pericarp
  pathways) and artificial Metabolite CNs are needed to augment training of a hyperlink
  predictor like NetAurHPD.
when_to_use_negative:
- The goal is auralization/waveform extraction or hyperlink classification from an
  existing Metabolite CN — use the SignalPropagation/M5 pipeline instead; this skill
  only generates artificial CNs from a subset of known reactions.
- Sufficient real, labeled Metabolite CNs (e.g. multiple harvesting seasons of tomato
  pericarp metabolomics data) already exist, so artificial augmentation is unnecessary.
- Candidate pathways involve metabolites absent from the Metabolite CN — only pathways
  with more than two compounds that appear in the Metabolite CN were included in the
  dataset.
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_0091
tools:
- name: PyTorch
  role: Implementation framework for the simulation/augmentation algorithms described
    in the paper's methods (Algorithm-style pseudo-code adapted for PyTorch).
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: NetAurHPD
  role: 'Downstream consumer: the artificial Metabolite CNs produced here augment
    training data for the NetAurHPD hyperlink prediction classifier.'
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: TomatoCyc
  role: Source of the pathway collection from which the directional pathway pool of
    known tomato reactions is drawn.
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:d324ad8da553e0ed4d17440878a77bdcb939ddc4a9b5d2f5a075e59982aae9c5
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - NetAurHPD
  - PyTorch
  - TomatoCyc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolite-concentration-simulation

## Summary

A data augmentation procedure that generates artificial Metabolite CNs by simulating metabolic activity: reactions from a subset of known pathways are iteratively executed over synthetic samples, then Pearson correlations between metabolites are thresholded to build each augmentation network.

## When to use

Use when labeled pathway data is sparse (e.g. few known tomato pericarp pathways) and artificial Metabolite CNs are needed to augment training of a hyperlink predictor like NetAurHPD.

## When NOT to use

- The goal is auralization/waveform extraction or hyperlink classification from an existing Metabolite CN — use the SignalPropagation/M5 pipeline instead; this skill only generates artificial CNs from a subset of known reactions.
- Sufficient real, labeled Metabolite CNs (e.g. multiple harvesting seasons of tomato pericarp metabolomics data) already exist, so artificial augmentation is unnecessary.
- Candidate pathways involve metabolites absent from the Metabolite CN — only pathways with more than two compounds that appear in the Metabolite CN were included in the dataset.

## Inputs

- Directional pathway pool of 126 known tomato pericarp metabolic pathways (substrate→product reactions, sourced from TomatoCyc)
- Set of metabolites appearing in the target Metabolite CN (e.g. the 52 common metabolites)
- Number of Species subgroups (organism variants) and number of samples per species
- Fixed initial concentration level uniformly distributed across metabolites
- Stopping criterion: fixed number of reaction iterations or no executable reaction remaining

## Outputs

- 10 Augmentation Metabolite CNs built from Pearson correlations with threshold |r| ≥ 0.3 and p ≤ 0.01
- Degree-0 entries (waveform of ones) for metabolites missing from the augmentations due to dropped pathways

## How to apply

Load the directional pathway pool (126 known tomato pericarp pathways from TomatoCyc) to label substrate and product metabolites. Create Species subgroups representing organism variants and generate several samples per species, each initialized with a fixed concentration level uniformly distributed across the metabolites. Mimic metabolic activity by iteratively executing reactions — transforming substrates into products when participant metabolites are present in sufficient amounts — stopping after a fixed number of iterations or when no reaction can proceed. Compute the Pearson Correlation Coefficient between every pair of metabolites over the final sample states and apply the threshold |r| ≥ 0.3 and p ≤ 0.01 to build each Augmentation Metabolite CN. Repeat to produce 10 Augmentations, adding any metabolite dropped from the augmentations with degree 0 so its waveform is ones; validate that all 10 networks are valid Metabolite CNs whose node degrees are consistent with the simulated metabolic activity (Supplementary Fig 8).

## Related tools

- **PyTorch** (Implementation framework for the simulation/augmentation algorithms described in the paper's methods (Algorithm-style pseudo-code adapted for PyTorch).) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **NetAurHPD** (Downstream consumer: the artificial Metabolite CNs produced here augment training data for the NetAurHPD hyperlink prediction classifier.) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **TomatoCyc** (Source of the pathway collection from which the directional pathway pool of known tomato reactions is drawn.)

## Evaluation signals

- Exactly 10 Augmentation networks are produced after repeated simulation runs.
- Each Augmentation is a valid Metabolite CN whose node degrees are consistent with the simulated metabolic activity.
- Correlation threshold |r| ≥ 0.3 and p ≤ 0.01 was applied when constructing each network.
- Any metabolite missing from the augmentations (from dropped pathways) appears with degree 0 so its waveform is ones.
- The generated networks match the procedure described in the paper (Supplementary Fig 8) and improve NetAurHPD's learning capabilities and overall performance.

## Limitations

- The augmentation is limited to the subset of known reactions (126 tomato pericarp pathways), so artificial networks only reflect reactions already labeled.
- Models do not consistently improve with more data; in some sessions the AUCs of all models decreased despite augmentation.
- Metabolites dropped from augmentations are added artificially with degree 0 (ones waveform), which carries no real correlation information.

## Evidence

- [methods] We mimic metabolic activity by executing a series of metabolic pathways (Supplementary Fig 8.d): [evidence span withheld — anchor status: not_found]
- [methods] we calculate the Pearson Correlation Coefficient between every pair of metabolites (Supplementary Fig 8.f): [evidence span withheld — anchor status: not_found]
- [methods] the selected correlation threshold was set to be |r|≥ 0.3 and p≤0.01: [evidence span withheld — anchor status: not_found]
