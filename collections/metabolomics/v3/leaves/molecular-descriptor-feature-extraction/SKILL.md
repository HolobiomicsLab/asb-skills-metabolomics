---
name: molecular-descriptor-feature-extraction
description: Use when you need to generate a curated molecular descriptor feature
  set from compound structures (SMILES) for regression modeling — e.
when_to_use_negative:
- The target property is not a size/shape-dependent collision cross section measurable
  by ion mobility-mass spectrometry — Mordred descriptors were selected because they
  encode shape, symmetry, and bulk properties relevant to CCS; for other targets different
  feature families may be needed
- You already have a fixed, pre-computed feature table and do not need to generate
  descriptors from raw structures — this skill computes a full 1613-descriptor Mordred
  space then prunes it, which is wasted work if features are already extracted
- Current training sets allow the prediction of the most common adduct ion types including
  [M − H]−, [M + H]+, [M + Na]+, and [M + K]+ but not dimers such as [2 M + H]+ or
  [2 M − H]−, or other less common adducts or in-source fragments — so do not apply
  this to dimer or uncommon adduct species
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_3474
tools:
- name: CCSP 2.0
  role: Notebook hosting the descriptor extraction, linear SVR training, and RFE feature-selection
    workflow
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package that computes the 1613 molecular descriptors from compound
    structures, providing the full feature pool that RFE then prunes
- name: Jupyter Notebook
  role: Execution environment for running the CCSP 2.0 notebook to extract descriptors
    and select features; a Google Colab variant and a locally hosted Jupyter Lab version
    with a Tk interface are both provided
  repo: https://github.com/facundof2016/CCSP2.0
- name: scikit-learn (Sklearn)
  role: Linear support vector regression engine used to fit CCS prediction models
    and drive recursive feature elimination; version V1.0.2 or later is required
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:0edb98c4897f156fa4b89f980651109379d2db6eaa254850cdc5ec86ffa77e4f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  - scikit-learn (Sklearn)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# molecular-descriptor-feature-extraction

## Summary

Extract and curate a large molecular descriptor feature space from SMILES structures using the Mordred Python package inside the CCSP 2.0 Jupyter notebook, then apply recursive feature elimination (RFE) with linear support vector regression to select the compact descriptor subsets (averaging 370, 303, and 414 descriptors for [M−H]−, [M+H]+, and [M+Na]+ adducts from the 1613-descriptor pool) that drive collision cross section prediction.

## When to use

Use when you need to generate a curated molecular descriptor feature set from compound structures (SMILES) for regression modeling — e.g., predicting collision cross section (CCS) values for [M−H]−, [M+H]+, or [M+Na]+ ion types — and must control dimensionality by selecting a subset of the ~1613 Mordred descriptors rather than using the full feature space.

## When NOT to use

- The target property is not a size/shape-dependent collision cross section measurable by ion mobility-mass spectrometry — Mordred descriptors were selected because they encode shape, symmetry, and bulk properties relevant to CCS; for other targets different feature families may be needed
- You already have a fixed, pre-computed feature table and do not need to generate descriptors from raw structures — this skill computes a full 1613-descriptor Mordred space then prunes it, which is wasted work if features are already extracted
- Current training sets allow the prediction of the most common adduct ion types including [M − H]−, [M + H]+, [M + Na]+, and [M + K]+ but not dimers such as [2 M + H]+ or [2 M − H]−, or other less common adducts or in-source fragments — so do not apply this to dimer or uncommon adduct species

## Inputs

- User-curated training set of IM-MS collision cross section (CCS) measurements for target adduct ion types ([M−H]−, [M+H]+, [M+Na]+) with paired molecular structures (SMILES or equivalent chemical representations)

## Outputs

- RFE-selected descriptor subset per adduct model (averaging 370, 303, and 414 descriptors for [M−H]−, [M+H]+, and [M+Na]+ respectively)
- Ranking of descriptor classes (e.g., ATS, Barysz matrix, Kier, TIC, constitutional) by SVR feature weight magnitude per adduct model
- CSV file descriptor_class_rankings.csv containing per-adduct descriptor counts and descriptor class rankings
- Trained linear SVR models ready for CCS prediction on new molecules

