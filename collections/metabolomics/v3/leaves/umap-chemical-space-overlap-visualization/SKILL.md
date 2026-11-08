---
name: umap-chemical-space-overlap-visualization
description: 'Use when you have SMILES sampled from a chemical language model checkpoint
  and need to verify metabolite-likeness: generated structures should be indistinguishable
  from withheld test-fold metabolites in fingerprint space.'
when_to_use_negative:
- The language model was trained on a non-human metabolome (e.g., plant or bacterial
  metabolism) — the article notes evolutionarily distant applications likely require
  bespoke models, so human-metabolome benchmarks like this do not transfer directly.
- Your goal is to prioritize generated structures by mass-to-charge ratio against
  MS/MS data rather than to assess overall metabolite-likeness of the sampled set.
- Sampled molecules have not been de-duplicated against the training fold via InChIKey
  — memorized training structures will deflate the apparent novelty and distort the
  overlap assessment.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2252
tools:
- name: DeepMet
  role: Chemical language model whose sampled molecules are the subject of the chemical-space
    overlap evaluation
  repo: github.com/skinniderlab/CLM
- name: RDKit
  role: Canonicalizes sampled SMILES, computes InChIKeys for training-set filtering,
    and generates molecular fingerprints for the random forest and UMAP embedding
- name: PyTorch
  role: Loads the trained RNN checkpoint from which SMILES are sampled
- name: Snakemake
  role: Runs the end-end clm workflow including train_models_RNN and sampling steps
    feeding this evaluation
  repo: github.com/skinniderlab/CLM
- name: scikit-learn
  role: Trains the random forest classifier that distinguishes filtered sampled molecules
    from withheld test-fold metabolites (AUC ~0.48 expected)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:845f90fd4ecdd69d98dd0e8b5c6db9064128a371e03f0babcc8a7113278171f2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DeepMet
  - PyTorch
  - Python
  - RDKit
  - Snakemake
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2252
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# umap-chemical-space-overlap-visualization

## Summary

Evaluates whether molecules sampled from a trained chemical language model (DeepMet/CLM) occupy the same chemical space as real metabolites by embedding RDKit molecular fingerprints with UMAP and confirming that a random forest classifier can only separate generated from withheld metabolites at chance level (AUC ~0.48).

## When to use

Use when you have SMILES sampled from a chemical language model checkpoint and need to verify metabolite-likeness: generated structures should be indistinguishable from withheld test-fold metabolites in fingerprint space.

## When NOT to use

- The language model was trained on a non-human metabolome (e.g., plant or bacterial metabolism) — the article notes evolutionarily distant applications likely require bespoke models, so human-metabolome benchmarks like this do not transfer directly.
- Your goal is to prioritize generated structures by mass-to-charge ratio against MS/MS data rather than to assess overall metabolite-likeness of the sampled set.
- Sampled molecules have not been de-duplicated against the training fold via InChIKey — memorized training structures will deflate the apparent novelty and distort the overlap assessment.

## Inputs

- Trained RNN model checkpoint from one cross-validation fold (clm train_models_RNN output, PyTorch)
- Sampled SMILES file (e.g., 500,000 generations from the clm sampling routine)
- Withheld test-fold metabolite SMILES for that fold
- Formula-matched PubChem control SMILES
- Training-fold SMILES file used for InChIKey de-duplication

## Outputs

- 2D UMAP embedding of molecular fingerprints showing overlap of generated vs. withheld metabolite chemical space
- Random forest AUC quantifying separability of generated vs. withheld metabolites (expect ~0.48)
- BioTransformer one-step transformation recapitulation fraction (expect ~77.5%)
- Nearest-neighbor Tanimoto similarities to formula-matched PubChem controls
- metabolite_likeness_evaluation.csv containing all metrics

## How to apply

Load a trained RNN checkpoint from one cross-validation fold (the train_models_RNN step of the clm Snakemake workflow) and sample ~500,000 SMILES with the clm sampling routine. Canonicalize the sampled SMILES with RDKit and remove any molecule whose InChIKey matches the training file for that fold, so the evaluation is strictly against unseen structures. Compute RDKit fingerprints for the filtered generated molecules and the withheld test-fold metabolites; train a scikit-learn random forest to distinguish the two classes and confirm the AUC is approximately 0.48 (random-guessing). Embed the fingerprint matrix with UMAP and inspect that the generated and metabolite clouds overlap rather than forming separated clusters. Corroborate with two additional signals: BioTransformer one-step transformation recapitulation (~77.5% of sampled molecules should be transformation products of known metabolites) and nearest-neighbor Tanimoto similarity against formula-matched PubChem control molecules; write all metrics to metabolite_likeness_evaluation.csv.

## Related tools

- **DeepMet** (Chemical language model whose sampled molecules are the subject of the chemical-space overlap evaluation) — github.com/skinniderlab/CLM
- **RDKit** (Canonicalizes sampled SMILES, computes InChIKeys for training-set filtering, and generates molecular fingerprints for the random forest and UMAP embedding)
- **PyTorch** (Loads the trained RNN checkpoint from which SMILES are sampled)
- **Snakemake** (Runs the end-end clm workflow including train_models_RNN and sampling steps feeding this evaluation) — github.com/skinniderlab/CLM
- **scikit-learn** (Trains the random forest classifier that distinguishes filtered sampled molecules from withheld test-fold metabolites (AUC ~0.48 expected))

## Examples

```
import umap; emb = umap.UMAP(n_components=2).fit_transform(fp_matrix)
```

## Evaluation signals

- Random forest AUC separating generated molecules from withheld metabolites is approximately 0.48 (random-guessing level); a substantially higher AUC indicates the model memorized training structures or generated off-target chemistry.
- BioTransformer one-step transformation recapitulation reaches approximately 77.5% of sampled molecules.
- Nearest-neighbor Tanimoto similarities (RDKit fingerprints) against formula-matched PubChem controls support structural proximity to the known metabolome.
- The UMAP projection shows intermixed generated and withheld-metabolite clouds rather than two separated clusters.
- All metrics are written to metabolite_likeness_evaluation.csv.

## Limitations

- Chance-level AUC alone is a negative result; it must be corroborated with BioTransformer transformation recapitulation and Tanimoto nearest-neighbor checks against formula-matched PubChem controls.
- The underlying model was trained and evaluated on human metabolite structures; applications to evolutionarily distant organisms (plant, bacterial metabolism) likely require retraining.
- The language model has inherent limitations anticipating metabolites that originate from divergent, as-of-yet undiscovered biosynthetic routes, so absence of a class in the UMAP overlap does not prove the model cannot produce it.

## Evidence

- [readme] This repository contains Python source code required to train and evaluate chemical language models.: [evidence span withheld — anchor status: external_doc]
