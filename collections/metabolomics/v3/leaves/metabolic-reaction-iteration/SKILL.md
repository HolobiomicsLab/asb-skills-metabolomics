---
name: metabolic-reaction-iteration
description: Use when labeled Metabolite CNs are too sparse to train a pathway predictor
  and only a subset of known chemical reactions (hyperlinks) is available to imitate
  the organism's metabolic activity and generate artificial networks.
when_to_use_negative:
- Input already consists of experimentally measured metabolomics profiles with sufficient
  labeled Metabolite CNs — simulation-based augmentation is meant to mitigate sparsity
  of labeled data
- The goal is graph auralization or M5 classifier training (waveform extraction and
  classification), which consumes the augmented networks downstream rather than producing
  them
- The available pathway pool lacks directional substrate-product information, so reactions
  cannot be executed as transformations
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: NetAurHPD
  role: Framework whose data augmentation component executes simulated metabolic pathways
    and constructs artificial Metabolite CNs; augmented networks feed the auralization
    pipeline
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: PyTorch
  role: Implementation framework used for the network algorithms, per Algorithm 1
    pseudo-code adapted for PyTorch
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:d9845827f66c03f23bfeba44e3d54344136b21f3c5c8fec54d12a14f6e0431f4
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolic-reaction-iteration

## Summary

A data augmentation procedure that mimics metabolic activity by iteratively executing known metabolic pathways (directional reactions from a 126-pathway tomato pericarp pool), then computes Pearson Correlation Coefficients between every metabolite pair to build artificial Metabolite CNs that enrich training data for NetAurHPD.

## When to use

Use when labeled Metabolite CNs are too sparse to train a pathway predictor and only a subset of known chemical reactions (hyperlinks) is available to imitate the organism's metabolic activity and generate artificial networks.

## When NOT to use

- Input already consists of experimentally measured metabolomics profiles with sufficient labeled Metabolite CNs — simulation-based augmentation is meant to mitigate sparsity of labeled data
- The goal is graph auralization or M5 classifier training (waveform extraction and classification), which consumes the augmented networks downstream rather than producing them
- The available pathway pool lacks directional substrate-product information, so reactions cannot be executed as transformations

## Inputs

- Directional pathway pool of known reactions (substrate/product metabolite labels), e.g., 126 tomato pericarp pathways sourced from TomatoCyc
- List of metabolites present in the Metabolite CN (e.g., the 52 common metabolites)
- Number of augmentations to generate (10 in the paper) and species/sample subgroup definitions

## Outputs

- 10 Augmentation Metabolite CNs (correlation networks over the same metabolite set)
- Per-metabolite degree-0 entries with ones waveforms for metabolites dropped from Augmentations

## How to apply

Load the directional pathway pool (e.g., 126 known tomato pericarp pathways from TomatoCyc) to label substrate and product metabolites, keeping only pathways with more than two compounds appearing in the Metabolite CN. Create Species subgroups representing organism variants and generate several samples per species, each initialized with a fixed concentration level uniformly distributed across metabolites. Mimic metabolic activity by iteratively executing reactions — transforming substrates into products when participant metabolites are in sufficient amounts — stopping after a fixed number of iterations or when no reaction can proceed. Compute the Pearson Correlation Coefficient between every pair of metabolites over the final sample states and apply the correlation threshold |r| >= 0.3 and p <= 0.01 to build each Augmentation Metabolite CN. Repeat to produce 10 Augmentations, adding any metabolite missing from the Augmentations (from dropped pathways) with degree 0 so its waveform is ones.

## Related tools

- **NetAurHPD** (Framework whose data augmentation component executes simulated metabolic pathways and constructs artificial Metabolite CNs; augmented networks feed the auralization pipeline) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **PyTorch** (Implementation framework used for the network algorithms, per Algorithm 1 pseudo-code adapted for PyTorch)

## Evaluation signals

- Exactly 10 Augmentation networks are produced, each a valid Metabolite CN
- Node degrees in each Augmentation CN are consistent with the simulated metabolic activity
- Every retained metabolite pair satisfies the correlation threshold |r| >= 0.3 and p <= 0.01
- Metabolites missing from Augmentations (from dropped pathways) are added with degree 0 so their waveform is ones
- Procedure matches the description illustrated in Supplementary Fig 8 of the paper

## Limitations

- Sparsity of labeled data remains the central challenge; augmentation does not fully substitute for real measurements
- Models do not consistently improve with more data — in some sessions the AUCs of all models decreased
- Only pathways with more than two compounds appearing in the Metabolite CN can be included, so some reactions are dropped and their metabolites become degree-0 nodes

## Evidence

- [methods] mimic metabolic activity: [evidence span withheld — anchor status: not_found]
- [methods] pearson correlation construction: [evidence span withheld — anchor status: not_found]
- [methods] correlation threshold: [evidence span withheld — anchor status: not_found]