## How to apply

Load the CCSP 2.0 notebook from the facundof2016/CCSP2.0 repository in a Python 3 Jupyter Lab, Jupyter Notebook, or Google Colab environment (Python 3 and Sklearn V1.0.2 or later are required per the README; verify package requirements against requirements.txt). Supply user-curated training sets of ion mobility-mass spectrometry CCS measurements (e.g., 463 test CCS measurements from the McLean CCS Compendium covering [M−H]−, [M+H]+, [M+Na]+ adducts) together with the compound structures. Compute 1613 molecular descriptors for each structure using the Mordred Python package, then train linear support vector regression models per adduct ion type and apply recursive feature elimination (RFE) to prune to 370, 303, and 414 descriptors on average for [M−H]−, [M+H]+, and [M+Na]+ respectively. Rank descriptor classes (ATS, Barysz matrix, Kier, TIC, constitutional) by the magnitude of their SVR feature weights within each adduct model to interpret which chemical property categories (shape, symmetry, bulk properties) dominate prediction. Write the per-adduct selected descriptor counts and class rankings to descriptor_class_rankings.csv.

## Related tools

- **CCSP 2.0** (Notebook hosting the descriptor extraction, linear SVR training, and RFE feature-selection workflow) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package that computes the 1613 molecular descriptors from compound structures, providing the full feature pool that RFE then prunes)
- **Jupyter Notebook** (Execution environment for running the CCSP 2.0 notebook to extract descriptors and select features; a Google Colab variant and a locally hosted Jupyter Lab version with a Tk interface are both provided) — https://github.com/facundof2016/CCSP2.0
- **scikit-learn (Sklearn)** (Linear support vector regression engine used to fit CCS prediction models and drive recursive feature elimination; version V1.0.2 or later is required)

## Examples

```
python -m jupyter notebook "CCSP 2.0 - CCS Prediction in Jupyter.ipynb"  # load notebook, compute 1613 Mordred descriptors from SMILES, train linear SVR per adduct ([M-H]−, [M+H]+, [M+Na]+), apply RFE to select ~370/303/414 descriptors, write descriptor_class_rankings.csv
```

## Evaluation signals

- RFE-selected descriptor averages must match the reported 370, 303, and 414 descriptors for [M−H]−, [M+H]+, and [M+Na]+ adducts when run against the 463-compound McLean CCS Compendium test set
- Dominant descriptor classes by SVR feature weight must reproduce the paper's reported findings (ATS, Barysz matrix, Kier, TIC, and constitutional descriptors encoding shape, symmetry, and bulk properties)
- The descriptor pool must contain 1613 molecular descriptors as computed by the Mordred Python package before RFE pruning
- Achieved CCS prediction median relative errors should be ≤2% (specifically 1.25, 1.73, and 1.87% for [M−H]−, [M+H]+, [M+Na]+), confirming the selected feature subset retains predictive power

## Limitations

- RFE-selected descriptor counts are averages that vary with training set composition; the specific descriptor subset will differ for user-curated training sets different from the McLean CCS Compendium test set used in the paper
- Descriptor classes were analyzed for the three adduct types ([M−H]−, [M+H]+, [M+Na]+) that the current training sets support; prediction of dimers ([2 M + H]+, [2 M − H]−), less common adducts, or in-source fragments is not supported
- One drawback of most ML methods is the need to define the specific training set to be used prior to predicting CCS values, so the selected feature space is coupled to the adduct types and molecules in the training set
- Future work will explore CCS predictions across multiple IM-MS instrument types (drift tube and traveling wave IM platforms), so current descriptor models may not transfer directly between instrument platforms
- A threshold ΔCCS of 2.8% and mass error of 10 ppm govern downstream CCS filtering with the selected descriptors; careful superclass-matched models may be needed to improve annotation confidence

## Evidence

- [abstract] Dominant descriptor classes reported: [evidence span withheld — anchor status: not_found]
- [readme] Execution environment options from README: [evidence span withheld — anchor status: external_doc]
