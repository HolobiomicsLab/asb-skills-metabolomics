---
name: synthetic-metabolomics-data-augmentation
description: Use when labeled pathway data is too sparse to train a pathway-prediction
  model like NetAurHPD, and a subset of known chemical reactions (e.g., TomatoCyc
  directional pathways) is available to imitate metabolic activity.
when_to_use_negative:
- The goal is network auralization or waveform representation of an existing real
  Metabolite CN (Algorithm 1 / SignalPropagation), not synthetic network generation.
- You need to predict pathway presence from already-trained M5 waveforms — then only
  hyperlink waveform averaging and Youden-index threshold optimization are needed.
- The organism's pathway pool is not directional or lacks substrate/product labels,
  so reactions cannot be executed as substrate-to-product transformations.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: NetAurHPD
  role: Framework whose data-augmentation module consumes the generated artificial
    Metabolite CNs to enrich M5 hyperlink-prediction training
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: PyTorch
  role: Implementation backend for the NetAurHPD pipeline into which the augmented
    networks feed
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:311b1d921b98e4b0fa5f12ddcb50b4f30a029c23c505c22f14cbc831d707458c
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - NetAurHPD
  - PyTorch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# synthetic-metabolomics-data-augmentation

## Summary

Generates artificial Metabolite CNs by simulating metabolic activity over a directional pool of known reactions (e.g., 126 tomato pericarp pathways) and computing Pearson correlations between metabolites, to mitigate the sparsity of labeled data for hyperlink prediction.

## When to use

Use when labeled pathway data is too sparse to train a pathway-prediction model like NetAurHPD, and a subset of known chemical reactions (e.g., TomatoCyc directional pathways) is available to imitate metabolic activity.

## When NOT to use

- The goal is network auralization or waveform representation of an existing real Metabolite CN (Algorithm 1 / SignalPropagation), not synthetic network generation.
- You need to predict pathway presence from already-trained M5 waveforms — then only hyperlink waveform averaging and Youden-index threshold optimization are needed.
- The organism's pathway pool is not directional or lacks substrate/product labels, so reactions cannot be executed as substrate-to-product transformations.

## Inputs

- Directional pathway pool of known reactions (e.g., 126 tomato pericarp pathways from TomatoCyc/MetaCyc) with substrate and product metabolite labels
- List of metabolites appearing in the Metabolite CN (e.g., the 52 common metabolites)
- Number of Augmentations to generate (10) and per-species sample counts with uniform initial concentrations

## Outputs

- 10 Augmentation Metabolite CNs (correlation networks, |r| ≥ 0.3, p ≤ 0.01)
- Degree-0 entries for metabolites dropped from a given Augmentation, whose waveform is ones
- Enriched training set combining real and artificial Metabolite CNs for M5 classifier training

## How to apply

Load the directional pathway pool (126 known tomato pericarp pathways) to label substrate and product metabolites, and create Species subgroups with several samples per species, each initialized with fixed metabolite concentration levels drawn uniformly. Mimic metabolic activity by iteratively executing reactions — transforming substrates into products only when participant metabolites are present in sufficient amounts — stopping after a fixed number of iterations or when no reaction can proceed. Compute the Pearson Correlation Coefficient between every pair of metabolites over the final sample states and build each Augmentation Metabolite CN with the correlation threshold |r| ≥ 0.3 and p ≤ 0.01. Repeat to produce 10 Augmentations, and add any metabolite missing from an Augmentation (from dropped pathways) with degree 0 so its waveform is ones; this enrichment improved NetAurHPD's learning capabilities and overall performance.

## Related tools

- **NetAurHPD** (Framework whose data-augmentation module consumes the generated artificial Metabolite CNs to enrich M5 hyperlink-prediction training) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **PyTorch** (Implementation backend for the NetAurHPD pipeline into which the augmented networks feed)

## Evaluation signals

- Exactly 10 Augmentation Metabolite CNs are produced, matching the paper's procedure (Supplementary Fig 8).
- Each Augmentation is a valid Metabolite CN whose node degrees are consistent with the simulated metabolic activity.
- Edge inclusion respects the |r| ≥ 0.3 and p ≤ 0.01 Pearson correlation threshold.
- Every metabolite from the original node set appears in each Augmentation; metabolites lost via dropped pathways have degree 0 and a ones waveform.
- Simulated reactions stop either after the fixed iteration count or when no reaction can proceed (all required substrates exhausted).

## Limitations

- Models do not consistently improve with more data; in some sessions the AUCs of all models decreased, so adding Augmentations is not guaranteed to help.
- NetAurHPD (downstream consumer of these Augmentations) has a low level of explainability and cannot provide biologists with a full understanding of its results.
- Sparsity of labeled data remains the core motivation; augmentation quality depends entirely on the coverage of the known reaction pool.

## Evidence

- [methods] the selected correlation threshold was set to be |r|≥ 0.3 and p≤0.01: [evidence span withheld — anchor status: not_found]
