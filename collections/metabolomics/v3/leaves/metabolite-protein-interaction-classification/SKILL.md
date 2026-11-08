---
name: metabolite-protein-interaction-classification
description: Use when gold-standard MPI pairs (piazza, reznik, STITCH ecoli/yeast)
  are available and features must be computed from constraint-based shadow prices
  to train and benchmark an MPI classifier.
when_to_use_negative:
- When no gold-standard MPI labels exist for the organism or gold standard of interest,
  so supervised training/validation is impossible
- When the question is about reaction feasibility or flux distributions rather than
  interaction prediction — shadow prices here are only features for classification
- When GUROBI (license required) or the curated irreversible model is unavailable,
  since shadow prices cannot be computed as in this workflow
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_1775
tools:
- name: SARTRE
  role: Framework orchestrating shadow-price feature extraction and MPI classification
  repo: https://github.com/fayazsoleymani/SARTRE
- name: COBRAToolbox
  role: Curation of iJO1366 and Yeast-GEM in MATLAB to irreversible optimized models
  repo: https://github.com/opencobra/cobratoolbox
- name: GUROBI
  role: Optimizer (via Python interface) computing shadow prices in optimization.py
  repo: https://www.gurobi.com/
- name: RDKit
  role: Generates 128-bit Morgan fingerprints as the baseline feature set (generate_fp.py)
  repo: https://www.rdkit.org/
- name: scikit-learn
  role: RandomForestClassifier and accuracy/AUC/F1 metrics used by evaluate.py
  repo: https://scikit-learn.org/stable/
- name: Bio.Entrez
  role: Identifier retrieval in generate_fp.py (prompts for email)
  repo: https://biopython.org/docs/1.76/api/Bio.Entrez.html
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_006
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:6e36a3cf071d483dfba00991f3765aef79c24ba0d50e9f04e7354bb101cc4126
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fayazsoleymani/SARTRE
  tools:
  - Bio.Entrez
  - COBRAToolbox
  - GUROBI
  - MATLAB
  - Python
  - RDKit
  - SARTRE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_1775
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# metabolite-protein-interaction-classification

## Summary

Train and evaluate random forest classifiers (scikit-learn) that predict metabolite-protein interactions (MPIs) using shadow prices from flux variability analysis of genome-scale metabolic models (iJO1366, Yeast8) as features, benchmarked against 128-bit RDKit molecular fingerprints.

## When to use

Use when gold-standard MPI pairs (piazza, reznik, STITCH ecoli/yeast) are available and features must be computed from constraint-based shadow prices to train and benchmark an MPI classifier.

## When NOT to use

- When no gold-standard MPI labels exist for the organism or gold standard of interest, so supervised training/validation is impossible
- When the question is about reaction feasibility or flux distributions rather than interaction prediction — shadow prices here are only features for classification
- When GUROBI (license required) or the curated irreversible model is unavailable, since shadow prices cannot be computed as in this workflow

## Inputs

- Curated irreversible GEM (ModelIrrevOpt90.mat from iJO1366.mat; yeast_IrrevOpt90.mat from Yeast-GEM.mat)
- Gold-standard MPI label sets (piazza, reznik, stitch_ecoli, stitch_yeast)
- Shadow-price feature table met_sps_t80_replaced.csv
- 128-bit RDKit Morgan fingerprint table met_fp128.csv
- Optional STITCH confidence score argument (150/400/700/900)

## Outputs

- Trained random forest MPI classifiers per gold standard and feature set
- Accuracy, AUC and F1 metrics per feature set (e.g. piazza_evaluation_metrics.csv, reznik_evaluation_metrics.csv)
- Statistical comparison of shadow-price vs fingerprint classifiers

## How to apply

