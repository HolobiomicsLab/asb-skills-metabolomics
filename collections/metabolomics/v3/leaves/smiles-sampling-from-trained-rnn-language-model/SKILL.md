---
name: smiles-sampling-from-trained-rnn-language-model
description: Use when you have a trained RNN chemical language model checkpoint (e.
when_to_use_negative:
- The goal is annotation or prioritization of existing metabolomics peaks by m/z rather
  than de novo structure generation (use DeepMet's m/z filtering and MS/MS meta-learning
  workflow instead)
- The target chemistry is evolutionarily distant from the model's training set (e.g.,
  plant or bacterial metabolism), since the language model was trained and evaluated
  only on human metabolite structures
- You need to predict MS/MS spectra or retention times from structures, which the
  authors note remain open problems independent of sampling
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: DeepMet / CLM
  role: Provides the trained RNN checkpoints and the sampling routine used to draw
    novel SMILES
  repo: https://github.com/skinniderlab/CLM.git
- name: PyTorch
  role: Backend executing the RNN sampling; GPU access greatly accelerates the end-end
    workflow
  repo: https://github.com/skinniderlab/CLM.git
- name: RDKit
  role: Canonicalizes sampled SMILES and enables InChIKey-based removal of training-set
    duplicates
- name: Snakemake
  role: Orchestrates the end-to-end clm workflow (preprocessing, training, sampling)
    via the included Snakefile
  repo: https://github.com/skinniderlab/CLM.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:603c64a609f469dfb1b69a118358daa993df7cdb1e4e761e6c7f18002a7228f8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM.git
  tools:
  - DeepMet
  - DeepMet / CLM
  - PyTorch
  - Python
  - RDKit
  - Snakemake
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# smiles-sampling-from-trained-rnn-language-model

## Summary

Generates large batches of novel SMILES by sampling from a trained chemical language model (DeepMet/CLM RNN checkpoints), producing the raw candidate structures needed for metabolite-likeness evaluation and downstream discovery.

## When to use

Use when you have a trained RNN chemical language model checkpoint (e.g., one cross-validation fold of DeepMet/CLM trained on known human metabolites) and need to sample hundreds of thousands of de novo SMILES for downstream filtering, classifier-based likeness testing, or MS/MS-based metabolite prioritization.

## When NOT to use

- The goal is annotation or prioritization of existing metabolomics peaks by m/z rather than de novo structure generation (use DeepMet's m/z filtering and MS/MS meta-learning workflow instead)
- The target chemistry is evolutionarily distant from the model's training set (e.g., plant or bacterial metabolism), since the language model was trained and evaluated only on human metabolite structures
- You need to predict MS/MS spectra or retention times from structures, which the authors note remain open problems independent of sampling

## Inputs

- Trained RNN model checkpoint from one cross-validation fold (produced by clm train_models_RNN)
- config.yaml specifying train_seeds and fold splits (train/test SMILES files and .vocabulary files)
- Training-set SMILES file for the fold (used to remove training duplicates via InChIKey)

## Outputs

- Canonicalized, de-duplicated set of ~500,000 sampled SMILES per fold
- Sampled-molecule pool for fingerprint-based random forest classification, UMAP chemical-space plots, and Tanimoto nearest-neighbor comparisons
- Metrics written to metabolite_likeness_evaluation.csv

## How to apply

Load a model checkpoint trained via the clm train_models_RNN step (PyTorch), where the number of models is controlled by the train_seeds entries in config.yaml, and run the clm sampling routine to draw 500,000 SMILES per fold. Canonicalize the raw sampled strings with RDKit (the same conversion-to-molecules machinery used during preprocessing, which also removes light fragments and neutralizes charges) and drop any sample whose InChIKey duplicates a molecule in the training file for that fold. The end-to-end pipeline is orchestrated by the repository's Snakefile in Snakemake; sampling runs on GPU-backed torch. The resulting canonical, de-duplicated sample set is then fed into metabolite-likeness tests (random forest AUC vs. withheld test metabolites, BioTransformer transformation recapitulation, Tanimoto similarity vs. formula-matched PubChem controls) and into construction of the 34.4-million-structure generated database.

## Related tools

- **DeepMet / CLM** (Provides the trained RNN checkpoints and the sampling routine used to draw novel SMILES) — https://github.com/skinniderlab/CLM.git
- **PyTorch** (Backend executing the RNN sampling; GPU access greatly accelerates the end-end workflow) — https://github.com/skinniderlab/CLM.git
- **RDKit** (Canonicalizes sampled SMILES and enables InChIKey-based removal of training-set duplicates)
- **Snakemake** (Orchestrates the end-to-end clm workflow (preprocessing, training, sampling) via the included Snakefile) — https://github.com/skinniderlab/CLM.git

## Examples

```
snakemake --use-conda --cores 8  # run the end-end clm workflow (preprocess, create_training_sets, train_models_RNN, sample) from the repository Snakefile
```

## Evaluation signals

- Exactly 500,000 SMILES sampled per fold checkpoint (or per train_seeds entry in config.yaml)
- All retained samples parse as valid canonical SMILES in RDKit after fragment removal and charge neutralization
- No retained sample shares an InChIKey with any molecule in the fold's training file (zero training leakage)
- Downstream random forest classifier separating sampled molecules from withheld test-fold metabolites yields AUC ≈ 0.48 (chance level)
- BioTransformer one-step transformation recapitulation reaches ~77.5%, with metrics written to metabolite_likeness_evaluation.csv

## Limitations

- The language model was trained and evaluated on the structures of human metabolites; evolutionarily distant applications (e.g., plant or bacterial metabolism) will likely require bespoke models
- The model has inherent limitations in anticipating metabolites that originate from divergent, as-of-yet undiscovered biosynthetic routes
- Novelty filtering by InChIKey against the training fold is essential — unfiltered samples may include memorized training structures that inflate apparent model quality
- GPU access is effectively required for practical sampling speed

## Evidence

- [readme] Repository purpose: [evidence span withheld — anchor status: external_doc]
