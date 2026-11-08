---
name: cross-organism-metabolite-protein-interaction-evaluation
description: Use when two organism-specific MPI classifiers must be compared on identical
  shared metabolite-protein pairs to test whether predictions are consistent across
  metabolic networks.
when_to_use_negative:
- When the goal is standard within-organism classifier evaluation (use evaluate.py
  per gold standard instead)
- When evaluating performance on subsystem-specific exclusions such as Alternate Carbon
  Metabolism or Cofactor and Prosthetic Group Biosynthesis (use subsys.py)
- When input features are molecular fingerprints (fp) rather than shadow prices —
  this cross-organism agreement task is framed around shadow-price features
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_2269
tools:
- name: SARTRE
  role: Framework providing the shadow-price features and evaluation scripts for cross-organism
    MPI prediction
  repo: https://github.com/fayazsoleymani/SARTRE
- name: scikit-learn
  role: Random forest classifier training and cosine-similarity/accuracy metrics
- name: Python
  role: Execution environment for shared.py in the subsys_shared directory
- name: iJO1366
  role: E. coli genome-scale metabolic model used to compute shadow prices for the
    E. coli side
- name: Yeast-GEM (Yeast8)
  role: S. cerevisiae consensus metabolic model used to compute shadow prices for
    the yeast side
  repo: https://github.com/SysBioChalmers/yeast-GEM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:061d6164a4e1ca28392be6c57bf76ac8b48f93e4a862c495c3bc5fd6d04976e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fayazsoleymani/SARTRE
  tools:
  - Python
  - SARTRE
  - Yeast-GEM (Yeast8)
  - iJO1366
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-organism-metabolite-protein-interaction-evaluation

## Summary

Trains organism-specific random forest classifiers on shadow-price features from two GEMs (iJO1366 for E. coli, Yeast-GEM/Yeast8 for S. cerevisiae), excludes metabolite-protein pairs shared between the GEMs, applies each model to the shared pairs, and quantifies cross-organism prediction agreement via accuracy and cosine similarity of prediction vectors.

## When to use

Use when two organism-specific MPI classifiers must be compared on identical shared metabolite-protein pairs to test whether predictions are consistent across metabolic networks.

## When NOT to use

- When the goal is standard within-organism classifier evaluation (use evaluate.py per gold standard instead)
- When evaluating performance on subsystem-specific exclusions such as Alternate Carbon Metabolism or Cofactor and Prosthetic Group Biosynthesis (use subsys.py)
- When input features are molecular fingerprints (fp) rather than shadow prices — this cross-organism agreement task is framed around shadow-price features

## Inputs

- Preprocessed shadow-price feature tables for stitch_ecoli (iJO1366-based)
- Preprocessed shadow-price feature tables for stitch_yeast (Yeast-GEM/Yeast8-based)
- Gold standard MPI labels from STITCH for both organisms
- List of shared metabolite-protein pairs existing in both GEMs

## Outputs

- Accuracy of the E. coli model on shared pairs (~0.754)
- Accuracy of the S. cerevisiae model on shared pairs (~0.755)
- Cosine similarity between the two prediction vectors (~0.980)
- shared_results.csv containing the two accuracies and the cosine similarity

## How to apply

Load the preprocessed shadow-price feature tables and gold standard labels for stitch_ecoli and stitch_yeast produced by cleaning.py. Identify and exclude the shared metabolite-protein pairs present in both GEMs from each dataset, then train a separate scikit-learn random forest classifier on the remaining pairs per organism (STITCH datasets take a confidence-score argument, e.g. 150/400/700/900). Apply both trained classifiers to the excluded shared pairs to obtain prediction vectors and compute each model's accuracy plus the cosine similarity between the two vectors. The rationale is that high agreement on pairs unseen during training but present in both networks supports that SARTRE's shadow-price features capture organism-independent interaction signals. Write the two accuracies and the cosine similarity to shared_results.csv.

## Related tools

- **SARTRE** (Framework providing the shadow-price features and evaluation scripts for cross-organism MPI prediction) — https://github.com/fayazsoleymani/SARTRE
- **scikit-learn** (Random forest classifier training and cosine-similarity/accuracy metrics)
- **Python** (Execution environment for shared.py in the subsys_shared directory)
- **iJO1366** (E. coli genome-scale metabolic model used to compute shadow prices for the E. coli side)
- **Yeast-GEM (Yeast8)** (S. cerevisiae consensus metabolic model used to compute shadow prices for the yeast side) — https://github.com/SysBioChalmers/yeast-GEM

## Examples

```
cd subsys_shared; python3 shared.py
```

## Evaluation signals

- E. coli model accuracy on shared pairs reproduces ~0.754
- S. cerevisiae model accuracy on shared pairs reproduces ~0.755
- Cosine similarity between the two prediction vectors is approximately 0.980
- Shared metabolite-protein pairs are verifiably excluded from each training set before model fitting
- Outputs (two accuracies and cosine similarity) are written to shared_results.csv

## Limitations

- Results depend on prior completion of optimization.py and cleaning.py for both stitch_ecoli and stitch_yeast; the shadow-price optimization requires an active GUROBI license
- STITCH-derived gold standards require choosing a confidence score threshold (150 low, 400 medium, 700 high, 900 highest), which affects the dataset composition
- Agreement is only assessed on the subset of MPIs shared between the two GEMs, so conclusions do not extend to organism-specific interactions

## Evidence

- [methods] workflow step: exclude shared pairs and train separate models: "evaluation has been done by excluding shared metabolite-protein pairs, which exist in two GEMs, from the datasets, and training two separate models in remaining pairs"
- [abstract] finding: agreement of organism-specific classifiers: "The quality of predictions is further supported by the excellent agreement of the organism-specific classifiers on unseen interactions shared between the two model organisms."
- [readme] README invocation for the shared-pairs evaluation: "cd subsys_shared
python3 shared.py

The accuracy of two models on the test set and cosine similarity of predictions to classifiers will be displayed."
- [other] task card validation targets: [evidence span withheld — anchor status: not_found]
- [methods] classifier library used: "The classifier and metrics are employed from [sklearn](https://scikit-learn.org/stable/) python library"