Curate the GEM with COBRAToolbox in MATLAB (modelPreparing_iJO1366.m turns iJO1366.mat into ModelIrrevOpt90.mat; the analogous script creates yeast_IrrevOpt90.mat from Yeast-GEM.mat). Compute FVA shadow prices per reaction with GUROBI via its Python interface (optimization.py, results saved per reaction in the opt/ directory), then preprocess with cleaning.py to produce met_sps_t80_replaced.csv and generate 128-bit Morgan fingerprints with RDKit via generate_fp.py to produce met_fp128.csv as the baseline feature set. Train the sklearn RandomForestClassifier with class undersampling and 5-fold cross-validation by running evaluate.py sp (shadow prices) and evaluate.py fp (fingerprints) in each gold standard directory; STITCH-derived datasets require a confidence-score argument (150 low, 400 medium, 700 high, 900 highest). Record accuracy, ROC-AUC and F1 for both feature sets and compare, expecting shadow prices to significantly outperform fingerprints.

## Related tools

- **SARTRE** (Framework orchestrating shadow-price feature extraction and MPI classification) — https://github.com/fayazsoleymani/SARTRE
- **COBRAToolbox** (Curation of iJO1366 and Yeast-GEM in MATLAB to irreversible optimized models) — https://github.com/opencobra/cobratoolbox
- **GUROBI** (Optimizer (via Python interface) computing shadow prices in optimization.py) — https://www.gurobi.com/
- **RDKit** (Generates 128-bit Morgan fingerprints as the baseline feature set (generate_fp.py)) — https://www.rdkit.org/
- **scikit-learn** (RandomForestClassifier and accuracy/AUC/F1 metrics used by evaluate.py) — https://scikit-learn.org/stable/
- **Bio.Entrez** (Identifier retrieval in generate_fp.py (prompts for email)) — https://biopython.org/docs/1.76/api/Bio.Entrez.html

## Examples

```
cd piazza; python3 optimization.py; python3 cleaning.py; python3 generate_fp.py; python3 evaluate.py sp; python3 evaluate.py fp
```

## Evaluation signals

- Shadow-price classifier attains average AUC ~0.86 for E. coli and ~0.85 for S. cerevisiae
- SARTRE significantly outperforms the 128-bit RDKit fingerprint baseline: P < 1e-5 on the piazza gold standard and P < 1e-7 on the reznik gold standard
- Feature importance ranking from RF feature_importances_ shows shadow-price features dominate over fingerprint bits
- Subsystem-exclusion runs reproduce accuracies of ~0.615 (Alternate Carbon Metabolism) and ~0.815 (Cofactor and Prosthetic Group Biosynthesis)
- Metrics files (e.g. piazza_evaluation_metrics.csv) contain accuracy, AUC and F1 entries for both sp and fp feature sets

## Limitations

- Requires an active GUROBI license before running optimization.py; results must be unzipped from the opt/ archives
- Accuracy drops on subsystem-restricted predictions (0.615 for Alternate Carbon Metabolism, 0.815 for Cofactor and Prosthetic Group Biosynthesis) due to a larger number of false positives
- Performance depends on STITCH confidence threshold choice (150/400/700/900); shadow prices covering different metabolic systems affect accuracy

## Evidence

- [abstract] SARTRE with random forest classifiers accurately predicts MPIs with average AUC 0.86/0.85: "the implementation of SARTRE with random forest classifiers accurately predicts metabolite-protein interactions, supported by an average area under the receiver operating curve of 0.86 and 0.85"
- [methods] Classifier and metrics from sklearn via evaluate.py per gold standard directory: "Preprocessing, constructing datasets, training the random forest classifier and evaluating it can be executed by running `evaluate.py` in each gold standard directory"
- [methods] Optimization with GUROBI and its Python interface: "The optimization process has been implemented by [GUROBI](https://www.gurobi.com/) optimizer and its python interface"
- [readme] Shadow price outputs saved per reaction in opt directory: "The results of optimization will be saved on the `opt` directory for each reaction by its number."
- [readme] STITCH confidence score argument values: "Gold standard from STITCH also gets an additional argument as a confidence score(150: low, 400: medium, 700: high, 900: highest) to perform this process."
- [results] Fingerprint baseline significantly outperformed on reznik gold standard: [evidence span withheld — anchor status: not_found]
