---
name: fingerprint-random-forest-metabolite-classification
description: Use when evaluating whether a chemical language model's generated molecules
  are metabolite-like by classifying them against held-out metabolites via fingerprints.
when_to_use_negative:
- When the goal is prioritizing generated structures by mass-to-charge ratio for MS/MS
  spectral matching rather than assessing global metabolite-likeness.
- When the training set is not human metabolites (e.g., plant or bacterial metabolism)
  — the article notes evolutionarily distant applications require bespoke models.
- When you already have confirmed spectral identifications and only need to compute
  structural similarity to a reference library.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3335
- http://edamontology.org/topic_0091
tools:
- name: DeepMet
  role: Generates candidate metabolite SMILES from a chemical language model trained
    on known human metabolites
  repo: https://github.com/skinniderlab/CLM
- name: PyTorch
  role: Runs the trained RNN sampling step (clm sampling routine) used to produce
    candidate SMILES
  repo: https://github.com/skinniderlab/CLM
- name: RDKit
  role: Canonicalizes sampled SMILES, computes InChIKeys for deduplication, and generates
    molecular fingerprints
- name: scikit-learn
  role: Trains the random forest classifier on fingerprints and computes AUC
- name: Snakemake
  role: Orchestrates the end-end workflow (preprocessing, train/test split, training,
    sampling)
  repo: https://github.com/skinniderlab/CLM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:c84f441290aeecc28888312bfd4306b135dd4b382790363cb1af4fb6557d0435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - DeepMet
  - PyTorch
  - Python
  - RDKit
  - Snakemake
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3335
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Fingerprint random forest metabolite classification

## Summary

Trains a scikit-learn random forest on molecular fingerprints to test whether generated SMILES (from DeepMet/CLM) are structurally indistinguishable from withheld real metabolites; an AUC near 0.5 (chance level) indicates the generator recapitulates the known metabolome.

## When to use

Use when evaluating whether a chemical language model's generated molecules are metabolite-like by classifying them against held-out metabolites via fingerprints.

## When NOT to use

- When the goal is prioritizing generated structures by mass-to-charge ratio for MS/MS spectral matching rather than assessing global metabolite-likeness.
- When the training set is not human metabolites (e.g., plant or bacterial metabolism) — the article notes evolutionarily distant applications require bespoke models.
- When you already have confirmed spectral identifications and only need to compute structural similarity to a reference library.

## Inputs

- Sampled SMILES strings from a trained RNN model checkpoint (one per cross-validation fold)
- Withheld test-fold metabolite structures (canonical SMILES/InChIKeys)
- Training-file InChIKeys per fold for deduplication
- Formula-matched PubChem control molecules for similarity analysis

## Outputs

- metabolite_likeness_evaluation.csv containing random forest AUC, BioTransformer recapitulation rate, and Tanimoto similarity metrics
- UMAP plot showing chemical-space overlap of generated vs. real metabolites
- Trained random forest classifier object

## How to apply

Sample 500,000 SMILES from a trained RNN checkpoint (trained via `clm train_models_RNN` in PyTorch across cross-validation folds), canonicalize with RDKit, and remove any molecule whose InChIKey matches the training file for that fold. Convert kept molecules and the withheld test-fold metabolites to molecular fingerprints, then train a random forest classifier (scikit-learn) to separate the two classes. If AUC ≈ 0.48 (random-guessing level), the generated structures are indistinguishable from real metabolites; complementary checks include UMAP visualization of chemical-space overlap, ~77.5% recapitulation of BioTransformer one-step transformations, and nearest-neighbor Tanimoto similarity against formula-matched PubChem controls. Write all metrics to metabolite_likeness_evaluation.csv.

## Related tools

- **DeepMet** (Generates candidate metabolite SMILES from a chemical language model trained on known human metabolites) — https://github.com/skinniderlab/CLM
- **PyTorch** (Runs the trained RNN sampling step (clm sampling routine) used to produce candidate SMILES) — https://github.com/skinniderlab/CLM
- **RDKit** (Canonicalizes sampled SMILES, computes InChIKeys for deduplication, and generates molecular fingerprints)
- **scikit-learn** (Trains the random forest classifier on fingerprints and computes AUC)
- **Snakemake** (Orchestrates the end-end workflow (preprocessing, train/test split, training, sampling)) — https://github.com/skinniderlab/CLM

## Examples

```
snakemake --use-conda --cores 8 train_models_RNN && python clm/sampling.py --num_smiles 500000 && python evaluate_metabolite_likeness.py
```

## Evaluation signals

- Random forest AUC vs. withheld metabolites is approximately 0.48 (chance level); a substantially higher AUC would indicate the generator produces non-metabolite structures
- BioTransformer one-step transformation recapitulation rate reaches ~77.5% of sampled molecules
- Nearest-neighbor Tanimoto similarity of each sampled molecule to formula-matched PubChem controls supports structural proximity to the known metabolome
- UMAP projection shows overlap of generated and real metabolite chemical space rather than cluster separation
- Zero sampled molecules have InChIKeys matching the fold's training set after canonicalization and filtering

## Limitations

- The language model was trained and evaluated only on human metabolites; evolutionarily distant applications (plant or bacterial metabolism) likely require bespoke models.
- The approach has inherent limitations for anticipating metabolites from divergent, undiscovered biosynthetic routes.
- The novelty metric can be permissive — small changes to training molecules can yield high benchmark scores (see mgenerators-failure-modes).

## Evidence

- [methods] Random forest AUC ≈ 0.48: [evidence span withheld — anchor status: not_found]
- [methods] Workflow: sample 500,000 SMILES, deduplicate, classify with random forest: [evidence span withheld — anchor status: not_found]
- [methods] RDKit canonicalization and InChIKey deduplication: [evidence span withheld — anchor status: not_found]
