---
name: augmented-training-set-composition-and-comparison
description: Use when training NetAurHPD (or a similar M5-based classifier) on metabolite
  correlation networks and you must decide how many artificial augmented networks
  to include in training, or when you need to quantify how augmentation affects AUC
  stability across repeated cross-validation sessions.
when_to_use_negative:
- When the labeled pathway data is already dense — augmentation was motivated by the
  sparsity of labeled data in metabolic pathway prediction, and 'the models do not
  consistently improve with more data; in some sessions the AUCs of all models decreased'
- When only a single training configuration is needed without variability assessment
  — this skill specifically requires 20 repeated sessions to quantify AUC spread
- When evaluating alternative architectures (e.g. CHESHIRE) rather than the augmentation
  composition of the NetAurHPD M5 classifier
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0605
tools:
- name: NetAurHPD
  role: 'Framework being ablated: its M5 classifier is retrained with 1/3/5/7 Augmentations
    and baseline signals'
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: M5
  role: Very deep convolutional neural network trained per fold as classification
    model on pathway waveforms
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: Adam optimizer
  role: Optimizer used at learning rate 0.00001 for 800 epochs during each augmented/baseline
    training run
- name: PyTorch
  role: Implementation backend for the network auralization and M5 training pipeline
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:7c8368f23cd135e7206b5788aca769cab0635ad8666a98fda93c7687b7754fe4
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - Adam optimizer
  - M5
  - NetAurHPD
  - PyTorch
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# augmented-training-set-composition-and-comparison

## Summary

A systematic ablation procedure for composing classifier training sets with different numbers of data augmentations (1, 3, 5, or 7 out of 10 pre-generated artificial Metabolite CNs) and comparing their micro-average AUC and variability to identify the optimal augmentation count.

## When to use

Use when training NetAurHPD (or a similar M5-based classifier) on metabolite correlation networks and you must decide how many artificial augmented networks to include in training, or when you need to quantify how augmentation affects AUC stability across repeated cross-validation sessions.

## When NOT to use

- When the labeled pathway data is already dense — augmentation was motivated by the sparsity of labeled data in metabolic pathway prediction, and 'the models do not consistently improve with more data; in some sessions the AUCs of all models decreased'
- When only a single training configuration is needed without variability assessment — this skill specifically requires 20 repeated sessions to quantify AUC spread
- When evaluating alternative architectures (e.g. CHESHIRE) rather than the augmentation composition of the NetAurHPD M5 classifier

## Inputs

- Metabolite CNs of tomato pericarp for harvesting seasons 2001, 2003, and 2004
- 126 directional TomatoCyc pathways (positive targets)
- 85 random negative pathways sampled as subsets of the 52 common metabolites
- 10 pre-computed Augmentations (artificial Metabolite CNs generated from subsets of known reactions)

## Outputs

- Micro-average AUC per model (baseline, 1/3/5/7 Augmentations) per year across 20 cross-validation sessions
- Box plots per number of Augmentations showing median, mean, maximum AUC, interquartile range, and whisker range (Fig 4 format)
- Selected optimal augmentation count (here: one Augmentation, micro-average AUC 0.87 in 2001, 0.857 in 2003, 0.884 in 2004)

## How to apply

Load the tomato pericarp Metabolite CNs for 2001, 2003, and 2004 together with the 126 directional TomatoCyc pathways used to pre-generate 10 Augmentations (artificial Metabolite CNs simulated from subsets of known reactions). For each year, run 20 sessions of 10-fold cross-validation (each pathway appearing once in a validation set) and, in every fold, train five M5 classifiers: a baseline on original Metabolite CN signals only, plus four models additionally trained on pathway waveforms drawn from 1, 3, 5, or 7 Augmentations randomly selected from the 10. Train each M5 model with the Adam optimizer at learning rate 0.00001 for 800 epochs, stride 8, using balanced positive-negative sets of 126 pathways each, where pathway signals are averages of participant metabolite waveforms. Record micro-average AUC per model and year across all 20 sessions, then aggregate into box plots (median, mean, maximum AUC, interquartile range, and whisker range) per number of Augmentations following the paper's Fig 4 format. The decision rule: pick the augmentation count whose model shows the best median/mean/maximum AUC with the smallest spread — in the paper this was one Augmentation, with average whisker range reduced from 0.082 (baseline) to 0.07.

## Related tools

- **NetAurHPD** (Framework being ablated: its M5 classifier is retrained with 1/3/5/7 Augmentations and baseline signals) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **M5** (Very deep convolutional neural network trained per fold as classification model on pathway waveforms) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **Adam optimizer** (Optimizer used at learning rate 0.00001 for 800 epochs during each augmented/baseline training run)
- **PyTorch** (Implementation backend for the network auralization and M5 training pipeline)

## Examples

```
instrument = SignalPropagation(momentum=0.999, response_len=10000, tqdm=lambda x: x, device); train_w, test_w = instrument.networkx_auralization(train_positive_hyperlink_dict, train_hyperlink_dict, test_hyperlink_dict, nodes, how_graph=True)
```

## Evaluation signals

- NetAurHPD with one Augmentation should show the best median/mean/maximum micro-average AUC across sessions
- Average whisker range should shrink from ~0.082 (NetAurHPD alone) to ~0.07 (with augmentation)
- Reproduced micro-average AUCs should land near 0.87 (2001), 0.857 (2003), and 0.884 (2004)
- Each pathway must appear exactly once in a validation set across the 10-fold cross-validation, repeated over 20 sessions
- Positive and negative sets must be balanced at 126 pathways each per training run

## Limitations

- Models do not consistently improve with more data — in some sessions the AUCs of all models decreased, so more augmentations do not guarantee gains
- NetAurHPD has a low level of explainability and cannot provide biologists a full understanding of its results
- Augmentation count selection is empirical: results depend on the 10 pre-generated Augmentations and the random selection of 1/3/5/7 per fold
- Sparsity of labeled data remains the most important challenge in metabolic pathway prediction, bounding what augmentation can fix

## Evidence

- [methods] one Augmentation was optimal, with micro-average AUC of 0.87 in 2001, 0.857 in 2003, and 0.884 in 2004: [evidence span withheld — anchor status: not_found]
- [methods] Training with one Augmentation improved NetAurHPD stability and reduced variability: "the range decreases from an average of 0.082 for NetAurHPD alone to an average of 0.07"
- [methods] 10-fold cross-validation with each pathway once in validation: "We evaluated the proposed solution with 10-fold cross-validation, where each pathway in the dataset appeared once in the validation set"
- [readme] README describes M5 as classification model trained on averaged hyperlink waveforms: "In NetAurHPD we average hyperlinks waveforms to represent a hyperlink throgh a signal. Based on these hyperlinks waveforms we train M5 (very deep convolutional neural network) as classification model."
- [abstract] Augmentation motivation from sparsity of labeled data: "The most important challenge in prediction of metabolic pathways is the sparsity of the labeled data."
